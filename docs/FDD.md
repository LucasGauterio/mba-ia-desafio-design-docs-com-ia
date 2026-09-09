# FDD: Sistema de Webhooks de Notificação de Pedidos

Versão: 1.0
Data: 2026-09-09
Responsável: Time de Plataforma / Pedidos

Documento de implementação. As decisões estão fechadas nos ADRs
([ADR-001](adrs/ADR-001-outbox-no-mysql.md) a
[ADR-007](adrs/ADR-007-snapshot-do-payload-na-insercao.md)); a proposta de arquitetura
está no [RFC](RFC.md). Este FDD não reabre decisões nem repete a justificativa de produto
do [PRD](PRD.md).

---

## 1. Contexto e motivação técnica

Os clientes B2B precisam saber quando o status de um pedido muda sem fazer polling no
`GET /orders` ([09:00] Marcos). O status só muda em `OrderService.changeStatus`
(`src/modules/orders/order.service.ts`), dentro de uma `prisma.$transaction` que já
atualiza `orders`, insere em `order_status_history` e ajusta o estoque.

A feature adiciona: um módulo `src/modules/webhooks` para configurar endpoints de webhook,
uma tabela outbox preenchida na mesma transação da mudança de status, um processo worker
que faz a entrega HTTP com retry, e uma tabela de dead letter com replay administrativo.
Atores: o **usuário autenticado** que gerencia a configuração de webhook pela API; o
**worker** que entrega; o **endpoint HTTP do cliente** que recebe; o **administrador** que
faz replay de DLQ.

Limites: só webhooks **outbound** (a plataforma envia, o cliente recebe) ([09:03] Sofia).
Só o evento `order.status_changed`.

## 2. Objetivos técnicos

- Se a transação de `changeStatus` commitou, o evento **foi registrado** na outbox; se deu
  rollback, o evento não existe ([09:41] Diego).
- Entrega em menos de 10 segundos no caminho feliz; o pior caso da fila é 2 segundos de
  polling mais o tempo de envio ([09:02] Marcos; [09:10] Larissa).
- Garantia **at-least-once**: um evento pode ser entregue mais de uma vez; nunca zero vez
  se foi registrado ([09:24] Diego).
- Toda requisição de saída assinada com HMAC-SHA256 e identificável por `X-Event-Id`.
- Ordenação por `order_id` preservada enquanto houver um único worker ([09:12] Diego).
- Zero alteração nos contratos HTTP existentes de pedidos, clientes e produtos.

## 3. Escopo e exclusões

**Incluído**

- Modelos `webhook_endpoint`, `webhook_outbox`, `webhook_delivery`, `webhook_dead_letter`.
- CRUD de configuração de webhook + rotação de secret + histórico de entregas (API HTTP).
- Função `publishWebhookEvent(tx, order, fromStatus, toStatus)` chamada em `changeStatus`.
- Filtro de status de interesse aplicado na inserção da outbox.
- Worker de entrega (`src/worker.ts` + `npm run worker`) com polling, retry, backoff e DLQ.
- Endpoint administrativo de replay de DLQ.

**Excluído** (ver `.claude/rules/honor-rejected-scope.md` e a seção "Fora de escopo" do PRD)

- E-mail ou qualquer alerta ao cliente quando o webhook falha ([09:37] Larissa).
- Dashboard ou painel visual ([09:39] Marcos).
- Arquivamento de linhas entregues da outbox ([09:08] Diego).
- Rate limiting de saída (questão em aberto no RFC) ([09:39] Larissa).
- Escalar para múltiplos workers ([09:13] Diego).
- Webhook inbound.

## 4. Fluxos detalhados

### 4.1 Criação do evento na outbox

1. Um cliente da API chama `PATCH /api/v1/orders/:id/status` (fluxo existente).
2. `OrderService.changeStatus` abre `prisma.$transaction` e faz o trabalho atual (validar
   transição, ajustar estoque, `order.update`, `orderStatusHistory.create`).
3. Ainda **dentro do mesmo `tx`**, `changeStatus` chama
   `publishWebhookEvent(tx, order, fromStatus, toStatus)`.
4. `publishWebhookEvent` consulta os `webhook_endpoint` ativos do `order.customerId` cujo
   `status_filter` contém `toStatus`. Se não houver nenhum, retorna sem escrever nada
   ([09:34] Bruno).
5. Se houver ao menos um, renderiza o payload do evento uma vez (snapshot, ver
   [ADR-007](adrs/ADR-007-snapshot-do-payload-na-insercao.md)) e insere **uma linha em
   `webhook_outbox` por endpoint alvo**, com `status = pending`, `attempts = 0`,
   `next_attempt_at = now()`, `id` UUID (que também é o `event_id`).
6. Se a inserção falhar (por exemplo, payload acima de 64 KB, ver §6), `publishWebhookEvent`
   lança um erro e a transação inteira sofre rollback ([09:40] Bruno).
7. Commit. O status mudou e os eventos estão registrados, ou nada aconteceu.

### 4.2 Processamento pelo worker

1. O worker (`src/worker.ts`) roda um loop. A cada **2 segundos** ([09:09] Diego):
2. Seleciona em lote pequeno os eventos com `status = pending` e `next_attempt_at <= now()`,
   ordenados por `created_at` ascendente (garante ordem por `order_id` com worker único).
3. Para cada evento: marca `status = processing`, resolve a secret vigente do
   `webhook_endpoint` (a atual e, se dentro do grace period, a anterior), calcula
   `X-Signature = HMAC-SHA256(payload, secret_atual)`.
4. Faz `POST` para a `url` do endpoint com timeout de **10 segundos** ([09:42] Diego),
   headers de §5.4, corpo = payload snapshot.
5. Grava uma linha em `webhook_delivery` com `outbox_id`, `attempt`, `status_code`,
   `response_body` (truncado), `duration_ms`, `error` (se houver).
6. **2xx**: marca o evento `status = delivered`, `delivered_at = now()`.
7. **Não 2xx, timeout ou erro de conexão**: trata como falha, vai para §4.3.

### 4.3 Retry com backoff

1. Falha no envio: `attempts = attempts + 1`.
2. Se `attempts < 5`: `status = pending`,
   `next_attempt_at = now() + backoff[attempts]`, onde
   `backoff = [1min, 5min, 30min, 2h, 12h]` ([09:17] Diego). O worker vai retentar quando a
   janela chegar.
3. Se `attempts >= 5`: vai para §4.4 (DLQ).

### 4.4 Dead letter queue

1. Esgotadas as 5 tentativas, o worker insere uma linha em `webhook_dead_letter` com
   `outbox_id`, `webhook_endpoint_id`, `payload` (o snapshot), `last_error`,
   `failed_at = now()` ([09:18] Diego).
2. O evento na `webhook_outbox` é marcado `status = dead_letter` (fica na tabela como
   histórico, mas o worker não o seleciona mais).
3. **Replay:** um administrador chama
   `POST /api/v1/admin/webhooks/dead-letter/:id/replay`. O serviço cria um **novo** evento
   em `webhook_outbox` com o mesmo payload, `status = pending`, `attempts = 0`,
   `next_attempt_at = now()`, e um **novo** `id`/`event_id` ([09:18] Diego). Registra em log
   quem executou o replay ([09:36] Sofia).

## 5. Contratos públicos

Base: `/api/v1`. Todos os endpoints de configuração exigem `Authorization: Bearer <jwt>`
(`authenticate`); o de replay exige adicionalmente role `ADMIN` (`requireRole('ADMIN')`).
Formato de erro: `{ "error": { "code", "message", "details"? } }` (padrão do
`src/middlewares/error.middleware.ts`).

### 5.1 `POST /api/v1/webhooks`: cadastrar endpoint de webhook

Auth: usuário autenticado. Status: `201`, `400` (`WEBHOOK_INVALID_URL`,
`WEBHOOK_EMPTY_STATUS_FILTER`, `VALIDATION_ERROR`), `404`/`422`
(`WEBHOOK_CUSTOMER_NOT_FOUND`), `409` (`WEBHOOK_DUPLICATE_URL`).

Request:
```json
{
  "customerId": "6f1d2c3a-1111-4a2b-9c3d-000000000001",
  "url": "https://hooks.atlascomercial.com.br/oms/orders",
  "statusFilter": ["SHIPPED", "DELIVERED"]
}
```

Response `201`:
```json
{
  "id": "b2a7e9d0-2222-4b3c-8d4e-000000000002",
  "customerId": "6f1d2c3a-1111-4a2b-9c3d-000000000001",
  "url": "https://hooks.atlascomercial.com.br/oms/orders",
  "statusFilter": ["SHIPPED", "DELIVERED"],
  "active": true,
  "secret": "whsec_EXEMPLO_valor_opaco_retornado_so_aqui",
  "secretExpiresAt": null,
  "createdAt": "2026-09-09T12:00:00.000Z"
}
```

A `secret` só aparece nesta resposta e na resposta de rotação; não é retornada em `GET`
([09:31] Marcos).

### 5.2 `GET /api/v1/webhooks?customerId=<uuid>`: listar endpoints de um customer

Auth: usuário autenticado. Status: `200`, `400` (`VALIDATION_ERROR` se `customerId`
ausente ou inválido).

Response `200` (formato `paginated`, `src/shared/http/response.ts`):
```json
{
  "data": [
    {
      "id": "b2a7e9d0-2222-4b3c-8d4e-000000000002",
      "customerId": "6f1d2c3a-1111-4a2b-9c3d-000000000001",
      "url": "https://hooks.atlascomercial.com.br/oms/orders",
      "statusFilter": ["SHIPPED", "DELIVERED"],
      "active": true,
      "secretRotatedAt": null,
      "createdAt": "2026-09-09T12:00:00.000Z"
    }
  ],
  "pagination": { "page": 1, "pageSize": 20, "total": 1, "totalPages": 1 }
}
```

### 5.3 `PATCH /api/v1/webhooks/:id`: editar endpoint

Auth: usuário autenticado. Status: `200`, `400` (`WEBHOOK_INVALID_URL`,
`WEBHOOK_EMPTY_STATUS_FILTER`, `VALIDATION_ERROR`), `404` (`WEBHOOK_NOT_FOUND`).

Request (campos opcionais):
```json
{ "url": "https://hooks.atlascomercial.com.br/oms/v2/orders", "statusFilter": ["PAID", "SHIPPED", "DELIVERED"], "active": true }
```

Response `200`: o objeto do endpoint atualizado (mesmo shape do `GET`).

### 5.4 `DELETE /api/v1/webhooks/:id`: remover endpoint

Auth: usuário autenticado. Status: `204` (sem corpo), `404` (`WEBHOOK_NOT_FOUND`).

### 5.5 `POST /api/v1/webhooks/:id/rotate-secret`: rotacionar a secret

Auth: usuário autenticado. Status: `200`, `404` (`WEBHOOK_NOT_FOUND`), `409`
(`WEBHOOK_INACTIVE`).

Request: corpo vazio.

Response `200`:
```json
{
  "id": "b2a7e9d0-2222-4b3c-8d4e-000000000002",
  "secret": "whsec_EXEMPLO_nova_secret_apos_rotacao",
  "previousSecretValidUntil": "2026-09-10T12:00:00.000Z"
}
```

A secret anterior continua válida para verificação por 24 horas
([09:21] Sofia; [ADR-004](adrs/ADR-004-assinatura-hmac-sha256-secret-por-endpoint.md)).

### 5.6 `GET /api/v1/webhooks/:id/deliveries`: histórico de entregas

Auth: usuário autenticado. Status: `200`, `404` (`WEBHOOK_NOT_FOUND`). Retorna os últimos
envios (default 100, `pageSize` máx. 100) ([09:34] Marcos).

Response `200`:
```json
{
  "data": [
    {
      "id": "c3b8f0e1-3333-4c4d-9e5f-000000000003",
      "eventId": "d4c9a1f2-4444-4d5e-af60-000000000004",
      "orderId": "aa11bb22-5555-4e6f-b071-000000000005",
      "toStatus": "SHIPPED",
      "attempt": 1,
      "statusCode": 200,
      "durationMs": 143,
      "success": true,
      "error": null,
      "sentAt": "2026-09-09T12:00:02.150Z"
    },
    {
      "id": "e5dab2f3-5555-4e6f-b071-000000000006",
      "eventId": "d4c9a1f2-4444-4d5e-af60-000000000004",
      "orderId": "aa11bb22-5555-4e6f-b071-000000000005",
      "toStatus": "SHIPPED",
      "attempt": 2,
      "statusCode": null,
      "durationMs": 10000,
      "success": false,
      "error": "WEBHOOK_DELIVERY_TIMEOUT",
      "sentAt": "2026-09-09T12:01:02.400Z"
    }
  ],
  "pagination": { "page": 1, "pageSize": 100, "total": 2, "totalPages": 1 }
}
```

### 5.7 `POST /api/v1/admin/webhooks/dead-letter/:id/replay`: reprocessar item da DLQ

Auth: usuário autenticado **com role `ADMIN`** ([09:36] Sofia). Status: `202`, `403`
(`FORBIDDEN`, role insuficiente), `404` (`WEBHOOK_DEAD_LETTER_NOT_FOUND`), `409`
(`WEBHOOK_ALREADY_REPLAYED` se já houver um replay pendente para o mesmo item).

Request: corpo vazio.

Response `202`:
```json
{
  "deadLetterId": "f6ebc3d4-6666-4f70-c182-000000000007",
  "newOutboxEventId": "07fcd4e5-7777-4081-d293-000000000008",
  "status": "pending",
  "replayedBy": "operator-user-uuid",
  "replayedAt": "2026-09-09T15:30:00.000Z"
}
```

### 5.8 Contrato do webhook enviado ao cliente (saída)

O worker faz `POST` para a `url` do endpoint.

Headers:

| Header | Valor | Semântica |
|---|---|---|
| `Content-Type` | `application/json` | corpo JSON ([09:44] Diego) |
| `X-Event-Id` | UUID do evento | único por evento; o cliente **deve** deduplicar por ele ([09:25] Diego) |
| `X-Webhook-Id` | UUID do `webhook_endpoint` | identifica qual cadastro gerou o envio, para clientes com vários ([09:44] Sofia) |
| `X-Signature` | `sha256=<hex>` | HMAC-SHA256 do corpo bruto com a secret do endpoint ([09:20] Sofia) |
| `X-Timestamp` | ISO 8601 do momento do envio | permite ao cliente detectar replay attack ([09:44] Diego) |

Corpo (payload snapshot, sem os itens do pedido, [09:43] Diego):
```json
{
  "eventId": "d4c9a1f2-4444-4d5e-af60-000000000004",
  "eventType": "order.status_changed",
  "timestamp": "2026-09-09T12:00:00.000Z",
  "orderId": "aa11bb22-5555-4e6f-b071-000000000005",
  "orderNumber": "ORD-000123",
  "fromStatus": "PROCESSING",
  "toStatus": "SHIPPED",
  "customerId": "6f1d2c3a-1111-4a2b-9c3d-000000000001",
  "totalCents": 459900
}
```

Resposta esperada do cliente: qualquer `2xx` em até 10 segundos conta como sucesso;
qualquer outra coisa conta como falha.

## 6. Matriz de erros previstos

Todos os códigos do módulo usam o prefixo `WEBHOOK_` ([09:29] Larissa) e herdam de
`AppError` (`src/shared/errors/app-error.ts`).

### 6.1 Erros da API HTTP

| Código | HTTP | Condição | Tratamento |
|---|---|---|---|
| `WEBHOOK_NOT_FOUND` | 404 | `:id` de webhook inexistente | resposta de erro padrão |
| `WEBHOOK_INVALID_URL` | 400 | `url` não começa com `https://` | rejeitado no schema Zod ([09:23] Sofia) |
| `WEBHOOK_EMPTY_STATUS_FILTER` | 400 | `statusFilter` vazio | rejeitado no schema Zod |
| `WEBHOOK_INVALID_STATUS_FILTER` | 400 | valor fora do enum `OrderStatus` | rejeitado no schema Zod contra `src/modules/orders/order.status.ts` |
| `WEBHOOK_CUSTOMER_NOT_FOUND` | 422 | `customerId` não existe em `customers` | `UnprocessableEntityError` no service |
| `WEBHOOK_DUPLICATE_URL` | 409 | já existe endpoint ativo com a mesma `url` para o mesmo `customerId` | `ConflictError` no service |
| `WEBHOOK_INACTIVE` | 409 | rotacionar secret de endpoint `active = false` | `ConflictError` no service |
| `WEBHOOK_SECRET_REQUIRED` | 400 | verificação interna: tentativa de envio sem secret vigente | não deveria ocorrer; logado em `ERROR` e o evento vai para retry |
| `WEBHOOK_DEAD_LETTER_NOT_FOUND` | 404 | `:id` de item de DLQ inexistente | resposta de erro padrão |
| `WEBHOOK_ALREADY_REPLAYED` | 409 | replay pedido para item que já tem evento pendente originado de replay | `ConflictError` no service |
| `FORBIDDEN` | 403 | replay chamado por usuário sem role `ADMIN` | tratado por `requireRole('ADMIN')` (erro já existente) |

### 6.2 Erros de processamento no worker (não são respostas HTTP; viram estado + log + métrica)

| Código | Condição | Tratamento |
|---|---|---|
| `WEBHOOK_DELIVERY_TIMEOUT` | cliente não respondeu em 10 s | conta como falha; `attempts++`; retry ou DLQ (§4.3, §4.4) |
| `WEBHOOK_DELIVERY_HTTP_ERROR` | resposta não 2xx | idem; `status_code` registrado em `webhook_delivery` |
| `WEBHOOK_DELIVERY_CONNECTION_ERROR` | DNS, TLS ou conexão recusada | idem |
| `WEBHOOK_PAYLOAD_TOO_LARGE` | payload renderizado excede 64 KB | erro lançado em `publishWebhookEvent` **dentro da transação** de `changeStatus`; a transação sofre rollback ([09:24] Larissa) |
| `WEBHOOK_SIGNATURE_ERROR` | falha ao calcular o HMAC (secret ausente ou inválida) | logado em `ERROR`; evento vai para retry |

## 7. Estratégias de resiliência

- **Timeout:** 10 segundos por tentativa de envio ([09:42] Diego). Sem resposta nesse
  prazo, a conexão é abortada e conta como falha.
- **Retry:** 5 tentativas no total, backoff exponencial fixo `1min / 5min / 30min / 2h /
  12h` ([09:17] Diego). O agendamento é por `next_attempt_at` na própria linha da outbox;
  não há timer em memória, então um restart do worker não perde retries.
- **DLQ:** após a 5ª falha, o evento vai para `webhook_dead_letter` e sai do ciclo do
  worker. Reprocessamento só manual, via endpoint administrativo (§5.7)
  ([ADR-003](adrs/ADR-003-retry-com-backoff-e-dlq.md)).
- **At-least-once:** se o worker cair depois de enviar mas antes de marcar `delivered`, o
  evento volta a `pending` (via um lease/timeout no `processing`) e será reenviado. O
  cliente deduplica por `X-Event-Id`
  ([ADR-005](adrs/ADR-005-entrega-at-least-once-com-x-event-id.md)).
- **Isolamento da transação de negócio:** nenhuma chamada de rede acontece dentro da
  `$transaction` de `changeStatus`; só a escrita das linhas da outbox
  ([ADR-001](adrs/ADR-001-outbox-no-mysql.md)).
- **Invariantes:**
  - Um evento registrado na outbox é entregue pelo menos uma vez ou termina em DLQ.
  - `attempts` nunca passa de 5 na `webhook_outbox`.
  - A ordem de entrega por `order_id` é a ordem de `created_at` enquanto o worker for único.

## 8. Observabilidade

Reusa o `logger` Pino existente (`src/shared/logger/index.ts`), sem nada novo ([09:29]
Bruno).

**Métricas** (contadores e histogramas expostos pelo worker e pela API):

- `webhook_outbox_pending` (gauge): eventos com `status = pending`.
- `webhook_events_published_total{to_status}`: inserções na outbox.
- `webhook_delivery_attempts_total{result}`: `result` em `success | http_error | timeout | connection_error`.
- `webhook_delivery_duration_ms` (histograma): tempo de cada `POST` de saída.
- `webhook_events_dead_lettered_total`: eventos que chegaram à DLQ.
- `webhook_replay_total`: replays administrativos executados.

**Logs estruturados** (eventos nomeados, sem vazar `secret` nem o corpo assinado):

- `webhook_event_published`: `{ eventId, webhookEndpointId, orderId, toStatus }`.
- `webhook_delivery_attempt`: `{ eventId, webhookEndpointId, attempt, statusCode, durationMs, result }`.
- `webhook_event_dead_lettered`: `{ eventId, webhookEndpointId, lastError }`.
- `webhook_replay_requested`: `{ deadLetterId, newOutboxEventId, replayedBy }` ([09:36] Sofia).
- Erros de envio em nível `WARN`; erros internos (secret ausente, payload grande) em `ERROR`.

**Tracing / correlação:** o `X-Request-Id` do `requestLogger`
(`src/middlewares/request-logger.middleware.ts`) que originou a mudança de status é
copiado para a linha da outbox e propagado nos logs de entrega, ligando a requisição
`PATCH /orders/:id/status` ao envio do webhook. O `eventId` correlaciona a linha da outbox,
as linhas de `webhook_delivery` e, se aplicável, a linha da DLQ.

## 9. Dependências e compatibilidade

| Componente | Versão mínima | Observação |
|---|---|---|
| Node.js | 20 | mesma do projeto (`package.json` `engines`) |
| Prisma | 5.22 | modelos novos na mesma `schema.prisma`; migration via `prisma migrate dev` |
| MySQL | 8 | outbox e DLQ como tabelas InnoDB; índices em `(status, next_attempt_at)` e `created_at` |
| Cliente HTTP de saída | a definir na implementação | o projeto não tem um cliente HTTP padronizado hoje; a escolha vira convenção do módulo ([ADR-006](adrs/ADR-006-reuso-dos-padroes-do-projeto.md)) |

Compatibilidade:

- Os contratos HTTP existentes (`/auth`, `/users`, `/customers`, `/products`, `/orders`)
  **não mudam**. `PATCH /orders/:id/status` ganha o efeito colateral de publicar eventos,
  mas a request e a response permanecem idênticas.
- O middleware de erro central (`src/middlewares/error.middleware.ts`) trata os erros
  `WEBHOOK_*` sem alteração, porque herdam de `AppError` ([09:29] Bruno).
- O worker é um processo adicional; não afeta o deploy da API além de exigir um segundo
  serviço rodando `npm run worker`.

## 10. Critérios de aceite técnicos

- [ ] Mudar o status de um pedido cujo customer tem endpoint ativo para aquele status cria
      N linhas em `webhook_outbox` (uma por endpoint alvo), dentro da transação.
- [ ] Se a inserção na outbox falha, `PATCH /orders/:id/status` retorna erro e o status do
      pedido **não** muda (rollback verificado em teste de integração).
- [ ] Mudança de status sem endpoint interessado não escreve nada na outbox.
- [ ] O worker entrega um evento em menos de 3 segundos quando o endpoint responde `200`
      (2 s de polling + envio).
- [ ] Endpoint que responde `500` é retentado exatamente 5 vezes, com os intervalos
      `1min / 5min / 30min / 2h / 12h`, e então vai para `webhook_dead_letter`.
- [ ] `X-Signature` verificável com a secret retornada na criação; após rotação, tanto a
      secret nova quanto a anterior verificam por 24 horas.
- [ ] `url` `http://` é rejeitada com `WEBHOOK_INVALID_URL` (400).
- [ ] Payload que excederia 64 KB causa `WEBHOOK_PAYLOAD_TOO_LARGE` e rollback.
- [ ] `GET /webhooks/:id/deliveries` retorna as tentativas com `attempt`, `statusCode`,
      `durationMs`, `success`.
- [ ] `POST /admin/webhooks/dead-letter/:id/replay` sem role `ADMIN` retorna `403`.
- [ ] Replay cria um novo evento pendente com novo `eventId` e registra `replayedBy` no log.
- [ ] Nenhum teste existente de `orders`, `auth`, `customers`, `products` quebra.

## 11. Riscos e mitigação

### A escrita extra na transação de `changeStatus` aumenta o tempo de lock

- **Probabilidade:** média. **Impacto:** contenção em picos de mudança de status.
- **Mitigação:**
  - A consulta de endpoints interessados usa índice por `(customer_id, active)` e o filtro
    de status é resolvido em memória.
  - As inserções na outbox são `INSERT` simples, sem `SELECT ... FOR UPDATE`.
  - Métrica `webhook_delivery_duration_ms` e o tempo da transação de `changeStatus` são
    monitorados; se subir, avaliar mover a renderização do payload para fora do lock.
- **Contingência:** feature flag para desligar a publicação de eventos (a `publishWebhookEvent`
  vira no-op) sem redeploy da API.

### Um cliente com muitos pedidos mudando ao mesmo tempo recebe muitas chamadas

- **Probabilidade:** média. **Impacto:** o worker "bombardeia" o endpoint do cliente.
- **Mitigação:**
  - Métrica `webhook_delivery_attempts_total` por `webhook_endpoint_id` para detectar o caso.
  - Rate limiting de saída fica registrado como questão em aberto no RFC, a ser
    implementado se a métrica indicar necessidade ([09:39] Larissa).
- **Contingência:** desativar (`active = false`) o endpoint do cliente afetado
  temporariamente.

### Secret vazada em log do lado do cliente

- **Probabilidade:** baixa, mas já ocorreu ([09:22] Diego). **Impacto:** um terceiro pode
  forjar eventos para aquele endpoint.
- **Mitigação:**
  - Secret única por endpoint limita o raio de exposição a um cliente
    ([ADR-004](adrs/ADR-004-assinatura-hmac-sha256-secret-por-endpoint.md)).
  - `POST /webhooks/:id/rotate-secret` com grace period de 24 horas permite troca sem
    downtime.
  - A plataforma nunca loga a `secret` nem o corpo assinado (§8).
- **Contingência:** rotação imediata e, se necessário, desativação do endpoint até o
  cliente confirmar a correção.

## 12. Integração com o sistema existente

Esta seção nomeia os arquivos reais do código base e descreve a integração de cada. Ver
`.claude/references/codebase/integration-points.md`.

### `src/modules/orders/order.service.ts`

`OrderService.changeStatus` roda a mudança de status dentro de `prisma.$transaction`. A
integração adiciona **uma chamada** ao final desse bloco, ainda dentro do `tx`:
`await publishWebhookEvent(tx, refreshed, from, to)`. `publishWebhookEvent` é uma **função
pura** que recebe o `Prisma.TransactionClient`, não um repository injetado ([09:41] Diego).
Ela lê `webhook_endpoint` e escreve `webhook_outbox` usando o mesmo `tx`, de modo que a
inserção dos eventos é atômica com a mudança de status. Se ela lançar, o `$transaction`
reverte tudo, inclusive `order.update` e `orderStatusHistory.create`.

### `src/modules/orders/order.status.ts`

O enum `OrderStatus` e nada mais. O schema Zod de `statusFilter` (no módulo de webhooks)
valida cada valor contra esse enum via `z.nativeEnum(OrderStatus)`, exatamente como
`src/modules/orders/order.schemas.ts` já faz em `updateOrderStatusSchema`. O filtro de
interesse do endpoint compara `toStatus` da transição com a lista `statusFilter`.

### `src/shared/errors/app-error.ts` e `src/shared/errors/http-errors.ts`

As classes de erro do módulo de webhooks herdam de `AppError` ou reusam
`ConflictError` / `UnprocessableEntityError` / `NotFoundError` com um `code` próprio
prefixado `WEBHOOK_` (por exemplo, `class WebhookNotFoundError extends NotFoundError` com
`code = 'WEBHOOK_NOT_FOUND'`). Seguem o padrão de `InvalidStatusTransitionError` e
`InsufficientStockError`, que já são subclasses especializadas ([09:28] Bruno).

### `src/middlewares/error.middleware.ts`

**Nenhuma alteração.** O middleware já serializa qualquer `AppError` para
`{ error: { code, message, details? } }` e já trata `ZodError` (usado pelos schemas de
webhook) e `Prisma.PrismaClientKnownRequestError` (P2002 cobre a constraint única de `url`
por customer, se a implementação optar por delegar ao banco em vez de checar no service)
([09:29] Bruno).

### `src/middlewares/auth.middleware.ts`

As rotas de configuração de webhook usam `authenticate` no router, como
`src/modules/customers/customer.routes.ts`. A rota de replay de DLQ encadeia
`authenticate` e `requireRole('ADMIN')`, exatamente como
`src/modules/users/user.routes.ts` faz em `GET /users/:id` ([09:36] Larissa). O
`replayedBy` do log vem de `req.user.id`.

### `src/middlewares/validate.middleware.ts`

Os schemas Zod de criação e edição de webhook são passados a `validate({ body, params, query })`
no router, como nos outros módulos. A regra "URL tem que ser https" é um `.refine()` no
schema, não lógica de service ([09:23] Sofia).

### `src/config/database.ts`

Exporta o singleton `prisma` usado pela API. O worker **não** importa esse singleton: ele
instancia o **próprio** `PrismaClient` em `src/worker.ts`, com a mesma `DATABASE_URL`,
porque `PrismaClient` é por processo ([09:30] Bruno). O padrão de criação é o mesmo de
`createPrismaClient()`.

### `src/server.ts`

Modelo para o novo entrypoint `src/worker.ts`: bootstrap assíncrono, tratamento de
`SIGINT` / `SIGTERM` para encerrar o loop de polling de forma limpa e `prisma.$disconnect()`
no shutdown ([09:11] Larissa). Um script `"worker": "tsx watch --env-file=.env src/worker.ts"`
(e o equivalente de produção) entra no `package.json`.

### `src/app.ts`

`buildControllers(prisma)` instancia o `WebhookController` (repository, depois service,
depois controller) e o adiciona ao objeto `Controllers`. `buildApiRouter` monta
`router.use('/webhooks', buildWebhookRouter(controllers.webhooks))` e
`router.use('/admin/webhooks', buildWebhookAdminRouter(controllers.webhooks))`, seguindo o
padrão de `src/routes/index.ts`.

### `src/shared/logger/index.ts`

O `logger` Pino é importado tanto pela API quanto pelo worker. Os eventos nomeados de §8
seguem o estilo `snake_case` com objeto de contexto primeiro, como `http_request` e
`server_started`. A lista de `redactPaths` já cobre `*.token` e `authorization`; a
implementação deve garantir que `secret` e o corpo assinado nunca entrem em um campo logado.

### `prisma/schema.prisma`

Quatro modelos novos, com as convenções do arquivo (PK `String @id @default(uuid())
@db.Char(36)`, `@@map` snake_case plural, timestamps, `@@index` nos campos de filtro):
`webhook_endpoint` (`customerId`, `url`, `secret`, `previousSecret`,
`previousSecretValidUntil`, `active`, `statusFilter Json`), `webhook_outbox`
(`webhookEndpointId`, `orderId`, `eventId`, `payload Json`, `status`, `attempts`,
`nextAttemptAt`, `createdAt`, `deliveredAt`), `webhook_delivery` (`outboxId`, `attempt`,
`statusCode`, `responseBody`, `durationMs`, `error`, `sentAt`), `webhook_dead_letter`
(`outboxId`, `webhookEndpointId`, `payload Json`, `lastError`, `failedAt`).

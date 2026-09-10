# RFC: Sistema de Webhooks de Notificação de Pedidos

| | |
|---|---|
| Autor | Time de Plataforma / Pedidos |
| Status | Em revisão |
| Data | 2026-09-10 |
| Revisores | Larissa (Tech Lead), Marcos (Product Manager), Bruno (Engenheiro Pleno, time de Pedidos), Diego (Engenheiro Sênior, time de Plataforma), Sofia (Engenheira de Segurança) |

---

## Resumo executivo (TL;DR)

Três clientes B2B fazem polling no `GET /orders` para saber quando o status de um pedido
muda, o que é lento e caro para eles. Propomos entregar essa notificação por **webhooks
outbound**: ao mudar o status do pedido, um evento é gravado numa tabela outbox **dentro da
mesma transação** de `changeStatus`; um **worker em processo separado** faz polling a cada
2 segundos e entrega o evento por HTTP, assinado com **HMAC-SHA256**. Falhas são retentadas
com backoff exponencial (5 tentativas) e, esgotadas, vão para uma **dead letter queue** com
replay manual. A garantia de entrega é **at-least-once**, com `X-Event-Id` para o cliente
deduplicar.

Ficam em aberto: rate limiting de saída para não bombardear um cliente, e a estratégia de
escalar para múltiplos workers no futuro.

## Contexto e problema

O status de um pedido só muda em
[`OrderService.changeStatus`](../src/modules/orders/order.service.ts#L126), que já roda uma
transação que atualiza `orders`, insere em `order_status_history` e ajusta o estoque dos
produtos. O sistema não tem hoje nenhum mecanismo de notificação externa, evento, fila ou
worker ([`CLAUDE.md`](../CLAUDE.md), seção "Ausências").

Restrições reais que moldaram a proposta:

- A notificação não pode acoplar a latência nem a disponibilidade da mudança de status a um
  endpoint HTTP de terceiro.
- Não pode haver o caso de o status mudar e o evento não ser registrado.
- O time é pequeno e evita operar infraestrutura nova.
- Os clientes consideram "tempo real" qualquer latência **abaixo de 10 segundos**.
- A Atlas Comercial espera a entrega para o fim de novembro; a estimativa é de 3 sprints,
  com a revisão de segurança da Sofia incluída no fim.

## Proposta técnica

Visão de arquitetura, sem descer ao detalhe de implementação (esse fica no [FDD](FDD.md)).

- **Outbox no MySQL.** Uma tabela `webhook_outbox` recebe uma linha de evento a cada
  mudança de status, na mesma transação SQL. O evento guarda o payload já renderizado
  (snapshot). Ver [ADR-001](adrs/ADR-001-outbox-no-mysql.md) e
  [ADR-007](adrs/ADR-007-snapshot-do-payload-na-insercao.md).
- **Configuração de webhook.** Uma tabela guarda, por endpoint do cliente, a `url` (https),
  a `secret`, o `customer_id`, o estado ativo e a lista de status que aquele endpoint quer
  receber. O CRUD dessa configuração é exposto na API, autenticado com o JWT do próprio
  sistema. O filtro de status é aplicado **na inserção** da outbox: se nenhum endpoint do
  customer quer aquele status, o evento nem é gravado.
- **Worker de entrega.** Um processo separado (`src/worker.ts` mais `npm run worker`), com
  `PrismaClient` próprio no mesmo banco, faz polling a cada 2 segundos, seleciona os
  eventos pendentes mais antigos, monta a requisição HTTP e envia. Ver
  [ADR-002](adrs/ADR-002-worker-separado-em-polling.md).
- **Assinatura e identidade.** Cada envio leva os headers `X-Signature` (HMAC-SHA256 do
  corpo com a secret do endpoint), `X-Event-Id` (UUID do evento), `X-Timestamp` e
  `X-Webhook-Id`. A secret é única por endpoint e rotacionável, com grace period de 24
  horas. Ver [ADR-004](adrs/ADR-004-assinatura-hmac-sha256-secret-por-endpoint.md).
- **Resiliência.** Timeout de 10 segundos por envio. Falha aciona retry com backoff
  exponencial de 5 tentativas (1m, 5m, 30m, 2h, 12h). Esgotadas as tentativas, o evento vai
  para `webhook_dead_letter`, com replay manual por um endpoint administrativo (role ADMIN).
  Ver [ADR-003](adrs/ADR-003-retry-com-backoff-e-dlq.md).
- **Garantia de entrega.** At-least-once; o cliente deduplica por `X-Event-Id`. Ver
  [ADR-005](adrs/ADR-005-entrega-at-least-once-com-x-event-id.md).
- **Reuso.** O módulo `src/modules/webhooks` segue o padrão dos demais domínios; erros
  herdam de `AppError` com prefixo `WEBHOOK_`; logger Pino e middleware de erro central sem
  alteração. Ver [ADR-006](adrs/ADR-006-reuso-dos-padroes-do-projeto.md).

## Alternativas consideradas

### Disparo síncrono de HTTP dentro de `changeStatus`

Chamar o endpoint do cliente diretamente no
[`OrderService.changeStatus`](../src/modules/orders/order.service.ts#L126), dentro ou logo
após a transação.

**Trade-off que motivou o descarte:** um cliente lento travaria a mudança de status de
outros pedidos, e um cliente fora do ar forçaria a escolha entre dar rollback numa mudança
de status legítima ou perder o evento.

### Redis Streams / Redis Cluster como fila de eventos

Publicar os eventos numa fila em Redis e ter consumidores lendo dela.

**Trade-off que motivou o descarte:** exigiria subir e operar um Redis Cluster, considerado
overengineering para o tamanho do time. O outbox no MySQL que já existe resolve o problema
sem essa dependência.

### Trigger de banco para acordar o worker

Usar um trigger no MySQL para notificar o worker de forma reativa, em vez de polling.

**Trade-off que motivou o descarte:** o MySQL não tem `LISTEN/NOTIFY`; um trigger só
executa SQL e não notifica processo externo. Improvisar seria frágil, e o polling de 2
segundos já cabe folgado no requisito de 10 segundos.

## Questões em aberto

- **Rate limiting de saída.** Se um cliente tem 50 pedidos mudando de status em um minuto,
  a plataforma faz 50 chamadas para o endpoint dele. Ficou como "observar e decidir
  depois"; implementar apenas se virar problema.
- **Escalar para múltiplos workers.** Enquanto houver um único worker, a ordenação por
  `order_id` é garantida de graça. Escalar para vários workers (particionando por
  `order_id` ou com lock pessimista) foi tratado como problema do futuro, sem decisão.
- **Endurecimento do RBAC do CRUD de webhooks.** Por ora, qualquer usuário autenticado pode
  gerenciar a configuração de webhook; só o replay de DLQ exige ADMIN. A equipe de
  segurança deixou para endurecer mais pra frente.

## Impacto e riscos

- **Impacto no código existente.** A única alteração de comportamento no fluxo atual é em
  [`OrderService.changeStatus`](../src/modules/orders/order.service.ts#L126), que passa a
  publicar o evento na outbox dentro da transação via uma função pura
  `publishWebhookEvent(tx, order, fromStatus, toStatus)`. O CRUD atual
  de pedidos, clientes e produtos não muda. Detalhe no [FDD](FDD.md), seção "Integração com
  o sistema existente".
- **Novo processo em produção.** O worker é um segundo processo para operar, monitorar e
  fazer deploy.
- **Latência mínima de 2 segundos** antes da primeira tentativa de entrega, por conta do
  polling.
- **Superfície de segurança.** Geração e armazenamento de secrets e a implementação do HMAC
  precisam de revisão dedicada da Sofia antes do deploy, com pelo menos dois dias úteis
  reservados.
- **Crescimento da outbox.** A tabela cresce com o volume de mudanças de status; o
  arquivamento de linhas entregues ficou fora do escopo desta feature.

## Decisões relacionadas

- [ADR-001: Padrão Outbox no MySQL](adrs/ADR-001-outbox-no-mysql.md)
- [ADR-002: Worker em processo separado, polling a cada 2 segundos](adrs/ADR-002-worker-separado-em-polling.md)
- [ADR-003: Retry com backoff exponencial (5 tentativas) e DLQ em tabela separada](adrs/ADR-003-retry-com-backoff-e-dlq.md)
- [ADR-004: Assinatura HMAC-SHA256 com secret única por endpoint](adrs/ADR-004-assinatura-hmac-sha256-secret-por-endpoint.md)
- [ADR-005: Entrega at-least-once com dedupe do lado do cliente via X-Event-Id](adrs/ADR-005-entrega-at-least-once-com-x-event-id.md)
- [ADR-006: Reuso dos padrões existentes do projeto](adrs/ADR-006-reuso-dos-padroes-do-projeto.md)
- [ADR-007: Snapshot do payload renderizado na inserção da outbox](adrs/ADR-007-snapshot-do-payload-na-insercao.md)

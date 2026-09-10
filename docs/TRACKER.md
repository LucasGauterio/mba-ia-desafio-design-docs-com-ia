# Tracker de Rastreabilidade

Referência cruzada de cada item registrado nos documentos com sua origem na transcrição
([`TRANSCRICAO.md`](../TRANSCRICAO.md), formato `[hh:mm] Nome`) ou no código (caminho de
arquivo real).

Este é o **único** lugar do pacote onde a origem de cada item é materializada: PRD, RFC,
FDD e ADRs têm prosa limpa, sem `[hh:mm]`. Serve de defesa contra alucinação: item sem
origem localizável foi corrigido ou removido do documento antes de esta tabela ser fechada.

Toda célula `Documento` e `Localização` é link relativo (a partir de `docs/`). Linhas
`TRANSCRICAO` levam a âncora da linha da fala em `TRANSCRICAO.md`; linhas `CODIGO` levam a
âncora da linha do símbolo citado.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|---|---|---|---|---|---|
| PRD-CTX-01 | [docs/PRD.md](PRD.md) | Contexto | Três clientes B2B (Atlas, MaxDistribuição, Nova Cargo) pediram notificação de mudança de status | TRANSCRICAO | [[09:00] Marcos](../TRANSCRICAO.md#L18) |
| PRD-CTX-02 | [docs/PRD.md](PRD.md) | Problema | Clientes fazem polling no GET /orders, integração lenta e cara | TRANSCRICAO | [[09:00] Marcos](../TRANSCRICAO.md#L18) |
| PRD-CTX-03 | [docs/PRD.md](PRD.md) | Restrição | Atlas pode migrar para concorrente se não entregar até fim do trimestre | TRANSCRICAO | [[09:00] Marcos](../TRANSCRICAO.md#L18) |
| PRD-CTX-04 | [docs/PRD.md](PRD.md) | Requisito Não Funcional | "Tempo real" para o cliente = abaixo de 10 segundos | TRANSCRICAO | [[09:02] Marcos](../TRANSCRICAO.md#L22) |
| PRD-CTX-05 | [docs/PRD.md](PRD.md) | Restrição | Só webhooks outbound (plataforma envia, cliente recebe) | TRANSCRICAO | [[09:02] Marcos](../TRANSCRICAO.md#L26) |
| PRD-OBJ-01 | [docs/PRD.md](PRD.md) | Métrica | Latência do commit à 1ª tentativa de entrega abaixo de 10 s no caminho feliz | TRANSCRICAO | [[09:02] Marcos](../TRANSCRICAO.md#L22) |
| PRD-OBJ-02 | [docs/PRD.md](PRD.md) | Métrica | 100% das mudanças de status confirmadas geram notificação registrada | TRANSCRICAO | [[09:06] Diego](../TRANSCRICAO.md#L48) |
| PRD-OBJ-03 | [docs/PRD.md](PRD.md) | Métrica | Janela de retry: 5 tentativas em ~15 h (1min/5min/30min/2h/12h) | TRANSCRICAO | [[09:17] Diego](../TRANSCRICAO.md#L104) |
| PRD-OBJ-04 | [docs/PRD.md](PRD.md) | Métrica | Prazo: fim de novembro, 3 sprints com revisão de segurança | TRANSCRICAO | [[09:45] Marcos](../TRANSCRICAO.md#L270) |
| PRD-FR-01 | [docs/PRD.md](PRD.md) | Requisito Funcional | Cadastrar endpoint de webhook (POST), secret gerada pela plataforma e devolvida | TRANSCRICAO | [[09:31] Marcos](../TRANSCRICAO.md#L184) |
| PRD-FR-01b | [docs/PRD.md](PRD.md) | Restrição | customer_id vem do body ou path, não do JWT (JWT é do operador) | TRANSCRICAO | [[09:32] Larissa](../TRANSCRICAO.md#L190) |
| PRD-FR-02 | [docs/PRD.md](PRD.md) | Requisito Funcional | Editar endpoint de webhook (PATCH) | TRANSCRICAO | [[09:33] Bruno](../TRANSCRICAO.md#L192) |
| PRD-FR-03 | [docs/PRD.md](PRD.md) | Requisito Funcional | Remover endpoint de webhook (DELETE) | TRANSCRICAO | [[09:33] Bruno](../TRANSCRICAO.md#L192) |
| PRD-FR-04 | [docs/PRD.md](PRD.md) | Requisito Funcional | Listar webhooks de um customer (GET) | TRANSCRICAO | [[09:33] Bruno](../TRANSCRICAO.md#L192) |
| PRD-FR-05 | [docs/PRD.md](PRD.md) | Requisito Funcional | Filtro de status de interesse por endpoint | TRANSCRICAO | [[09:33] Marcos](../TRANSCRICAO.md#L194) |
| PRD-FR-05b | [docs/PRD.md](PRD.md) | Decisão | Filtro aplicado na inserção da outbox; sem endpoint interessado, não insere | TRANSCRICAO | [[09:34] Bruno](../TRANSCRICAO.md#L198) |
| PRD-FR-06 | [docs/PRD.md](PRD.md) | Requisito Funcional | Rotacionar secret; antiga válida 24 h em paralelo | TRANSCRICAO | [[09:21] Sofia](../TRANSCRICAO.md#L130) |
| PRD-FR-07 | [docs/PRD.md](PRD.md) | Requisito Funcional | Entregar notificação assinada de forma assíncrona | TRANSCRICAO | [[09:09] Diego](../TRANSCRICAO.md#L60) |
| PRD-FR-08 | [docs/PRD.md](PRD.md) | Requisito Funcional | Retry 5x com backoff e fila de falhas | TRANSCRICAO | [[09:15] Diego](../TRANSCRICAO.md#L92) |
| PRD-FR-09 | [docs/PRD.md](PRD.md) | Requisito Funcional | Identificador único de evento; cliente deduplica | TRANSCRICAO | [[09:25] Diego](../TRANSCRICAO.md#L150) |
| PRD-FR-10 | [docs/PRD.md](PRD.md) | Requisito Funcional | Histórico das últimas ~100 entregas de um endpoint | TRANSCRICAO | [[09:34] Marcos](../TRANSCRICAO.md#L202) |
| PRD-FR-11 | [docs/PRD.md](PRD.md) | Requisito Funcional | Replay manual de item da fila de falhas por administrador | TRANSCRICAO | [[09:18] Diego](../TRANSCRICAO.md#L114) |
| PRD-FR-11b | [docs/PRD.md](PRD.md) | Restrição | Replay exige role ADMIN e é auditado | TRANSCRICAO | [[09:36] Sofia](../TRANSCRICAO.md#L210) |
| PRD-FR-12 | [docs/PRD.md](PRD.md) | Requisito Funcional | Evento registrado na transação da mudança de status; falha reverte o status | TRANSCRICAO | [[09:40] Bruno](../TRANSCRICAO.md#L238) |
| PRD-NFR-01 | [docs/PRD.md](PRD.md) | Requisito Não Funcional | Notificação em menos de 10 s no caminho feliz | TRANSCRICAO | [[09:02] Marcos](../TRANSCRICAO.md#L22) |
| PRD-NFR-02 | [docs/PRD.md](PRD.md) | Requisito Não Funcional | Pior caso da fila: 2 s de polling mais tempo de envio | TRANSCRICAO | [[09:10] Larissa](../TRANSCRICAO.md#L68) |
| PRD-NFR-03 | [docs/PRD.md](PRD.md) | Requisito Não Funcional | Confiabilidade: commit implica evento registrado; rollback implica evento inexistente | TRANSCRICAO | [[09:41] Diego](../TRANSCRICAO.md#L240) |
| PRD-NFR-04 | [docs/PRD.md](PRD.md) | Requisito Não Funcional | Garantia at-least-once | TRANSCRICAO | [[09:24] Diego](../TRANSCRICAO.md#L146) |
| PRD-NFR-05 | [docs/PRD.md](PRD.md) | Requisito Não Funcional | Ordem garantida por pedido, não global | TRANSCRICAO | [[09:12] Diego](../TRANSCRICAO.md#L80) |
| PRD-NFR-06 | [docs/PRD.md](PRD.md) | Requisito Não Funcional | Assinatura HMAC-SHA256 sobre o corpo | TRANSCRICAO | [[09:20] Sofia](../TRANSCRICAO.md#L120) |
| PRD-NFR-07 | [docs/PRD.md](PRD.md) | Requisito Não Funcional | Secret única por endpoint, nunca global | TRANSCRICAO | [[09:21] Sofia](../TRANSCRICAO.md#L126) |
| PRD-NFR-08 | [docs/PRD.md](PRD.md) | Requisito Não Funcional | URL de destino tem que usar HTTPS | TRANSCRICAO | [[09:23] Sofia](../TRANSCRICAO.md#L138) |
| PRD-NFR-09 | [docs/PRD.md](PRD.md) | Requisito Não Funcional | Corpo da notificação não pode passar de 64 KB; erro em vez de truncar | TRANSCRICAO | [[09:24] Larissa](../TRANSCRICAO.md#L144) |
| PRD-NFR-10 | [docs/PRD.md](PRD.md) | Requisito Não Funcional | Timeout de 10 s por tentativa de envio | TRANSCRICAO | [[09:42] Diego](../TRANSCRICAO.md#L250) |
| PRD-NFR-11 | [docs/PRD.md](PRD.md) | Requisito Não Funcional | Nunca logar a secret nem o corpo assinado | TRANSCRICAO | [[09:22] Diego](../TRANSCRICAO.md#L132) |
| PRD-NFR-12 | [docs/PRD.md](PRD.md) | Requisito Não Funcional | Revisão de segurança antes do deploy, 2 dias úteis reservados | TRANSCRICAO | [[09:46] Sofia](../TRANSCRICAO.md#L274) |
| PRD-NFR-13 | [docs/PRD.md](PRD.md) | Requisito Não Funcional | Reuso da stack: sem libs novas de erro, log ou validação | TRANSCRICAO | [[09:29] Bruno](../TRANSCRICAO.md#L174) |
| PRD-NFR-14 | [docs/PRD.md](PRD.md) | Restrição | Contratos HTTP existentes de pedidos, clientes e produtos não mudam | CODIGO | [src/routes/index.ts#L21](../src/routes/index.ts#L21) |
| PRD-DEP-01 | [docs/PRD.md](PRD.md) | Dependência | Alteração no fluxo de mudança de status para registrar o evento | TRANSCRICAO | [[09:40] Bruno](../TRANSCRICAO.md#L238) |
| PRD-DEP-02 | [docs/PRD.md](PRD.md) | Dependência | Novo processo de entrega em produção | TRANSCRICAO | [[09:11] Diego](../TRANSCRICAO.md#L70) |
| PRD-DEP-03 | [docs/PRD.md](PRD.md) | Dependência | Janela de revisão de segurança (Sofia) | TRANSCRICAO | [[09:46] Sofia](../TRANSCRICAO.md#L274) |
| PRD-DEP-04 | [docs/PRD.md](PRD.md) | Dependência | Endpoints HTTPS dos clientes com verificação de assinatura e dedupe | TRANSCRICAO | [[09:26] Marcos](../TRANSCRICAO.md#L156) |
| PRD-OOS-01 | [docs/PRD.md](PRD.md) | Restrição | Fora de escopo: e-mail de alerta ao cliente quando o webhook falha | TRANSCRICAO | [[09:37] Larissa](../TRANSCRICAO.md#L220) |
| PRD-OOS-02 | [docs/PRD.md](PRD.md) | Restrição | Fora de escopo: dashboard/painel visual | TRANSCRICAO | [[09:40] Larissa](../TRANSCRICAO.md#L234) |
| PRD-OOS-03 | [docs/PRD.md](PRD.md) | Restrição | Fora de escopo: garantia de ordem global | TRANSCRICAO | [[09:13] Larissa](../TRANSCRICAO.md#L86) |
| PRD-OOS-04 | [docs/PRD.md](PRD.md) | Restrição | Fora de escopo: rate limiting de saída (observar e decidir depois) | TRANSCRICAO | [[09:39] Larissa](../TRANSCRICAO.md#L230) |
| PRD-OOS-05 | [docs/PRD.md](PRD.md) | Restrição | Fora de escopo: arquivamento de notificações entregues | TRANSCRICAO | [[09:08] Diego](../TRANSCRICAO.md#L56) |
| PRD-OOS-06 | [docs/PRD.md](PRD.md) | Restrição | Fora de escopo: webhooks de entrada | TRANSCRICAO | [[09:03] Sofia](../TRANSCRICAO.md#L28) |
| PRD-OOS-07 | [docs/PRD.md](PRD.md) | Restrição | Fora de escopo: replay automático da fila de falhas e múltiplos workers | TRANSCRICAO | [[09:13] Diego](../TRANSCRICAO.md#L84) |
| PRD-RISK-01 | [docs/PRD.md](PRD.md) | Risco | Escrita extra na mudança de status aumenta contenção em picos | TRANSCRICAO | [[09:04] Bruno](../TRANSCRICAO.md#L32) |
| PRD-RISK-02 | [docs/PRD.md](PRD.md) | Risco | Cliente com muitas mudanças simultâneas recebe rajada de chamadas | TRANSCRICAO | [[09:38] Diego](../TRANSCRICAO.md#L224) |
| PRD-RISK-03 | [docs/PRD.md](PRD.md) | Risco | Secret vazada em log do lado do cliente | TRANSCRICAO | [[09:22] Diego](../TRANSCRICAO.md#L132) |
| PRD-RISK-04 | [docs/PRD.md](PRD.md) | Risco | Prazo de 3 sprints não cobrir a revisão de segurança | TRANSCRICAO | [[09:47] Larissa](../TRANSCRICAO.md#L276) |
| PRD-TEST-01 | [docs/PRD.md](PRD.md) | Trade-off | Revisão de segurança guiada por roteiro antes do deploy | TRANSCRICAO | [[09:46] Sofia](../TRANSCRICAO.md#L274) |
| PRD-TEST-02 | [docs/PRD.md](PRD.md) | Restrição | Nenhum teste existente de pedidos/auth/clientes/produtos pode quebrar | CODIGO | [tests/orders.test.ts](../tests/orders.test.ts) |
| RFC-CTX-01 | [docs/RFC.md](RFC.md) | Contexto | Status só muda em OrderService.changeStatus, dentro de transação | CODIGO | [src/modules/orders/order.service.ts#L126](../src/modules/orders/order.service.ts#L126) |
| RFC-CTX-02 | [docs/RFC.md](RFC.md) | Contexto | Sistema não tem hoje notificação externa, evento, fila ou worker | CODIGO | [src/app.ts#L55](../src/app.ts#L55) |
| RFC-CTX-03 | [docs/RFC.md](RFC.md) | Restrição | Notificação não pode acoplar latência/disponibilidade da mudança de status a HTTP externo | TRANSCRICAO | [[09:04] Bruno](../TRANSCRICAO.md#L32) |
| RFC-CTX-04 | [docs/RFC.md](RFC.md) | Restrição | Time pequeno; evita operar infraestrutura nova | TRANSCRICAO | [[09:07] Diego](../TRANSCRICAO.md#L52) |
| RFC-PROP-01 | [docs/RFC.md](RFC.md) | Decisão | Outbox no MySQL, linha por evento na transação de changeStatus | TRANSCRICAO | [[09:06] Diego](../TRANSCRICAO.md#L48) |
| RFC-PROP-02 | [docs/RFC.md](RFC.md) | Decisão | Tabela de configuração: url, secret, customer_id, ativo, status de interesse | TRANSCRICAO | [[09:21] Bruno](../TRANSCRICAO.md#L128) |
| RFC-PROP-03 | [docs/RFC.md](RFC.md) | Decisão | Worker separado (src/worker.ts + npm run worker), PrismaClient próprio, polling 2 s | TRANSCRICAO | [[09:11] Larissa](../TRANSCRICAO.md#L72) |
| RFC-PROP-04 | [docs/RFC.md](RFC.md) | Decisão | Headers X-Signature, X-Event-Id, X-Timestamp, X-Webhook-Id | TRANSCRICAO | [[09:44] Diego](../TRANSCRICAO.md#L262) |
| RFC-PROP-05 | [docs/RFC.md](RFC.md) | Decisão | Timeout 10 s, retry 5x backoff 1m/5m/30m/2h/12h, DLQ em tabela separada | TRANSCRICAO | [[09:17] Diego](../TRANSCRICAO.md#L104) |
| RFC-PROP-06 | [docs/RFC.md](RFC.md) | Decisão | Módulo em src/modules/webhooks, erros WEBHOOK_, Pino e middleware de erro sem alteração | TRANSCRICAO | [[09:30] Larissa](../TRANSCRICAO.md#L180) |
| RFC-ALT-01 | [docs/RFC.md](RFC.md) | Alternativa descartada | Disparo síncrono de HTTP dentro de changeStatus; travaria mudança de status e forçaria rollback | TRANSCRICAO | [[09:04] Bruno](../TRANSCRICAO.md#L32) |
| RFC-ALT-02 | [docs/RFC.md](RFC.md) | Alternativa descartada | Redis Streams / Redis Cluster; overengineering para o time | TRANSCRICAO | [[09:07] Diego](../TRANSCRICAO.md#L52) |
| RFC-ALT-03 | [docs/RFC.md](RFC.md) | Alternativa descartada | Trigger de banco para acordar o worker; MySQL não tem LISTEN/NOTIFY | TRANSCRICAO | [[09:09] Diego](../TRANSCRICAO.md#L64) |
| RFC-OQ-01 | [docs/RFC.md](RFC.md) | Questão em aberto | Rate limiting de saída: observar e decidir depois | TRANSCRICAO | [[09:39] Diego](../TRANSCRICAO.md#L228) |
| RFC-OQ-02 | [docs/RFC.md](RFC.md) | Questão em aberto | Escalar para múltiplos workers: problema do futuro | TRANSCRICAO | [[09:13] Diego](../TRANSCRICAO.md#L84) |
| RFC-OQ-03 | [docs/RFC.md](RFC.md) | Questão em aberto | Endurecer RBAC do CRUD de webhooks | TRANSCRICAO | [[09:37] Sofia](../TRANSCRICAO.md#L216) |
| RFC-IMP-01 | [docs/RFC.md](RFC.md) | Trade-off | Única alteração de comportamento: publishWebhookEvent(tx, order, fromStatus, toStatus) em changeStatus | TRANSCRICAO | [[09:41] Bruno](../TRANSCRICAO.md#L242) |
| RFC-IMP-02 | [docs/RFC.md](RFC.md) | Risco | Latência mínima de 2 s antes da 1ª tentativa (polling) | TRANSCRICAO | [[09:10] Larissa](../TRANSCRICAO.md#L68) |
| RFC-IMP-03 | [docs/RFC.md](RFC.md) | Risco | Crescimento da outbox; arquivamento fora do escopo | TRANSCRICAO | [[09:08] Diego](../TRANSCRICAO.md#L56) |
| ADR-001 | [docs/adrs/ADR-001-outbox-no-mysql.md](adrs/ADR-001-outbox-no-mysql.md) | Decisão | Padrão Outbox no MySQL; índice em status e created_at | TRANSCRICAO | [[09:08] Diego](../TRANSCRICAO.md#L56) |
| ADR-001-ALT | [docs/adrs/ADR-001-outbox-no-mysql.md](adrs/ADR-001-outbox-no-mysql.md) | Alternativa descartada | Disparo síncrono; Redis Streams | TRANSCRICAO | [[09:06] Diego](../TRANSCRICAO.md#L48) |
| ADR-001-REF | [docs/adrs/ADR-001-outbox-no-mysql.md](adrs/ADR-001-outbox-no-mysql.md) | Integração | Inserção na $transaction de changeStatus | CODIGO | [src/modules/orders/order.service.ts#L131](../src/modules/orders/order.service.ts#L131) |
| ADR-002 | [docs/adrs/ADR-002-worker-separado-em-polling.md](adrs/ADR-002-worker-separado-em-polling.md) | Decisão | Worker em processo separado, polling de 2 s | TRANSCRICAO | [[09:10] Larissa](../TRANSCRICAO.md#L68) |
| ADR-002-ORD | [docs/adrs/ADR-002-worker-separado-em-polling.md](adrs/ADR-002-worker-separado-em-polling.md) | Restrição | Single-worker; ordenação por order_id via created_at | TRANSCRICAO | [[09:12] Diego](../TRANSCRICAO.md#L80) |
| ADR-002-ALT | [docs/adrs/ADR-002-worker-separado-em-polling.md](adrs/ADR-002-worker-separado-em-polling.md) | Alternativa descartada | Trigger de banco; worker dentro da API | TRANSCRICAO | [[09:09] Diego](../TRANSCRICAO.md#L64) |
| ADR-002-REF | [docs/adrs/ADR-002-worker-separado-em-polling.md](adrs/ADR-002-worker-separado-em-polling.md) | Integração | Novo entrypoint modelado em src/server.ts | CODIGO | [src/server.ts#L6](../src/server.ts#L6) |
| ADR-003 | [docs/adrs/ADR-003-retry-com-backoff-e-dlq.md](adrs/ADR-003-retry-com-backoff-e-dlq.md) | Decisão | Retry 5x, backoff 1m/5m/30m/2h/12h, DLQ em webhook_dead_letter | TRANSCRICAO | [[09:17] Diego](../TRANSCRICAO.md#L104) |
| ADR-003-ALT1 | [docs/adrs/ADR-003-retry-com-backoff-e-dlq.md](adrs/ADR-003-retry-com-backoff-e-dlq.md) | Alternativa descartada | 3 tentativas; agressivo demais | TRANSCRICAO | [[09:16] Diego](../TRANSCRICAO.md#L100) |
| ADR-003-ALT2 | [docs/adrs/ADR-003-retry-com-backoff-e-dlq.md](adrs/ADR-003-retry-com-backoff-e-dlq.md) | Alternativa descartada | Retry indefinido; evento pendurado para sempre | TRANSCRICAO | [[09:15] Diego](../TRANSCRICAO.md#L96) |
| ADR-003-ALT3 | [docs/adrs/ADR-003-retry-com-backoff-e-dlq.md](adrs/ADR-003-retry-com-backoff-e-dlq.md) | Alternativa descartada | Marcar "failed" na própria outbox; polui a leitura | TRANSCRICAO | [[09:18] Diego](../TRANSCRICAO.md#L110) |
| ADR-003-RPL | [docs/adrs/ADR-003-retry-com-backoff-e-dlq.md](adrs/ADR-003-retry-com-backoff-e-dlq.md) | Decisão | Replay manual via POST /admin/webhooks/dead-letter/:id/replay | TRANSCRICAO | [[09:18] Diego](../TRANSCRICAO.md#L114) |
| ADR-004 | [docs/adrs/ADR-004-assinatura-hmac-sha256-secret-por-endpoint.md](adrs/ADR-004-assinatura-hmac-sha256-secret-por-endpoint.md) | Decisão | HMAC-SHA256 sobre o corpo, header X-Signature, secret por endpoint | TRANSCRICAO | [[09:22] Sofia](../TRANSCRICAO.md#L134) |
| ADR-004-ROT | [docs/adrs/ADR-004-assinatura-hmac-sha256-secret-por-endpoint.md](adrs/ADR-004-assinatura-hmac-sha256-secret-por-endpoint.md) | Decisão | Rotação de secret com grace period de 24 h | TRANSCRICAO | [[09:21] Sofia](../TRANSCRICAO.md#L130) |
| ADR-004-ALT | [docs/adrs/ADR-004-assinatura-hmac-sha256-secret-por-endpoint.md](adrs/ADR-004-assinatura-hmac-sha256-secret-por-endpoint.md) | Alternativa descartada | Secret global da plataforma; vaza uma, vaza tudo | TRANSCRICAO | [[09:21] Sofia](../TRANSCRICAO.md#L126) |
| ADR-004-REF | [docs/adrs/ADR-004-assinatura-hmac-sha256-secret-por-endpoint.md](adrs/ADR-004-assinatura-hmac-sha256-secret-por-endpoint.md) | Integração | Validação de url https via schema Zod | CODIGO | [src/middlewares/validate.middleware.ts#L11](../src/middlewares/validate.middleware.ts#L11) |
| ADR-005 | [docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md](adrs/ADR-005-entrega-at-least-once-com-x-event-id.md) | Decisão | At-least-once com X-Event-Id (UUID) para dedupe do cliente | TRANSCRICAO | [[09:26] Larissa](../TRANSCRICAO.md#L158) |
| ADR-005-ALT | [docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md](adrs/ADR-005-entrega-at-least-once-com-x-event-id.md) | Alternativa descartada | Exactly-once; exigiria coordenação dos dois lados | TRANSCRICAO | [[09:25] Diego](../TRANSCRICAO.md#L154) |
| ADR-005-DOC | [docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md](adrs/ADR-005-entrega-at-least-once-com-x-event-id.md) | Trade-off | Dedupe é responsabilidade do cliente; documentar no portal | TRANSCRICAO | [[09:26] Marcos](../TRANSCRICAO.md#L156) |
| ADR-006 | [docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md](adrs/ADR-006-reuso-dos-padroes-do-projeto.md) | Decisão | Reuso máximo: módulo, AppError, prefixo WEBHOOK_, Pino, error middleware, Zod, UUID | TRANSCRICAO | [[09:30] Larissa](../TRANSCRICAO.md#L180) |
| ADR-006-MOD | [docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md](adrs/ADR-006-reuso-dos-padroes-do-projeto.md) | Restrição | Módulo webhooks igual aos outros (controller/service/repository/routes/schemas), cabeado em buildControllers | CODIGO | [src/app.ts#L26](../src/app.ts#L26) |
| ADR-006-ERR | [docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md](adrs/ADR-006-reuso-dos-padroes-do-projeto.md) | Integração | Erros herdam de AppError; middleware de erro sem alteração | CODIGO | [src/shared/errors/app-error.ts#L3](../src/shared/errors/app-error.ts#L3) |
| ADR-006-LOG | [docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md](adrs/ADR-006-reuso-dos-padroes-do-projeto.md) | Integração | Logger Pino reutilizado sem nada novo | CODIGO | [src/shared/logger/index.ts#L32](../src/shared/logger/index.ts#L32) |
| ADR-006-DB | [docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md](adrs/ADR-006-reuso-dos-padroes-do-projeto.md) | Integração | Worker abre PrismaClient próprio, mesma DATABASE_URL | CODIGO | [src/config/database.ts#L4](../src/config/database.ts#L4) |
| ADR-006-ADMIN | [docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md](adrs/ADR-006-reuso-dos-padroes-do-projeto.md) | Integração | Replay usa requireRole('ADMIN') existente | CODIGO | [src/middlewares/auth.middleware.ts#L49](../src/middlewares/auth.middleware.ts#L49) |
| ADR-007 | [docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md](adrs/ADR-007-snapshot-do-payload-na-insercao.md) | Decisão | Snapshot do payload renderizado na inserção da outbox | TRANSCRICAO | [[09:52] Larissa](../TRANSCRICAO.md#L310) |
| ADR-007-ALT | [docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md](adrs/ADR-007-snapshot-do-payload-na-insercao.md) | Alternativa descartada | Guardar só order_id e renderizar no envio | TRANSCRICAO | [[09:52] Larissa](../TRANSCRICAO.md#L310) |
| ADR-007-PL | [docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md](adrs/ADR-007-snapshot-do-payload-na-insercao.md) | Decisão | Payload sem items do pedido; cliente consulta GET /orders/:id | TRANSCRICAO | [[09:43] Diego](../TRANSCRICAO.md#L256) |
| ADR-006-UUID | [docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md](adrs/ADR-006-reuso-dos-padroes-do-projeto.md) | Decisão | id da outbox = UUID, padrão do projeto | TRANSCRICAO | [[09:51] Larissa](../TRANSCRICAO.md#L304) |
| FDD-OBJ-01 | [docs/FDD.md](FDD.md) | Requisito Não Funcional | Commit implica evento registrado; rollback implica evento inexistente | TRANSCRICAO | [[09:41] Diego](../TRANSCRICAO.md#L240) |
| FDD-FLOW-01 | [docs/FDD.md](FDD.md) | Decisão | publishWebhookEvent consulta endpoints ativos do customer com o status no filtro | TRANSCRICAO | [[09:34] Bruno](../TRANSCRICAO.md#L198) |
| FDD-FLOW-02 | [docs/FDD.md](FDD.md) | Decisão | Worker seleciona pendentes por created_at, marca processing, envia, grava delivery | TRANSCRICAO | [[09:09] Diego](../TRANSCRICAO.md#L60) |
| FDD-FLOW-03 | [docs/FDD.md](FDD.md) | Decisão | Backoff persistido em next_attempt_at; restart não perde retries | TRANSCRICAO | [[09:17] Diego](../TRANSCRICAO.md#L104) |
| FDD-FLOW-04 | [docs/FDD.md](FDD.md) | Decisão | Replay cria novo evento com novo event_id e loga o autor | TRANSCRICAO | [[09:36] Sofia](../TRANSCRICAO.md#L210) |
| FDD-CONTRATO-01 | [docs/FDD.md](FDD.md) | Contrato | POST /api/v1/webhooks: 201 retorna secret; 400/404/409 nos erros | TRANSCRICAO | [[09:31] Marcos](../TRANSCRICAO.md#L184) |
| FDD-CONTRATO-02 | [docs/FDD.md](FDD.md) | Contrato | GET /api/v1/webhooks lista por customerId, formato paginated | CODIGO | [src/shared/http/response.ts#L22](../src/shared/http/response.ts#L22) |
| FDD-CONTRATO-03 | [docs/FDD.md](FDD.md) | Contrato | GET /api/v1/webhooks/:id/deliveries: últimos 100 com resultado, resposta, duração | TRANSCRICAO | [[09:34] Marcos](../TRANSCRICAO.md#L202) |
| FDD-CONTRATO-04 | [docs/FDD.md](FDD.md) | Contrato | POST /api/v1/admin/webhooks/dead-letter/:id/replay: role ADMIN, 202 | TRANSCRICAO | [[09:35] Diego](../TRANSCRICAO.md#L206) |
| FDD-CONTRATO-05 | [docs/FDD.md](FDD.md) | Contrato | POST /api/v1/webhooks/:id/rotate-secret: nova secret + previousSecretValidUntil (24 h) | TRANSCRICAO | [[09:21] Sofia](../TRANSCRICAO.md#L130) |
| FDD-CONTRATO-06 | [docs/FDD.md](FDD.md) | Contrato | Payload de saída: eventId, eventType, timestamp, orderId, orderNumber, fromStatus, toStatus, customerId, totalCents | TRANSCRICAO | [[09:43] Diego](../TRANSCRICAO.md#L256) |
| FDD-CONTRATO-07 | [docs/FDD.md](FDD.md) | Contrato | Headers de saída: Content-Type, X-Event-Id, X-Webhook-Id, X-Signature, X-Timestamp | TRANSCRICAO | [[09:44] Diego](../TRANSCRICAO.md#L262) |
| FDD-CONTRATO-08 | [docs/FDD.md](FDD.md) | Contrato | X-Webhook-Id identifica qual cadastro gerou o envio | TRANSCRICAO | [[09:44] Sofia](../TRANSCRICAO.md#L264) |
| FDD-ERR-01 | [docs/FDD.md](FDD.md) | Erro | WEBHOOK_INVALID_URL (400): url não https | TRANSCRICAO | [[09:23] Sofia](../TRANSCRICAO.md#L138) |
| FDD-ERR-02 | [docs/FDD.md](FDD.md) | Erro | WEBHOOK_PAYLOAD_TOO_LARGE: payload acima de 64 KB, rollback na transação | TRANSCRICAO | [[09:24] Larissa](../TRANSCRICAO.md#L144) |
| FDD-ERR-03 | [docs/FDD.md](FDD.md) | Erro | WEBHOOK_DELIVERY_TIMEOUT: sem resposta em 10 s | TRANSCRICAO | [[09:42] Diego](../TRANSCRICAO.md#L250) |
| FDD-ERR-04 | [docs/FDD.md](FDD.md) | Erro | Prefixo WEBHOOK_ em todos os códigos do módulo; herdam de AppError | TRANSCRICAO | [[09:29] Larissa](../TRANSCRICAO.md#L172) |
| FDD-ERR-05 | [docs/FDD.md](FDD.md) | Erro | FORBIDDEN (403): replay sem role ADMIN, via requireRole | CODIGO | [src/middlewares/auth.middleware.ts#L49](../src/middlewares/auth.middleware.ts#L49) |
| FDD-RES-01 | [docs/FDD.md](FDD.md) | Requisito Não Funcional | Timeout 10 s por tentativa | TRANSCRICAO | [[09:42] Diego](../TRANSCRICAO.md#L250) |
| FDD-RES-02 | [docs/FDD.md](FDD.md) | Decisão | Nenhuma chamada de rede dentro da $transaction de changeStatus | TRANSCRICAO | [[09:41] Diego](../TRANSCRICAO.md#L240) |
| FDD-OBS-01 | [docs/FDD.md](FDD.md) | Requisito Não Funcional | Logs estruturados de publicação, tentativa, DLQ, replay, sem vazar secret | TRANSCRICAO | [[09:29] Bruno](../TRANSCRICAO.md#L174) |
| FDD-OBS-02 | [docs/FDD.md](FDD.md) | Requisito Não Funcional | Correlação via X-Request-Id da requisição de mudança de status | CODIGO | [src/middlewares/request-logger.middleware.ts#L5](../src/middlewares/request-logger.middleware.ts#L5) |
| FDD-OBS-03 | [docs/FDD.md](FDD.md) | Requisito Não Funcional | Replay logado com replayedBy para auditoria | TRANSCRICAO | [[09:36] Sofia](../TRANSCRICAO.md#L210) |
| FDD-DEP-01 | [docs/FDD.md](FDD.md) | Dependência | Node 20, mesma do projeto | CODIGO | [package.json#L7](../package.json#L7) |
| FDD-DEP-02 | [docs/FDD.md](FDD.md) | Dependência | Prisma 5.22, modelos novos na mesma schema.prisma | CODIGO | [prisma/schema.prisma](../prisma/schema.prisma) |
| FDD-DEP-03 | [docs/FDD.md](FDD.md) | Restrição | Projeto não tem cliente HTTP de saída padronizado; escolha vira convenção | CODIGO | [package.json](../package.json) |
| FDD-INT-01 | [docs/FDD.md](FDD.md) | Integração | publishWebhookEvent(tx, order, from, to) ao fim do bloco de changeStatus | CODIGO | [src/modules/orders/order.service.ts#L126](../src/modules/orders/order.service.ts#L126) |
| FDD-INT-02 | [docs/FDD.md](FDD.md) | Integração | statusFilter validado com z.nativeEnum(OrderStatus) | CODIGO | [src/modules/orders/order.status.ts#L3](../src/modules/orders/order.status.ts#L3) |
| FDD-INT-03 | [docs/FDD.md](FDD.md) | Integração | Erros do módulo reusam AppError / ConflictError / UnprocessableEntityError / NotFoundError | CODIGO | [src/shared/errors/http-errors.ts#L3](../src/shared/errors/http-errors.ts#L3) |
| FDD-INT-04 | [docs/FDD.md](FDD.md) | Integração | Middleware de erro central sem alteração serializa AppError e ZodError | CODIGO | [src/middlewares/error.middleware.ts#L14](../src/middlewares/error.middleware.ts#L14) |
| FDD-INT-05 | [docs/FDD.md](FDD.md) | Integração | Rotas usam authenticate; replay encadeia requireRole('ADMIN') como user.routes.ts | CODIGO | [src/middlewares/auth.middleware.ts#L49](../src/middlewares/auth.middleware.ts#L49) |
| FDD-INT-06 | [docs/FDD.md](FDD.md) | Integração | Worker instancia PrismaClient próprio, não o singleton | CODIGO | [src/config/database.ts#L10](../src/config/database.ts#L10) |
| FDD-INT-07 | [docs/FDD.md](FDD.md) | Integração | src/worker.ts modelado no bootstrap/shutdown de src/server.ts | CODIGO | [src/server.ts#L6](../src/server.ts#L6) |
| FDD-INT-08 | [docs/FDD.md](FDD.md) | Integração | buildControllers instancia WebhookController; buildApiRouter monta /webhooks e /admin/webhooks | CODIGO | [src/app.ts#L26](../src/app.ts#L26) |
| FDD-INT-09 | [docs/FDD.md](FDD.md) | Integração | Schemas Zod passados a validate({ body, params, query }); regra https via .refine() | CODIGO | [src/middlewares/validate.middleware.ts#L11](../src/middlewares/validate.middleware.ts#L11) |
| FDD-INT-10 | [docs/FDD.md](FDD.md) | Integração | logger Pino importado por API e worker; redactPaths já cobre token/authorization | CODIGO | [src/shared/logger/index.ts#L4](../src/shared/logger/index.ts#L4) |
| FDD-INT-11 | [docs/FDD.md](FDD.md) | Integração | Quatro modelos novos em schema.prisma seguindo as convenções (uuid, @@map, @@index) | CODIGO | [prisma/schema.prisma#L40](../prisma/schema.prisma#L40) |
| FDD-INT-12 | [docs/FDD.md](FDD.md) | Integração | Router de webhooks segue o padrão de customer.routes.ts | CODIGO | [src/modules/customers/customer.routes.ts#L14](../src/modules/customers/customer.routes.ts#L14) |
| FDD-INT-13 | [docs/FDD.md](FDD.md) | Integração | requireRole('ADMIN') encadeado como em GET /users/:id | CODIGO | [src/modules/users/user.routes.ts#L15](../src/modules/users/user.routes.ts#L15) |
| FDD-INT-14 | [docs/FDD.md](FDD.md) | Integração | updateOrderStatusSchema já usa z.nativeEnum(OrderStatus); mesmo padrão para statusFilter | CODIGO | [src/modules/orders/order.schemas.ts#L18](../src/modules/orders/order.schemas.ts#L18) |

## Cobertura

- **Total de linhas de dados:** 143
- **Linhas com Fonte = TRANSCRICAO e timestamp `[hh:mm] Nome` válido:** 111 (78%)
- **Linhas com Fonte = CODIGO e caminho de arquivo real:** 32
- **Itens identificáveis nos documentos cobertos:** todos os requisitos funcionais (12),
  requisitos não funcionais (14), decisões dos 7 ADRs, alternativas descartadas (11),
  questões em aberto (3), contratos (8), erros representativos (5), pontos de integração
  (14), riscos (4) e itens fora de escopo (7). Cobertura estimada acima de 90%.

Arquivos de código citados, todos existentes no repositório:
[`src/modules/orders/order.service.ts`](../src/modules/orders/order.service.ts),
[`src/modules/orders/order.status.ts`](../src/modules/orders/order.status.ts),
[`src/modules/orders/order.schemas.ts`](../src/modules/orders/order.schemas.ts),
[`src/shared/errors/app-error.ts`](../src/shared/errors/app-error.ts),
[`src/shared/errors/http-errors.ts`](../src/shared/errors/http-errors.ts),
[`src/middlewares/error.middleware.ts`](../src/middlewares/error.middleware.ts),
[`src/middlewares/auth.middleware.ts`](../src/middlewares/auth.middleware.ts),
[`src/middlewares/validate.middleware.ts`](../src/middlewares/validate.middleware.ts),
[`src/middlewares/request-logger.middleware.ts`](../src/middlewares/request-logger.middleware.ts),
[`src/config/database.ts`](../src/config/database.ts),
[`src/server.ts`](../src/server.ts),
[`src/app.ts`](../src/app.ts),
[`src/routes/index.ts`](../src/routes/index.ts),
[`src/shared/logger/index.ts`](../src/shared/logger/index.ts),
[`src/shared/http/response.ts`](../src/shared/http/response.ts),
[`src/modules/customers/customer.routes.ts`](../src/modules/customers/customer.routes.ts),
[`src/modules/users/user.routes.ts`](../src/modules/users/user.routes.ts),
[`prisma/schema.prisma`](../prisma/schema.prisma),
[`package.json`](../package.json),
[`tests/orders.test.ts`](../tests/orders.test.ts).

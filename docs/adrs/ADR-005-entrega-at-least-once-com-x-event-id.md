# ADR-005: Entrega at-least-once com dedupe do lado do cliente via X-Event-Id

**Status:** Aceito
**Data:** 2026-09-10
**Decisões relacionadas:** ADR-001, ADR-003, ADR-004

## Contexto e problema

Com outbox (ADR-001) e retry (ADR-003), há cenários em que a plataforma envia um evento, o
cliente processa e responde, mas a resposta se perde ou chega depois do timeout de 10
segundos. Nesse caso o worker vai retentar e o cliente receberá o mesmo evento mais de uma
vez.

Era preciso decidir qual garantia de entrega oferecer e como o cliente distingue uma
duplicata.

## Decisão

A garantia é **at-least-once**: o cliente pode receber o mesmo evento duas ou mais vezes e
precisa estar preparado para isso.

Cada evento carrega um `event_id` (UUID) gerado no momento em que o evento entra na outbox,
único por evento, enviado no header `X-Event-Id`. O cliente **deduplica pelo `event_id`**
do lado dele. Esse é o mesmo mecanismo que Stripe e GitHub usam para webhooks.

O comportamento at-least-once e a necessidade de dedupe serão documentados de forma
destacada no portal de desenvolvedor dos clientes.

## Alternativas consideradas

- **Exactly-once (garantir que o cliente processa o evento exatamente uma vez).**
  Descartada: exigiria coordenação entre os dois lados (a plataforma e o sistema do
  cliente), o que aumenta muito a complexidade. At-least-once com `event_id` resolve a
  grande maioria dos casos com esforço muito menor.

## Consequências

Positivas:
- Modelo de entrega simples e alinhado ao padrão de mercado; qualquer time que já integra
  Stripe/GitHub conhece o padrão.
- A plataforma não precisa manter estado de "confirmação de processamento" por cliente.

Negativas e limitações aceitas:
- A responsabilidade de deduplicar é transferida para o cliente, o que precisa ser
  comunicado com clareza no portal de desenvolvedor.
- O `event_id` (UUID) passa a ser um campo obrigatório do payload e da tabela de outbox, e
  fica atrelado à decisão de usar UUID como identificador (ver ADR-006).

## Referências

- `webhook_outbox` (modelo novo) terá `id` UUID usado também como `event_id`, seguindo o
  padrão de PKs `@db.Char(36)` de [`prisma/schema.prisma`](../../prisma/schema.prisma#L16).
- [ADR-003](ADR-003-retry-com-backoff-e-dlq.md) : o retry que gera as duplicatas.
- [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md) : decisão de usar UUID como identificador.
- [FDD](../FDD.md) : semântica do header `X-Event-Id` e o formato do payload.

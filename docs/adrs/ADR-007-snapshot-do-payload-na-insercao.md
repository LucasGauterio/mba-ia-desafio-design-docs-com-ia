# ADR-007: Snapshot do payload renderizado na inserção da outbox

**Status:** Aceito
**Data:** 2026-09-09
**Decisões relacionadas:** ADR-001, ADR-005

## Contexto e problema

O evento na `webhook_outbox` (ADR-001) descreve uma mudança de status de um pedido. Entre o
momento em que o evento entra na outbox e o momento em que o worker consegue entregá-lo
podem se passar de 2 segundos a ~15 horas (ADR-002, ADR-003). Nesse intervalo, o pedido
pode sofrer outras mudanças.

A questão era se a linha da outbox guarda o **payload já renderizado** ou apenas o
`order_id`, deixando o worker montar o payload no momento do envio ([09:51] Bruno).

## Decisão

A outbox guarda o **payload renderizado no momento da inserção** (snapshot). O evento
reflete o estado do pedido **quando aquele status mudou**, não o estado no momento do envio
([09:52] Larissa / Diego / Bruno).

O payload é um JSON com `event_id`, `event_type` (`"order.status_changed"`), `timestamp`
ISO 8601, `order_id`, `order_number`, `from_status`, `to_status`, `customer_id` e campos
básicos do pedido como `total_cents`. Os itens do pedido não vão no payload, para não
inflá-lo; o cliente que quiser detalhes consulta `GET /orders/:id` ([09:43] Diego).

## Alternativas consideradas

- **Guardar só `order_id` e renderizar o payload no momento do envio.** Descartada: se o
  pedido mudar de estado entre a inserção e o envio, o evento entregaria um estado
  diferente do que disparou a notificação, gerando casos confusos para o cliente ([09:52]
  Larissa).

## Consequências

Positivas:
- Cada notificação é fiel à transição que a originou, mesmo com atraso de entrega e
  mudanças posteriores no pedido.
- O worker não precisa de acesso de leitura consistente ao estado atual do pedido no
  momento do envio; ele só lê a própria linha da outbox.

Negativas e limitações aceitas:
- O payload fica armazenado por evento, aumentando o tamanho de cada linha da outbox (o
  limite de 64 KB por payload, definido como requisito não funcional, protege contra casos
  extremos).
- Uma mudança futura no formato do payload não se aplica retroativamente a eventos já
  inseridos, incluindo os que estão na DLQ aguardando replay.

## Referências

- `src/modules/orders/order.service.ts` : a renderização do snapshot ocorre na publicação
  do evento, dentro da transação de `changeStatus`.
- `prisma/schema.prisma` : coluna de payload em `webhook_outbox`.
- `docs/FDD.md` : formato completo do payload e a matriz de erros (`WEBHOOK_PAYLOAD_TOO_LARGE`).

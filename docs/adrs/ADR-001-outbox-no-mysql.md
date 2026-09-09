# ADR-001: Padrão Outbox no MySQL para os eventos de webhook

**Status:** Aceito
**Data:** 2026-09-09
**Decisões relacionadas:** ADR-002, ADR-006, ADR-007

## Contexto e problema

Três clientes B2B pediram para ser notificados quando o status de um pedido muda, em vez de
fazer polling no `GET /orders`. O status do pedido só muda em `OrderService.changeStatus`
(`src/modules/orders/order.service.ts`), que já roda uma transação pesada: atualiza
`orders`, insere em `order_status_history` e ajusta `stockQuantity` dos produtos do pedido.

A pergunta de arquitetura foi como disparar a notificação a partir dessa mudança de status.
Duas forças em tensão: a notificação não pode acoplar a latência nem a disponibilidade da
mudança de status a um endpoint HTTP externo, e ao mesmo tempo não pode haver o caso de o
status mudar e o evento não ser registrado. O time é pequeno e evita subir infraestrutura
nova.

## Decisão

Adotar o padrão **Outbox** persistido no MySQL já existente. Quando o status de um pedido
muda, dentro da **mesma transação SQL** que atualiza `orders` e `order_status_history`, uma
linha de evento é inserida numa tabela `webhook_outbox`. Um worker separado lê essa tabela
e faz as chamadas HTTP (ver ADR-002).

A tabela tem índice no campo de status do evento (pendente, processando, falhou, entregue)
e em `created_at`, para o worker selecionar em lote os pendentes mais antigos. Se a
transação principal commitou, o evento foi registrado; se deu rollback, o evento some junto.

## Alternativas consideradas

- **Disparo síncrono de HTTP dentro de `changeStatus`.** Descartada: um cliente lento
  travaria a mudança de status de outros pedidos, e um cliente fora do ar forçaria a
  escolha entre dar rollback numa mudança de status legítima ou perder o evento.
- **Redis Streams / Redis Cluster como fila de eventos.** Descartada: exigiria subir e
  operar infraestrutura nova, considerado overengineering para o tamanho do time. O outbox
  no MySQL existente resolve o problema sem essa dependência.

## Consequências

Positivas:
- Atomicidade real entre a mudança de status e o registro do evento, sem transação
  distribuída.
- Nenhuma dependência de infraestrutura nova; usa o MySQL e o Prisma que já existem.
- A tabela funciona como trilha de auditoria e ponto de reprocessamento.

Negativas e limitações aceitas:
- A entrega deixa de ser imediata: passa a depender do ciclo de polling do worker (ver
  ADR-002), com latência de até 2 s no pior caso.
- A tabela `webhook_outbox` cresce com o volume de mudanças de status; o arquivamento de
  linhas entregues foi deixado fora do escopo desta feature.
- `OrderService.changeStatus` ganha uma escrita a mais dentro da transação, aumentando um
  pouco o tempo de lock (ver ADR-006 para a forma de integração).

## Referências

- `src/modules/orders/order.service.ts` : `OrderService.changeStatus`, onde a inserção na
  outbox acontece dentro da `$transaction`.
- `src/modules/orders/order.status.ts` : enum `OrderStatus`, base do filtro de eventos.
- `prisma/schema.prisma` : onde os modelos `webhook_outbox` e correlatos serão definidos.
- `docs/RFC.md` : proposta técnica consolidada.

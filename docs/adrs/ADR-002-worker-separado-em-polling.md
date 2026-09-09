# ADR-002: Worker em processo separado, polling a cada 2 segundos

**Status:** Aceito
**Data:** 2026-09-09
**Decisões relacionadas:** ADR-001, ADR-003, ADR-006

## Contexto e problema

Com o padrão outbox decidido (ADR-001), os eventos ficam numa tabela do MySQL e alguém
precisa lê-los e disparar as chamadas HTTP para os endpoints dos clientes. Duas questões:
como esse componente descobre que há eventos novos, e onde ele roda.

Os clientes consideram "tempo real" qualquer coisa abaixo de 10 segundos. O MySQL não tem
um mecanismo de notificação para processos externos equivalente ao `LISTEN/NOTIFY` do
PostgreSQL. A API hoje roda como um único processo (`src/server.ts`); se o componente de
envio rodasse dentro dela, um restart da API derrubaria o envio junto.

## Decisão

O envio é feito por um **worker em processo separado**, com um novo entry-point
(`src/worker.ts`) e um script `npm run worker`, no mesmo estilo do `src/server.ts`. O
worker conecta no **mesmo banco** com um `PrismaClient` próprio (ver ADR-006).

O worker descobre eventos novos por **polling em loop**: a cada **2 segundos** busca os
eventos pendentes mais antigos, processa e marca. A latência mínima de notificação passa a
ser 2 segundos no pior caso, o que foi aceito por atender folgadamente o requisito de
"abaixo de 10 segundos".

Enquanto houver **um único worker**, os eventos de um mesmo pedido são processados em ordem
de `created_at` da outbox, garantindo ordenação **por `order_id`**. Não há garantia de
ordenação global, e os clientes nunca pediram isso.

## Alternativas consideradas

- **Trigger de banco para acordar o worker de forma reativa.** Descartada: um trigger no
  MySQL só executa SQL, não notifica um processo externo; improvisar (escrever em arquivo,
  bater num endpoint) ficaria frágil, e o polling de 2 s já atende o requisito de latência.
- **Worker dentro da mesma instância da API.** Descartada: acoplaria o ciclo de vida do
  envio ao da API; um restart da API pararia o processamento de eventos.

## Consequências

Positivas:
- O envio é resiliente a restarts da API e escala/reinicia de forma independente.
- Implementação simples: um loop de polling, sem infraestrutura de mensageria.
- Ordenação por pedido garantida sem esforço adicional enquanto o worker for único.

Negativas e limitações aceitas:
- Latência de até 2 segundos antes de a primeira tentativa de envio ocorrer.
- Passa a existir um segundo processo para operar, monitorar e fazer deploy.
- Ordenação global não é garantida. Escalar para múltiplos workers (particionando por
  `order_id` ou com lock pessimista) fica como trabalho futuro fora desta feature.

## Referências

- `src/server.ts` : padrão de bootstrap e shutdown que o `src/worker.ts` vai seguir.
- `src/config/database.ts` : `PrismaClient` singleton; o worker instancia o seu próprio.
- `src/modules/orders/order.status.ts` : enum `OrderStatus` usado na seleção de eventos.
- `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` : o que o worker faz quando um envio falha.

# ADR-003: Retry com backoff exponencial (5 tentativas) e DLQ em tabela separada

**Status:** Aceito
**Data:** 2026-09-10
**Decisões relacionadas:** ADR-001, ADR-002

## Contexto e problema

O worker (ADR-002) faz chamadas HTTP para endpoints fora da infraestrutura da plataforma.
Esses endpoints ficam indisponíveis: manutenção planejada, incidentes, deploys do lado do
cliente. Já houve cliente com indisponibilidade de duas horas em manutenção planejada. Um
envio sem resposta em 10 segundos é tratado como falha.

Precisava-se decidir quantas vezes retentar, com qual espaçamento, e o que fazer quando o
teto de tentativas é atingido, sem deixar eventos "pendurados para sempre" na outbox se um
cliente sumiu.

## Decisão

**Retry com backoff exponencial, 5 tentativas**, com a progressão **1 minuto, 5 minutos,
30 minutos, 2 horas, 12 horas**. Isso cobre cerca de 15 horas entre a primeira falha e a
última tentativa, considerado aceitável porque um cliente indisponível por 15 horas já tem
um problema próprio sério.

Depois da 5ª tentativa sem sucesso, o evento é considerado **falha permanente** e movido
para uma **tabela separada `webhook_dead_letter`**, que guarda o payload, o motivo da falha
e o timestamp.

O reprocessamento é **manual**, via `POST /admin/webhooks/dead-letter/:id/replay`, que
recoloca o evento na outbox como pendente.

## Alternativas consideradas

- **3 tentativas.** Descartada: seria agressivo demais. Um cliente com indisponibilidade
  matinal seria retentado três vezes em cerca de 30 minutos e o evento morreria antes de o
  cliente voltar.
- **Retry indefinido com backoff.** Descartada: um evento ficaria pendurado para sempre se
  o endpoint do cliente sumisse de vez, sem um ponto claro de intervenção.
- **Marcar o evento como "failed" na própria `webhook_outbox`, sem tabela de DLQ.**
  Descartada: manter os eventos falhados na outbox principal polui a leitura do worker; uma
  tabela separada mantém a outbox limpa e serve de evidência para debug e reprocessamento.

## Consequências

Positivas:
- Tolera indisponibilidades longas do cliente sem intervenção, dentro de uma janela de
  ~15 horas.
- A outbox principal só contém eventos ativos; a DLQ isola os casos que exigem ação humana.
- O endpoint de replay dá um caminho de recuperação auditável.

Negativas e limitações aceitas:
- Uma notificação pode demorar até ~15 horas para chegar ao cliente em cenário de falha
  prolongada.
- Eventos que falham em definitivo exigem ação manual de um administrador; não há
  reprocessamento automático da DLQ.
- Mais um modelo de dados e um endpoint administrativo para manter.

## Referências

- [`auth.middleware.ts`](../../src/middlewares/auth.middleware.ts#L49) : `requireRole`
  protege o endpoint de replay (uso análogo ao de
  [`user.routes.ts`](../../src/modules/users/user.routes.ts#L15)).
- `webhook_dead_letter` e `webhook_outbox` são modelos novos, a serem definidos em
  [`prisma/schema.prisma`](../../prisma/schema.prisma).
- [ADR-002](ADR-002-worker-separado-em-polling.md) : o worker que aplica o retry.
- [FDD](../FDD.md) : fluxo detalhado de retry e DLQ e a matriz de erros.

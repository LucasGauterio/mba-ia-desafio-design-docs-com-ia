# ADR-006: Reuso dos padrões existentes do projeto para o módulo de webhooks

**Status:** Aceito
**Data:** 2026-09-09
**Decisões relacionadas:** ADR-001, ADR-002, ADR-005

## Contexto e problema

O OMS já tem uma estrutura consolidada: cada domínio é um módulo em `src/modules` com
controller, service, repository, routes e schemas; erros herdam de `AppError` com códigos
em maiúsculas; o logger é Pino; um middleware de erro central trata `AppError`, `ZodError`
e erros conhecidos do Prisma; a validação de entrada é feita por schemas Zod; todas as
chaves primárias são UUID `CHAR(36)`.

A feature de webhooks poderia introduzir bibliotecas ou padrões próprios (um cliente HTTP
diferente, um formato de erro próprio, um esquema de logging separado). A decisão era se a
feature adota a stack e as convenções que já existem ou diverge delas.

## Decisão

**Reuso máximo do que já existe.** Concretamente:

- O módulo de webhooks vive em `src/modules/webhooks` com a mesma estrutura de
  controller/service/repository/routes/schemas dos outros domínios.
- Os erros do módulo herdam de `AppError` e usam o prefixo `WEBHOOK_` em todos os códigos
  (`WEBHOOK_NOT_FOUND`, `WEBHOOK_INVALID_URL`, `WEBHOOK_SECRET_REQUIRED`, e assim por
  diante). O middleware de erro central não precisa de nenhuma alteração para tratá-los.
- O logging usa o `logger` Pino já existente, sem nada novo.
- As validações de entrada são schemas Zod, incluindo a exigência de `url` https.
- O replay de DLQ usa o `requireRole('ADMIN')` já existente.
- Os identificadores (incluindo o `id` da outbox, que também é o `event_id`) são UUID,
  seguindo o padrão do resto do projeto.
- O worker abre um `PrismaClient` próprio, com a mesma `DATABASE_URL`, porque `PrismaClient`
  é por processo.

## Alternativas consideradas

- **Introduzir padrões ou bibliotecas próprias para o módulo de webhooks** (formato de erro
  dedicado, logger separado, id auto-incremental na outbox). Descartada: aumentaria a
  superfície de manutenção sem ganho, e divergiria de uma codebase que já tem convenções
  claras e funcionando.

## Consequências

Positivas:
- O time trabalha com padrões que já conhece; onboarding e revisão de código são mais
  rápidos.
- O middleware de erro, o logger e o `requireRole` funcionam sem modificação.
- Consistência de contrato: os erros de webhook têm o mesmo formato de resposta dos demais
  erros da API.

Negativas e limitações aceitas:
- A feature fica presa às limitações das ferramentas atuais (por exemplo, não há um cliente
  HTTP de saída padronizado no projeto hoje, então o worker precisa escolher um e segui-lo
  como nova convenção).
- `OrderService.changeStatus` passa a ter uma dependência conceitual do módulo de webhooks
  (via a função de publicação de evento), embora sem injeção de repository (ver a seção de
  integração no FDD).

## Referências

- `src/modules/orders/order.service.ts` : `OrderService.changeStatus`, onde o evento é
  publicado dentro da transação.
- `src/shared/errors/app-error.ts` e `src/shared/errors/http-errors.ts` : `AppError` e as
  subclasses reutilizadas.
- `src/middlewares/error.middleware.ts` : trata `AppError` sem alteração.
- `src/shared/logger/index.ts` : `logger` Pino reutilizado.
- `src/config/database.ts` : `PrismaClient`; o worker instancia o seu próprio.

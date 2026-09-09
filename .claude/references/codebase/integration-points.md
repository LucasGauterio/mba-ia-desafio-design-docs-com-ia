# Reference · codebase: pontos de extensão

Gerado por `design-docs-baseline`. Catálogo dos lugares do código que uma feature nova
provavelmente vai tocar. Matéria-prima da seção **"Integração com o sistema existente"**
do FDD: todo caminho aqui é real e verificável.

## PE-01 · Mudança de status do pedido (transação)

- **Arquivo:** `src/modules/orders/order.service.ts` → `OrderService.changeStatus`
- **Hoje:** dentro de `prisma.$transaction`, valida a transição via
  `src/modules/orders/order.status.ts`, ajusta `Product.stockQuantity`, atualiza
  `Order.status` e insere `OrderStatusHistory`.
- **Como uma feature se conecta:** qualquer efeito que precise ser atômico com a mudança de
  status (registrar um evento, enfileirar uma notificação) deve entrar **dentro da mesma
  `$transaction`**, recebendo o `tx` (`Prisma.TransactionClient`): não abrir transação nova.
- **Cuidado:** a transação já faz 3+ escritas; acréscimos devem ser escritas locais no
  banco, nunca I/O de rede. Se a escrita extra falhar, o rollback tem que levar tudo junto.

## PE-02 · Máquina de estados do pedido

- **Arquivo:** `src/modules/orders/order.status.ts`
- **Hoje:** `transitions`, `canTransition`, `allowedTransitions`, `isTerminal`,
  `shouldDebitStock`, `shouldReplenishStock`.
- **Como uma feature se conecta:** consumir `OrderStatus` e as funções puras para saber
  quais transições existem e reagir a elas. É a fonte da verdade sobre "que status um
  pedido pode assumir".
- **Cuidado:** enum `OrderStatus` tem 6 valores fixos; features que filtram por status
  devem referenciar o enum, não strings soltas.

## PE-03 · Classes de erro e códigos

- **Arquivos:** `src/shared/errors/app-error.ts`, `src/shared/errors/http-errors.ts`,
  `src/shared/errors/index.ts`
- **Hoje:** `AppError(message, statusCode, errorCode, details?)` + subclasses; códigos em
  `SCREAMING_SNAKE_CASE`.
- **Como uma feature se conecta:** criar subclasses novas de `AppError` (ou usar
  `ConflictError`/`UnprocessableEntityError` com `code` custom) seguindo o padrão. Um
  módulo novo com prefixo de código próprio (ex. `WEBHOOK_*`) é o caminho idiomático.
- **Cuidado:** o `errorMiddleware` já serializa qualquer `AppError` automaticamente: não
  precisa alterá-lo para erros novos.

## PE-04 · Middleware de erro central

- **Arquivo:** `src/middlewares/error.middleware.ts`
- **Hoje:** trata `AppError`, `ZodError`, `Prisma.PrismaClientKnownRequestError`.
- **Como uma feature se conecta:** endpoints novos que lançam `AppError`/`ZodError` são
  cobertos sem mudança. Só precisa de alteração se surgir uma classe de erro que **não**
  herde de `AppError`.

## PE-05 · Autenticação e RBAC

- **Arquivo:** `src/middlewares/auth.middleware.ts` → `authenticate`, `requireRole(...roles)`
- **Hoje:** `authenticate` popula `req.user`; `requireRole('ADMIN')` protege `GET /users/:id`.
- **Como uma feature se conecta:** rotas novas usam `authenticate` no router; rotas
  administrativas encadeiam `requireRole('ADMIN')`: exatamente o padrão de
  `src/modules/users/user.routes.ts`.

## PE-06 · Padrão de módulo e roteamento

- **Arquivos:** `src/modules/<domínio>/*`, `src/routes/index.ts`, `src/app.ts`
  (`buildControllers`, `buildApiRouter`)
- **Hoje:** cada domínio = controller/service/repository/routes/schemas; registrado em
  `buildControllers` e montado em `buildApiRouter` sob `/api/v1/<domínio>`.
- **Como uma feature se conecta:** um domínio novo (ex. `src/modules/webhooks/`) segue a
  mesma estrutura, é instanciado em `buildControllers` e montado em `buildApiRouter`.

## PE-07 · Validação de entrada

- **Arquivo:** `src/middlewares/validate.middleware.ts` + `<domínio>.schemas.ts`
- **Hoje:** `validate({ body?, query?, params? })` com schemas Zod; erro → `ValidationError`.
- **Como uma feature se conecta:** schemas Zod novos por endpoint; regras como "URL precisa
  ser https" são um `.refine()` no schema, não lógica de serviço.

## PE-08 · Cliente Prisma por processo

- **Arquivo:** `src/config/database.ts` (singleton `prisma`)
- **Hoje:** um `PrismaClient` para o processo do servidor HTTP.
- **Como uma feature se conecta:** um processo novo (worker) instancia **seu próprio**
  `PrismaClient` com a mesma `DATABASE_URL`: `PrismaClient` é por processo, não
  compartilhável entre processos Node.

## PE-09 · Novo entrypoint / processo

- **Arquivo de referência:** `src/server.ts` (padrão de bootstrap + shutdown), `package.json` (scripts)
- **Hoje:** só o servidor HTTP.
- **Como uma feature se conecta:** um processo separado seria um `src/<nome>.ts` novo com
  seu bootstrap e um script `npm run <nome>`, conectando no mesmo MySQL via Prisma próprio.

## PE-10 · Logger

- **Arquivo:** `src/shared/logger/index.ts`
- **Hoje:** Pino com redação de secrets, eventos nomeados.
- **Como uma feature se conecta:** importar `logger` e emitir eventos nomeados no mesmo
  estilo (`snake_case`, objeto de contexto primeiro). Nada novo a configurar.

## PE-11 · Modelo de dados / migrations

- **Arquivos:** `prisma/schema.prisma`, `prisma/migrations/`
- **Hoje:** 7 modelos, PKs `CHAR(36)` UUID, `@@index` explícitos, enums.
- **Como uma feature se conecta:** modelos novos seguem as convenções (UUID default,
  `@@map` snake_case plural, timestamps, índices em campos de filtro). Migration nova via
  `prisma migrate dev`.

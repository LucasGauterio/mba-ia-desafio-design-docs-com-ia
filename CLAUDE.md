# CLAUDE.md: Order Management System (OMS)

Contexto da aplicação **como ela é hoje**, derivado apenas do código-fonte
(`src/`, `prisma/`, `tests/`, configs). Não descreve features futuras.

## O que é

API REST de **gestão de pedidos** (Order Management System). Opera sobre um catálogo de
produtos com controle de estoque, clientes, usuários operadores e um ciclo de vida de
pedido com máquina de estados e auditoria de mudanças de status.

## Stack

| Camada | Tecnologia |
|---|---|
| Runtime | Node.js >= 20, ES Modules (`"type": "module"`) |
| Linguagem | TypeScript 5.6 (`tsx` em dev, `tsc` no build) |
| HTTP | Express 4.21 |
| ORM / banco | Prisma 5.22 + MySQL 8 |
| Validação | Zod 3.23 |
| Log | Pino 9.5 (+ `pino-http`, `pino-pretty` em dev) |
| Auth | `jsonwebtoken` 9 (JWT HS256), `bcrypt` 5 |
| IDs | `uuid` 11 (`@db.Char(36)` em todas as PKs) |
| Testes | Vitest 2.1 + Supertest |

## Estrutura

```
src/
├── server.ts               # bootstrap: buildApp + app.listen + shutdown SIGINT/SIGTERM
├── app.ts                  # buildApp(deps) + buildControllers(prisma) (DI manual)
├── config/                 # env.ts (Zod), database.ts (PrismaClient singleton)
├── middlewares/            # auth, error, validate, request-logger
├── routes/index.ts         # monta /api/v1/{auth,users,customers,products,orders}
├── shared/
│   ├── errors/             # AppError + subclasses + índice
│   ├── http/response.ts    # paginated()
│   └── logger/index.ts     # Pino
└── modules/<domínio>/      # auth, users, customers, products, orders
    ├── <d>.controller.ts   # RequestHandlers, try/catch → next(err)
    ├── <d>.service.ts      # regra de negócio, transações
    ├── <d>.repository.ts   # acesso Prisma
    ├── <d>.routes.ts       # Router Express + authenticate + validate()
    └── <d>.schemas.ts      # schemas Zod + tipos inferidos
```

Cada domínio segue o mesmo padrão controller → service → repository. O wiring é manual em
[`buildControllers(prisma)`](src/app.ts#L26): instancia repository, depois service, depois
controller, para cada domínio.

## Convenções transversais

- **Autenticação:** [`authenticate`](src/middlewares/auth.middleware.ts#L27) valida
  `Authorization: Bearer <jwt>` e popula `req.user = { id, email, role }`. Roles:
  `ADMIN`, `OPERATOR`. [`requireRole(...roles)`](src/middlewares/auth.middleware.ts#L49) faz
  RBAC: hoje usado só em `GET /users/:id`
  ([`user.routes.ts`](src/modules/users/user.routes.ts#L15)); os demais endpoints exigem
  apenas autenticação.
- **Erros:** classe base
  [`AppError(message, statusCode, errorCode, details?)`](src/shared/errors/app-error.ts#L3).
  Subclasses em [`http-errors.ts`](src/shared/errors/http-errors.ts#L3): `BadRequestError`,
  `ValidationError`, `UnauthorizedError`, `ForbiddenError`, `NotFoundError`, `ConflictError`,
  `UnprocessableEntityError`, `InvalidStatusTransitionError`, `InsufficientStockError`.
  Códigos em `SCREAMING_SNAKE_CASE` (`VALIDATION_ERROR`, `NOT_FOUND`,
  `INVALID_STATUS_TRANSITION`, `INSUFFICIENT_STOCK`, `CONFLICT`, ...).
- **Middleware de erro central**
  ([`error.middleware.ts`](src/middlewares/error.middleware.ts#L14)): mapeia `AppError`,
  `ZodError` e `Prisma.PrismaClientKnownRequestError` (P2002 → 409, P2025 → 404) para
  `{ "error": { "code", "message", "details"? } }`. Erros não tratados → 500
  `INTERNAL_SERVER_ERROR` + `logger.error`.
- **Validação:**
  [`validate({ body?, query?, params? })`](src/middlewares/validate.middleware.ts#L11) roda
  schemas Zod e lança `ValidationError` com `details: [{ path, message }]`.
- **Resposta HTTP:** listas usam
  [`paginated(data, page, pageSize, total)`](src/shared/http/response.ts#L22) →
  `{ data, pagination: { page, pageSize, total, totalPages } }`.
- **Logging:** [`logger` Pino](src/shared/logger/index.ts#L32), timestamp ISO, `base.service =
  "order-management-api"`, redação de `authorization`, `cookie`, `*.password`, `*.token`.
  [`requestLogger`](src/middlewares/request-logger.middleware.ts#L5) gera/propaga
  `X-Request-Id` e emite `http_request` com método, path, status, duração, `userId`.
- **Config:** [`src/config/env.ts`](src/config/env.ts#L3) valida `process.env` com Zod
  (`NODE_ENV`, `PORT`, `LOG_LEVEL`, `DATABASE_URL`, `JWT_SECRET` (>=16 chars),
  `JWT_EXPIRES_IN` (default `8h`)); processo aborta se inválido.
  [`src/config/database.ts`](src/config/database.ts#L10) exporta o singleton `prisma`.

## Modelo de dados ([`prisma/schema.prisma`](prisma/schema.prisma), MySQL)

Entidades: `User`, `Customer`, `Product`, `Order`, `OrderItem`, `OrderStatusHistory`,
`OrderNumberSequence`. PKs `CHAR(36)` UUID. Enums: `UserRole {ADMIN, OPERATOR}`,
`OrderStatus {PENDING, PAID, PROCESSING, SHIPPED, DELIVERED, CANCELLED}`.

Invariantes relevantes:
- **Máquina de estados do pedido**
  ([`order.status.ts`](src/modules/orders/order.status.ts#L3)): mapa de transições
  permitidas + `canTransition`, `allowedTransitions`, `isTerminal`; `shouldDebitStock`
  (`PENDING→PAID`), `shouldReplenishStock` (`→CANCELLED` vindo de `PAID`/`PROCESSING`).
- **[`OrderService.changeStatus`](src/modules/orders/order.service.ts#L126)** roda dentro de
  `prisma.$transaction`: valida a transição, debita/repõe `Product.stockQuantity`, atualiza
  `Order.status`, insere linha em `OrderStatusHistory` (`fromStatus`, `toStatus`,
  `changedById`, `reason`), retorna o pedido recarregado.
- **[`OrderService.create`](src/modules/orders/order.service.ts#L50)** também transacional:
  valida cliente e produtos (ativos), calcula totais no servidor, reserva `orderNumber` via
  `OrderNumberSequence` (upsert incremental → `ORD-000123`), cria pedido + itens + histórico
  inicial.
- Estoque nunca é debitado fora de transação; `OrderItem` tem `onDelete: Cascade`.

## Entrypoints e execução

- **Único processo:** o servidor HTTP
  ([`src/server.ts`](src/server.ts#L6) → `buildApp` → `app.listen`). Não há worker, cron,
  consumer ou segundo entrypoint.
- Rotas montadas em `/api/v1` ([`routes/index.ts`](src/routes/index.ts#L21)): `/auth`
  (`register`, `login`, `me`), `/users` (`GET /:id` ADMIN), `/customers` (CRUD),
  `/products` (CRUD), `/orders`
  (`GET /`, `GET /:id`, `POST /`, `PATCH /:id/status`, `DELETE /:id`). `GET /health` é público.
- Scripts ([`package.json`](package.json#L10)): `dev` (tsx watch), `build` (tsc), `start`,
  `db:migrate` (`prisma migrate dev`), `db:reset`, `db:seed`
  ([`prisma/seed.ts`](prisma/seed.ts)), `test` (`vitest run`), `lint`, `format`.
- [`docker-compose.yml`](docker-compose.yml) sobe só o MySQL 8 (`oms-mysql`, porta 3306,
  utf8mb4). [`.env.example`](.env.example) lista as variáveis. Migrations em
  `prisma/migrations/`.
- Testes: `tests/*.test.ts` (auth, orders) com
  [`tests/helpers/factories.ts`](tests/helpers/factories.ts)
  (`bootstrapAuthenticatedUser`, `createTestCustomer`, `createTestProduct`, `getTestApp`) e
  [`tests/setup.ts`](tests/setup.ts) (limpa as tabelas em `beforeEach`).

## Ausências (o que o sistema NÃO tem hoje)

- Nenhum mecanismo de **notificação externa, webhook ou chamada HTTP de saída**.
- Nenhum **event bus / domain events / outbox**.
- Nenhuma **fila / broker de mensagens** (RabbitMQ, Kafka, SQS, Redis Streams...).
- Nenhum **processo worker, scheduler ou cron**: só o servidor HTTP.
- Nenhuma **camada de cache** (Redis etc.).
- Nenhum **envio de e-mail** ou canal de comunicação com clientes.
- Nenhum **rate limiting** de entrada ou saída.

## Regras para o agente

A entrega deste repositório é **documental**. Regras sempre ativas (`.claude/rules/`):

- [`source-code-is-read-only.md`](.claude/rules/source-code-is-read-only.md): não editar
  `src/`, `prisma/`, `tests/` nem configs. Só `docs/**`, `CLAUDE.md`, `README.md`, `.claude/**`.
- [`traceability-required.md`](.claude/rules/traceability-required.md): a rastreabilidade
  vive só no `docs/TRACKER.md`; a prosa do PRD/RFC/FDD/ADR fica limpa, sem `[hh:mm]`.
- [`no-cross-document-duplication.md`](.claude/rules/no-cross-document-duplication.md):
  cada documento na sua altura; referência cruzada em vez de cópia.
- [`honor-rejected-scope.md`](.claude/rules/honor-rejected-scope.md): itens que a reunião
  descartou/adiou não viram requisito.
- [`repo-file-links.md`](.claude/rules/repo-file-links.md): toda menção a um arquivo real do
  repositório é link relativo; código citando um trecho leva âncora de linha `#Lnn`.

## Workflow `design-docs`

Este repositório carrega o workflow **`design-docs`** (`.claude/skills/design-docs-*`,
`.claude/commands/`, `.claude/references/`). Ele transforma uma transcrição de reunião + o
código existente em um pacote de design docs. Ponto de entrada: `/design-docs`. Doc do
workflow: [`.claude/README.md`](.claude/README.md). Plano/registro do processo:
[`DESIGN_DOCS_PROCESS.md`](DESIGN_DOCS_PROCESS.md).

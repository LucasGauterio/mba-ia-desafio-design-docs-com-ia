# Reference · codebase: mapa detalhado da aplicação existente

Gerado por `design-docs-baseline`. Consulta durante a autoria dos design docs. Só reflete
o código atual; não contém feature nova.

## 1. Entrypoints e bootstrap

| Arquivo | Papel |
|---|---|
| `src/server.ts` | `bootstrap()`: `buildApp({ prisma })`, `app.listen(env.PORT)`, log `server_started`; `shutdown()` em `SIGINT`/`SIGTERM` (fecha servidor, `prisma.$disconnect`). |
| `src/app.ts` | `buildApp(deps)`: `express.json({ limit: '1mb' })`, `requestLogger`, `GET /health`, `buildApiRouter(controllers)` em `/api/v1`, 404 handler (`NotFoundError`), `errorMiddleware`. `buildControllers(prisma)`: DI manual repository→service→controller por domínio. |
| `src/routes/index.ts` | `buildApiRouter`: monta `/auth`, `/users`, `/customers`, `/products`, `/orders`. |
| `src/config/database.ts` | `createPrismaClient()` + singleton `prisma` (log `warn`/`error` em dev). |
| `src/config/env.ts` | `env` validado por Zod; `process.exit(1)` se inválido. |

Não há segundo processo (worker/cron/consumer). Um novo entrypoint seguiria o padrão de
`src/server.ts` + um script `npm run <x>`.

## 2. Módulos (`src/modules/<domínio>/`)

Padrão por domínio: `controller.ts` (RequestHandlers `try/catch → next(err)`),
`service.ts` (regra + transações), `repository.ts` (Prisma), `routes.ts`
(`Router` + `authenticate` + `validate()`), `schemas.ts` (Zod + tipos).

| Domínio | Rotas (`/api/v1/...`) | Notas |
|---|---|---|
| `auth` | `POST /auth/register`, `POST /auth/login`, `GET /auth/me` | `AuthService.login` compara `bcrypt`, assina JWT `{ sub, email, role }` com `env.JWT_SECRET` / `env.JWT_EXPIRES_IN`. |
| `users` | `GET /users/:id` (`authenticate` + `requireRole('ADMIN')`) | Único uso de RBAC hoje. |
| `customers` | `GET /`, `GET /:id`, `POST /`, `PATCH /:id`, `DELETE /:id` | `authenticate` no router inteiro. `address` é `Json`. |
| `products` | `GET /`, `GET /:id`, `POST /`, `PATCH /:id`, `DELETE /:id` | `authenticate` no router inteiro. `priceCents`, `stockQuantity`, `active`. |
| `orders` | `GET /` (filtros), `GET /:id`, `POST /`, `PATCH /:id/status`, `DELETE /:id` | `authenticate` no router inteiro. Detalhe abaixo. |

### Módulo `orders` em detalhe

- `order.schemas.ts`: `createOrderSchema` (`customerId` uuid, `items[]` `{productId,quantity}`,
  `discountCents` >=0 default 0, `notes?`), `updateOrderStatusSchema`
  (`toStatus: nativeEnum(OrderStatus)`, `reason?`), `listOrdersQuerySchema`
  (`page`, `pageSize` 1..100, `status?`, `customerId?`, `from?`, `to?`), `orderIdParamSchema`.
- `order.repository.ts`: `list` (filtro + paginação via `$transaction([findMany, count])`),
  `findByIdWithRelations` (items+product, history asc, customer), `findById`, `deleteById`.
  `OrderWithRelations` = `Order & { items, history, customer }`.
- `order.service.ts`:
  - `list(query)` → `paginated(...)`.
  - `getById(id)` → `NotFoundError('Order')` se ausente.
  - `create(input, userId)` → `prisma.$transaction`: valida customer, valida todos os
    produtos existem e estão `active` (senão `UnprocessableEntityError('INACTIVE_PRODUCT')`),
    agrega itens duplicados, calcula `unitPriceCents`/`totalCents`/`subtotalCents`,
    valida `discountCents <= subtotal`, `reserveOrderNumber(tx)` (upsert em
    `OrderNumberSequence` → `ORD-000001`), cria `Order` com `items` e `history` inicial
    (`fromStatus: null`, `toStatus: PENDING`, `reason: 'order created'`).
  - `changeStatus(id, input, userId)` → `prisma.$transaction`: carrega order + items;
    `from === to` → `ConflictError('INVALID_STATUS_TRANSITION')`; `!canTransition` →
    `InvalidStatusTransitionError`; `shouldDebitStock` → `debitStock(tx, items)`
    (checa `stockQuantity`, senão `InsufficientStockError` com `unavailable[]`, depois
    `decrement`); `shouldReplenishStock` → `replenishStock` (`increment`);
    `tx.order.update({ status })`; `tx.orderStatusHistory.create(...)`; retorna order recarregado.
  - `delete(id)` → só se `status ∈ {PENDING, CANCELLED}`, senão
    `ConflictError('INVALID_ORDER_STATE_FOR_DELETE')`.
- `order.status.ts`: `transitions` = `{ PENDING:[PAID,CANCELLED], PAID:[PROCESSING,CANCELLED],
  PROCESSING:[SHIPPED,CANCELLED], SHIPPED:[DELIVERED], DELIVERED:[], CANCELLED:[] }`.
  `STOCK_DEBIT_TRANSITION = { from: PENDING, to: PAID }`. `shouldReplenishStock` = `to ===
  CANCELLED && from ∈ {PAID, PROCESSING}`.

## 3. Modelo de dados (`prisma/schema.prisma`)

| Entidade | Campos-chave | Índices / relações |
|---|---|---|
| `User` | `id`, `email @unique`, `passwordHash`, `name`, `role UserRole @default(OPERATOR)`, timestamps | relações: `createdOrders`, `statusHistoryChanges` |
| `Customer` | `id`, `name`, `email @unique`, `phone`, `document`, `address Json` | `@@index([document])`; `orders` |
| `Product` | `id`, `sku @unique`, `name`, `description?`, `priceCents Int`, `stockQuantity Int @default(0)`, `active Bool @default(true)` | `@@index([active])`, `@@index([name])`; `items` |
| `Order` | `id`, `orderNumber @unique`, `customerId`, `status OrderStatus @default(PENDING)`, `subtotalCents`, `discountCents`, `totalCents`, `notes?`, `createdById`, timestamps | `@@index` em `customerId`, `status`, `createdAt`, `createdById`; relações `customer`, `createdBy`, `items`, `history` |
| `OrderItem` | `id`, `orderId`, `productId`, `quantity`, `unitPriceCents`, `totalCents` | `onDelete: Cascade` no `order`; `@@index` `orderId`, `productId` |
| `OrderStatusHistory` | `id`, `orderId`, `fromStatus OrderStatus?`, `toStatus OrderStatus`, `changedAt @default(now())`, `changedById`, `reason? VarChar(500)` | `onDelete: Cascade` no `order`; `@@index` `orderId`, `changedAt` |
| `OrderNumberSequence` | `id Int @id @default(1)`, `nextValue Int @default(1)` | tabela de contador único |

Migration inicial: `prisma/migrations/20260519182739_init/migration.sql`. `datasource`
usa `DATABASE_URL` + `SHADOW_DATABASE_URL`. Seed (`prisma/seed.ts`) cria admin/operador,
10 customers (inclui "Logística Atlas Comercial Ltda"), 20 products, 26 orders em vários
estados com histórico.

## 4. Middlewares (ordem em `buildApp`)

1. `express.json({ limit: '1mb' })`
2. `requestLogger` (`src/middlewares/request-logger.middleware.ts`): `X-Request-Id`,
   `res.on('finish')` → log `http_request`.
3. (por rota) `authenticate` / `requireRole(...)` / `validate({...})`
4. 404 → `next(new NotFoundError(...))`
5. `errorMiddleware` (`src/middlewares/error.middleware.ts`): sempre por último.

## 5. Erros e códigos (`src/shared/errors/`)

| Classe | status | code |
|---|---|---|
| `BadRequestError` | 400 | `BAD_REQUEST` (configurável) |
| `ValidationError` | 400 | `VALIDATION_ERROR` (+ `details[]`) |
| `UnauthorizedError` | 401 | `UNAUTHORIZED` |
| `ForbiddenError` | 403 | `FORBIDDEN` |
| `NotFoundError` | 404 | `NOT_FOUND` (`"<Resource> not found"`) |
| `ConflictError` | 409 | `CONFLICT` (configurável, ex. `INVALID_STATUS_TRANSITION`) |
| `UnprocessableEntityError` | 422 | `UNPROCESSABLE_ENTITY` (configurável, ex. `INACTIVE_PRODUCT`) |
| `InvalidStatusTransitionError` (⊂ Conflict) | 409 | `INVALID_STATUS_TRANSITION` + `{from,to}` |
| `InsufficientStockError` (⊂ Unprocessable) | 422 | `INSUFFICIENT_STOCK` + `{unavailable[]}` |

Formato de resposta de erro: `{ "error": { "code", "message", "details"? } }`.
`AppError.details` só aparece quando definido.

## 6. Observabilidade atual

- **Logs:** Pino JSON (pretty em dev). Eventos nomeados: `server_started`,
  `shutdown_initiated`, `http_server_closed`, `bootstrap_failed`, `http_request`,
  `"Unhandled error in request"`. Redação de secrets configurada.
- **Métricas:** nenhuma (sem Prometheus/OTel).
- **Tracing:** nenhum. `X-Request-Id` é o único correlacionador.

## 7. Testes

Vitest + Supertest. `tests/setup.ts` limpa todas as tabelas em `beforeEach` (ordem
respeitando FKs). `tests/helpers/factories.ts`: `getTestApp` (cacheia `buildApp`),
`createTestUser`, `loginAndGetToken`, `bootstrapAuthenticatedUser(role)`,
`createTestCustomer`, `createTestProduct`. Cobertura atual: `tests/auth.test.ts`,
`tests/orders.test.ts` (criação, transições, estoque, filtros, delete).

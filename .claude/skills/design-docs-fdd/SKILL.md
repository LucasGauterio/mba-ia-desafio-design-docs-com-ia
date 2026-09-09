---
name: design-docs-fdd
description: >-
  Escreve o FDD (Feature Design Document): o "como implementar" em detalhe,
  acionável para um dev começar a codar. Produz docs/FDD.md com contratos,
  matriz de erros WEBHOOK_*, fluxos e a seção obrigatória "Integração com o
  sistema existente". Roda depois de design-docs-rfc; ao final aciona design-docs-diagrams.
---

# design-docs-fdd: FDD da feature

## Insumos

- `docs/adrs/*` e `docs/RFC.md` (decisões e proposta: o FDD constrói em cima, não reabre).
- `docs/_workbench/transcript-ledger.md` → requisitos funcionais, RNFs, restrições, detalhes
  técnicos secundários (formato do payload, headers, timeout 10 s, snapshot na inserção).
- `.claude/references/architecture/fdd.md` (esqueleto de 12 seções + checklist).
- `.claude/references/codebase/integration-points.md` (PE-01..PE-11): base da seção 12.
- `.claude/references/codebase/existing-app.md`.
- `.claude/rules/*`.

## Passos

1. Seguir o esqueleto de 12 seções da reference.
2. **Contratos públicos (seção 5):** ≥ 4 endpoints HTTP, cada um com auth, status codes,
   exemplo de request e de response em ```json. Cobrir no mínimo: `POST` cadastro de
   webhook, `PATCH`/`DELETE`/`GET` de configuração, `GET /webhooks/:id/deliveries`,
   `POST /admin/webhooks/dead-letter/:id/replay`. Documentar também o payload enviado ao
   cliente e a semântica dos headers `X-Event-Id`, `X-Signature`, `X-Timestamp`,
   `X-Webhook-Id`.
3. **Matriz de erros (seção 6):** tabela com códigos exclusivamente `WEBHOOK_*`
   (`WEBHOOK_NOT_FOUND`, `WEBHOOK_INVALID_URL`, `WEBHOOK_SECRET_REQUIRED`, ...), HTTP,
   condição, tratamento.
4. **Fluxos (seção 4):** outbox (inserção dentro da `$transaction` de `changeStatus`),
   worker (polling 2 s), retry (5 tentativas, 1m/5m/30m/2h/12h), DLQ (tabela separada, replay).
5. **Observabilidade (seção 8):** métricas **e** logs **e** tracing/correlação.
6. **Integração com o sistema existente (seção 12):** nomear **≥ 4 caminhos de arquivo
   reais** e descrever a integração de cada. Usar os PE do `integration-points.md`. Cobrir
   pelo menos `src/modules/orders/order.service.ts`, `src/shared/errors/*`,
   `src/middlewares/auth.middleware.ts`, `src/config/database.ts` (worker com Prisma próprio).
7. **Prosa limpa: sem `[hh:mm]` nem colchetes de timestamp** em nenhuma seção. Caminhos de
   arquivo do código (`src/...`, `prisma/...`) são conteúdo técnico e aparecem normalmente.
   A origem de cada afirmação vai para o Tracker.
8. Ao terminar o texto, **invocar `design-docs-diagrams`** para gerar
   `docs/diagrams/webhooks-diagrams.md`.
9. Atualizar as linhas `fdd` (e depois `diagrams`) em `docs/_workbench/run-state.md`.

## Saída

`docs/FDD.md` (+ `docs/diagrams/webhooks-diagrams.md` via `design-docs-diagrams`).

## Checklist antes de concluir

- [ ] 12 seções presentes, incl. "Integração com o sistema existente".
- [ ] ≥ 4 endpoints com request/response de exemplo e status codes.
- [ ] Matriz de erros só com `WEBHOOK_*`.
- [ ] Fluxos cobrem outbox, worker, retry, DLQ.
- [ ] Observabilidade cita métricas, logs e tracing.
- [ ] Seção 12 nomeia ≥ 4 caminhos de arquivo que existem no repositório.
- [ ] **Sem `[hh:mm]` nem citações de fonte no corpo.**
- [ ] Toda afirmação verificável tem linha correspondente no `docs/TRACKER.md`.
- [ ] Não repete narrativa de negócio do PRD nem reabre decisão de ADR.
- [ ] `docs/diagrams/webhooks-diagrams.md` gerado.

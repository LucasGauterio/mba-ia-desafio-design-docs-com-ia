---
name: design-docs-adr
description: >-
  Escreve os ADRs (Architecture Decision Records, formato MADR) das decisões
  arquiteturais fechadas na reunião. Produz 5 a 8 arquivos em
  docs/adrs/ADR-NNN-titulo-kebab.md. Primeira skill de autoria do workflow
  design-docs; roda depois de design-docs-baseline / -spec / -ledger.
---

# design-docs-adr: ADRs das decisões

## Insumos

- `docs/_workbench/transcript-ledger.md` → seção "Decisões fechadas" + "Detalhes técnicos secundários".
- `.claude/references/architecture/adr.md` (formato MADR, regra dos 3 Es, lista das 6
  decisões principais).
- `.claude/references/codebase/*` (para o ADR que referencia o código).
- `.claude/rules/*`.

## Passos

1. Do ledger, listar as decisões candidatas. Confirmar que cada uma passa nos 3 Es
   (estrutural, evidente, estável). Descartar detalhe operacional isolado (vai pro FDD).
2. Selecionar de 5 a 8 decisões. Cobrir **no mínimo 5 das 6 principais**:
   outbox no MySQL; retry/backoff/DLQ; HMAC-SHA256 secret por endpoint; at-least-once com
   `X-Event-Id`; worker separado em polling; reuso dos padrões do projeto.
3. Numerar sequencialmente. Nome: `ADR-NNN-titulo-em-kebab-case.md`.
4. Escrever cada ADR no formato MADR de 7 seções (ver reference). Header só Status/Data/
   Decisões relacionadas. Sem trechos de código. ≤ 5 referências de arquivo. 100-250 linhas.
5. No ADR de "reuso dos padrões existentes" (candidato natural), referenciar explicitamente
   `src/shared/errors/app-error.ts`, `src/shared/logger/index.ts`,
   `src/modules/orders/order.service.ts`, `src/app.ts`. **Pelo menos 1 ADR** precisa citar
   o código real.
6. **Links de arquivo (regra `repo-file-links.md`):** na seção "Referências" e em qualquer
   menção a arquivo real, usar **link relativo a partir de `docs/adrs/`** (`../../src/...`)
   com **âncora de linha `#Lnn`** do símbolo. Links para o RFC = `../RFC.md`; para outro
   ADR = `ADR-00X-....md`. Não linkar caminho que ainda não existe.
7. **Prosa limpa: nenhum ADR contém `[hh:mm]` nem colchetes de timestamp.** A origem de
   cada afirmação vai para `docs/TRACKER.md` (gerado depois por `design-docs-tracker`).
8. Atualizar a linha `adr` em `docs/_workbench/run-state.md`.

## Saída

`docs/adrs/ADR-001-*.md` … `docs/adrs/ADR-00N-*.md` (5 a 8 arquivos). Manter o
`docs/adrs/README.md` existente.

## Checklist antes de concluir

- [ ] 5 a 8 arquivos, formato `ADR-NNN-titulo-kebab.md`.
- [ ] Cada ADR: Status, Contexto e problema, Decisão, Alternativas consideradas (≥ 1),
      Consequências (positivas **e** negativas com trade-off), Referências.
- [ ] Conjunto cobre ≥ 5 das 6 decisões principais.
- [ ] ≥ 1 ADR referencia arquivos/módulos/classes do código.
- [ ] Sem trechos de código; ≤ 5 referências por ADR, cada uma link relativo com `#Lnn` que resolve.
- [ ] **Sem `[hh:mm]` nem citações de fonte no corpo.**
- [ ] Toda afirmação verificável tem linha correspondente no `docs/TRACKER.md`.
- [ ] Nenhuma decisão inventada; nenhum item descartado/adiado como decisão.

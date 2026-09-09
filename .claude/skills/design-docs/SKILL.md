---
name: design-docs
description: >-
  Orquestrador do workflow design-docs. Executa o pipeline completo: baseline da
  aplicação, análise da spec e da transcrição, ADRs, RFC, FDD (+ diagramas), PRD,
  Tracker e README do processo: na ordem do enunciado, parando para validar
  entre blocos. É RETOMÁVEL: lê docs/_workbench/run-state.md e continua do
  primeiro estágio não concluído. Use /design-docs para rodar ou retomar;
  /design-docs --restart para recomeçar do zero.
---

# design-docs: pipeline completo

## Retomada (fazer sempre no início)

1. Ler `.claude/design-docs.config.json`.
2. Se `docs/_workbench/run-state.md` existe e não veio `--restart`: retomar do primeiro
   estágio com status ≠ `concluído`.
3. Se não existe (ou `--restart`): criar `docs/_workbench/run-state.md` a partir do
   template abaixo, todos os estágios `pendente`.

### Template `docs/_workbench/run-state.md`

```markdown
# Estado da execução do workflow design-docs

- **run id:** <timestamp>
- **inputs:** spec=<spec> transcript=<transcript> outputDir=<outputDir>
- **iteração:** 1
- **último score de validação:**:- **próxima ação:** rodar estágio `baseline`

| Estágio | Status | Saída |
|---|---|---|
| baseline | pendente | `CLAUDE.md`, `.claude/references/codebase/*` |
| spec | pendente | `docs/_workbench/deliverables-checklist.md` |
| ledger | pendente | `docs/_workbench/transcript-ledger.md` |
| adr | pendente | `docs/adrs/ADR-*.md` |
| rfc | pendente | `docs/RFC.md` |
| fdd | pendente | `docs/FDD.md` |
| diagrams | pendente | `docs/diagrams/webhooks-diagrams.md` |
| prd | pendente | `docs/PRD.md` |
| tracker | pendente | `docs/TRACKER.md` |
| readme | pendente | `README.md` |
| validation | pendente | `docs/_workbench/validation-report.md` |
```

## Ordem de execução

Para cada estágio ainda `pendente`, marcar `em progresso`, invocar a skill, marcar
`concluído`, atualizar "próxima ação" e commitar (`chore(run): <estágio>`).

1. `baseline` → skill **design-docs-baseline**
2. `spec` → skill **design-docs-spec**
3. `ledger` → skill **design-docs-ledger**
  : **checkpoint:** revisar o ledger com o usuário antes de seguir (classificação correta?
   descartados de fora?).
4. `adr` → skill **design-docs-adr**
5. `rfc` → skill **design-docs-rfc**
  : **checkpoint:** ADR + RFC prontos para revisão.
6. `fdd` → skill **design-docs-fdd** (que aciona **design-docs-diagrams** → estágio `diagrams`)
7. `prd` → skill **design-docs-prd**
  : **checkpoint:** FDD + PRD prontos para revisão.
8. `tracker` → skill **design-docs-tracker**
9. `readme` → skill **design-docs-readme**
10. `validation` → skill **design-docs-validate**

## Ao final

- Se `validation` não atingiu 100% dos critérios: registrar as falhas em
  `.claude/workflow-build/plan-progress.md` ("Log de ciclos de teste"), ajustar os
  **assets do workflow** (não o output), incrementar `iteração` e rodar de novo do estágio
  afetado.
- Se 100%: reportar o caminho de cada artefato e o score.

## Regras

- Uma skill por estágio; nunca pular a ordem.
- Todo estágio respeita `.claude/rules/*`.
- `docs/_workbench/` não entra no entregável final `dev`.

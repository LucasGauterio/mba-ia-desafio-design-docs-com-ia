# Progresso: construção do workflow `design-docs`

> Estado da construção do workflow. Numa sessão de manutenção, ler este arquivo antes de
> agir e retomar do primeiro item `- [ ]`. Fonte do plano:
> `docs/process/design-docs-workflow-plan.md`.

- **Estado:** concluído. Pipeline validado em 34 de 34 critérios de aceite.

## Fases

- [x] 0. Setup: plano em `docs/process/design-docs-workflow-plan.md`, este registro criado.
- [x] 1. `design-docs-baseline` (skill + command) e o baseline gerado: `CLAUDE.md`,
      `.claude/references/codebase/existing-app.md`, `.claude/references/codebase/integration-points.md`.
- [x] 2. Guias de método: `references/architecture/{fdd,rfc,adr,diagrams,c4}.md`,
      `references/documentation/{taxonomy,prd,tracker,writing-style}.md`,
      `requirements/deliverables.default.md`, `guidelines/ai-as-maestro.md`,
      `references/INDEX.md`.
- [x] 3. Rules: `source-code-is-read-only`, `traceability-required`,
      `no-cross-document-duplication`, `honor-rejected-scope` + wiring no `CLAUDE.md`.
- [x] 4. Análise de inputs: `design-docs-spec`, `design-docs-ledger`.
- [x] 5. Autoria: `design-docs-adr`, `-rfc`, `-fdd`, `-diagrams`, `-prd`, `-tracker`,
      `-readme`; orquestrador `design-docs` retomável via `docs/_workbench/run-state.md`;
      `design-docs.config.json`.
- [x] 6. Validação: `design-docs-validate` com checagens mecânicas por documento.
- [x] 7. Teste em worktree `design-docs/<timestamp>` a partir de `dev`: pipeline completo,
      validação 34 de 34.
- [x] 8. Entrega em `dev`: pacote validado, `docs/_workbench/` fora da entrega.
- [x] 9. Documentação: `.claude/README.md` e `docs/process/design-docs-workflow-plan.md`.

## Iterações do ciclo de teste

1. Travessão longo (em-dash) usado como aposto nos ADRs, RFC e FDD, contra a regra de
   estilo; corrigido em massa para dois-pontos.
2. Seção "Integração com o sistema existente" do FDD começou genérica; reescrita nomeando
   os arquivos reais e a forma de integração de cada.
3. Bloco "Cobertura" do Tracker com contagens estimadas erradas; recontado por `grep`.
4. `gitleaks` barrou o commit do FDD por entropia nos valores de `secret` dos exemplos;
   trocados por placeholders sem entropia.

Detalhe completo na seção "Iterações e ajustes" do `README.md`.

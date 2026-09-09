# Progresso: construção do workflow `design-docs`

> Estado da construção do workflow. Numa sessão de manutenção, ler este arquivo antes de
> agir e retomar do primeiro item `- [ ]`. Fonte do plano:
> `DESIGN_DOCS_PROCESS.md`.

- **Estado:** concluído. Pipeline validado em 36 de 36 critérios (34 do `DESAFIO.md`, mais
  "prosa limpa" e "zero em-dash").

## Fases

- [x] 0. Setup: plano em `DESIGN_DOCS_PROCESS.md`, este registro criado.
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
      `-readme`; orquestrador `design-docs` retomável via `docs/_workbench/run-state.md`.
      Inputs (transcrição, spec, feature, revisores) derivados do documento passado na
      chamada (`/design-docs @TRANSCRICAO.md`) e gravados no `run-state.md`: sem arquivo de
      config.
- [x] 6. Validação: `design-docs-validate` com checagens mecânicas por documento.
- [x] 7. Teste em worktree `.worktrees/design-docs-run-<timestamp>` a partir de `dev`:
      pipeline completo, validação 36 de 36.
- [x] 8. Entrega em `dev`: pacote validado, `docs/_workbench/` fora da entrega.
- [x] 9. Documentação: `.claude/README.md` e `DESIGN_DOCS_PROCESS.md`.

## Iterações do ciclo de teste

1. Travessão longo (em-dash) usado como aposto nos ADRs, RFC e FDD, contra a regra de
   estilo; corrigido em massa para dois-pontos.
2. Seção "Integração com o sistema existente" do FDD começou genérica; reescrita nomeando
   os arquivos reais e a forma de integração de cada.
3. Bloco "Cobertura" do Tracker com contagens estimadas erradas; recontado por `grep`.
4. `gitleaks` barrou o commit do FDD por entropia nos valores de `secret` dos exemplos;
   trocados por placeholders sem entropia.
5. Citação de fonte inline (`[hh:mm] Nome`) em toda a prosa; regra mudada para
   rastreabilidade só no `docs/TRACKER.md`, os 4 tipos de documento regerados.
6. `docs/process/` (que o `DESAFIO.md` não previa) movido para `DESIGN_DOCS_PROCESS.md` na
   raiz. Worktrees passaram a ficar dentro do projeto, em `.worktrees/` (gitignored).
7. `.claude/design-docs.config.json` (inventado, não previsto pelo `DESAFIO.md`) removido.
   O orquestrador passou a derivar os parâmetros da transcrição informada na própria
   chamada (`/design-docs @TRANSCRICAO.md`).
8. Diagramas deixaram de ser um arquivo `docs/diagrams/<feature>-diagrams.md` separado e
   passaram a ser a última seção embutida do `docs/FDD.md` ("13. Diagramas"). A skill
   `design-docs-diagrams` passou a ser dona dessa seção.

Detalhe completo na seção "Iterações e ajustes" do `README.md`.

# Workflow `design-docs`

Transforma **uma transcrição de reunião técnica + o código de uma aplicação existente** em
um pacote de design docs rastreável (PRD, RFC, FDD, 5 a 8 ADRs, diagramas, Tracker, README
do processo). Todo o conhecimento de referência que o pipeline precisa está em
`.claude/references/`; não há dependência de fontes externas em runtime.

## Como rodar

1. Ajuste `.claude/design-docs.config.json` (`spec`, `transcript`, `outputDir`, `feature`,
   `reviewers`).
2. `/design-docs`: roda o pipeline completo, ou retoma de onde parou.
   - `/design-docs --restart` recomeça do zero.
3. Comandos por etapa, para iterar isolado:
   `/design-docs-baseline`, `/design-docs-spec`, `/design-docs-ledger`, `/design-docs-adr`,
   `/design-docs-rfc`, `/design-docs-fdd`, `/design-docs-diagrams`, `/design-docs-prd`,
   `/design-docs-tracker`, `/design-docs-readme`, `/design-docs-validate`.

Ordem do pipeline: baseline, spec, ledger, adr, rfc, fdd (+ diagrams), prd, tracker,
readme, validation. Checkpoints de revisão após `ledger`, após `adr+rfc`, após `fdd+prd`.

## O que é o quê

| Pasta | Conteúdo |
|---|---|
| `skills/design-docs*` | uma skill por etapa; `design-docs` é o orquestrador retomável |
| `commands/` | atalhos `/design-docs*` para as skills |
| `rules/` | restrições sempre ativas (código read-only, rastreabilidade, altura dos docs, escopo rejeitado) |
| `requirements/deliverables.default.md` | perfil de entregáveis (o do `DESAFIO.md`); troque para outra spec |
| `references/codebase/` | baseline da aplicação (gerado por `design-docs-baseline`) |
| `references/architecture/` | guia de FDD, RFC, ADR (MADR), diagramas, C4 |
| `references/documentation/` | guia de taxonomia, PRD, Tracker, estilo de escrita |
| `references/INDEX.md` | índice do que cada reference cobre + cobertura dos critérios de aceite |
| `workflow-build/plan-progress.md` | registro da construção do workflow |
| `guidelines/ai-as-maestro.md` | como conduzir a IA (prompts dirigidos, iteração) |

## Retomada entre sessões

- **Construção/manutenção do workflow:** `.claude/workflow-build/plan-progress.md`.
- **Execução do pipeline:** `docs/_workbench/run-state.md` (na worktree da run). O
  orquestrador lê e continua do primeiro estágio não concluído.

`docs/_workbench/` é área de trabalho: não entra no entregável final.

## Convenção de worktrees

Todas as worktrees ficam na **pasta pai do repositório** (`../`, ex. `G:/Projects/`),
nunca dentro do repo.

| Papel | Branch | Worktree |
|---|---|---|
| Entregável e workflow | `dev` | checkout principal |
| Cada execução de teste | `design-docs/<timestamp>` | `../design-docs-run-<timestamp>` |

### Runbook de uma execução de teste

```
RUN=$(date +%Y%m%d-%H%M%S)
git worktree add -b design-docs/$RUN ../design-docs-run-$RUN dev
cd ../design-docs-run-$RUN
claude -p "/design-docs" --permission-mode acceptEdits --output-format stream-json | tee run.log
claude -p "/design-docs-validate" --permission-mode acceptEdits | tee validation.md
# ler validation.md; se houver falhas, corrigir os assets em dev e refazer a run
git worktree remove ../design-docs-run-$RUN     # ao terminar
```

Correções vão sempre nos **assets do workflow** em `dev`, nunca na worktree da run. Cada
ciclo vira insumo da seção "Iterações e ajustes" do README do processo.

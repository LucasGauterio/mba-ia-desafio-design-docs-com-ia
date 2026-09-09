# Processo: workflow `design-docs`

Registro de como o pacote de design docs deste repositório foi produzido, e de como o
workflow reutilizável foi construído. Companion do [`README.md`](README.md) da raiz (que é
o README do processo exigido pelo desafio) e da doc do workflow em
[`.claude/README.md`](.claude/README.md).

## Objetivo

Em vez de escrever cada documento à mão, foi construído um **workflow do Claude Code**
(`.claude/`) que transforma uma transcrição de reunião mais o código de uma aplicação
existente em um pacote de design docs rastreável (PRD, RFC, FDD, 5 a 8 ADRs, diagramas,
Tracker, README do processo). O mesmo workflow serve para transcrições futuras.

**Restrições:** entrega puramente documental (não tocar em `src/`, `prisma/`, `tests/`,
configs); todo item rastreável à transcrição ou a um arquivo real, com a origem
materializada no `docs/TRACKER.md` (a prosa dos documentos fica limpa, sem timestamps);
nada inventado; itens que a reunião descartou ou adiou não viram requisito.

## Design do workflow

### Skills e commands (`.claude/skills/`, `.claude/commands/`)

Nome igual ao resultado da execução. Prefixo `design-docs-` agrupa tudo.

| Skill / command | Resultado |
|---|---|
| `design-docs` (orquestrador, retomável) | roda o pipeline completo |
| `design-docs-baseline` | `CLAUDE.md` e `.claude/references/codebase/*` |
| `design-docs-spec` | `docs/_workbench/deliverables-checklist.md` |
| `design-docs-ledger` | `docs/_workbench/transcript-ledger.md` |
| `design-docs-adr` | `docs/adrs/ADR-NNN-*.md` |
| `design-docs-rfc` | `docs/RFC.md` |
| `design-docs-fdd` | `docs/FDD.md` |
| `design-docs-diagrams` | `docs/diagrams/<feature>-diagrams.md` |
| `design-docs-prd` | `docs/PRD.md` |
| `design-docs-tracker` | `docs/TRACKER.md` |
| `design-docs-readme` | `README.md` reescrito |
| `design-docs-validate` | `docs/_workbench/validation-report.md` e score |

### Rules (`.claude/rules/`, sempre ativas)

`source-code-is-read-only`, `traceability-required`, `no-cross-document-duplication`,
`honor-rejected-scope`.

### References (`.claude/references/`, guias de método autocontidos)

- `codebase/existing-app.md`, `codebase/integration-points.md` (gerados pelo baseline).
- `architecture/fdd.md`, `architecture/rfc.md`, `architecture/adr.md`,
  `architecture/diagrams.md`, `architecture/c4.md`.
- `documentation/taxonomy.md`, `documentation/prd.md`, `documentation/tracker.md`,
  `documentation/writing-style.md`.
- `requirements/deliverables.default.md` (perfil de entregáveis; troca para outra spec).
- `guidelines/ai-as-maestro.md`.
- `references/INDEX.md` (índice do que cada reference cobre).

### Entrada da run (sem arquivo de config)

Não há arquivo de configuração. A transcrição é informada na própria chamada:
`/design-docs @TRANSCRICAO.md`. O orquestrador resolve os parâmetros e os grava no bloco
`inputs resolvidos` de `docs/_workbench/run-state.md`:

- `transcript`: o documento passado na chamada.
- `spec`: 2º argumento, senão `DESAFIO.md` se existir, senão `(nenhuma)` (só o perfil
  `.claude/requirements/deliverables.default.md`).
- `outputDir`: `docs` (fixo).
- `feature.name` / `feature.slug`: derivados do título da transcrição.
- `reviewers`: a lista de participantes da transcrição.

Para rodar em outra transcrição ou aplicação, basta chamar `/design-docs` apontando o novo
documento.

## Fases da construção

0. **Setup.** Persistir este documento (`DESIGN_DOCS_PROCESS.md` na raiz) e criar o registro
   de progresso em `.claude/workflow-build/plan-progress.md`.
1. **`design-docs-baseline`.** Skill que inspeciona o código (ignorando `DESAFIO.md`,
   `TRANSCRICAO.md`, `README.md`) e escreve `CLAUDE.md` e `.claude/references/codebase/`.
2. **Guias de método.** Escrever `.claude/references/architecture/*`,
   `.claude/references/documentation/*`, `requirements/deliverables.default.md`,
   `guidelines/ai-as-maestro.md`, autocontidos.
3. **Rules.** Os quatro arquivos de `.claude/rules/` e o wiring no `CLAUDE.md`.
4. **Análise de inputs.** Skills `design-docs-spec` e `design-docs-ledger`.
5. **Autoria e orquestrador.** As sete skills de autoria e `design-docs` (retomável via
   `docs/_workbench/run-state.md`).
6. **Validação.** Skill `design-docs-validate` com checagens mecânicas por documento.
7. **Teste em worktree.** Runbook no `.claude/README.md`; rodar o pipeline numa worktree
   `.worktrees/design-docs-run-<timestamp>` a partir de `dev`, validar, iterar.
8. **Entrega.** Trazer o pacote validado para `dev`, revalidar contra os critérios de
   aceite, remover a área de trabalho `docs/_workbench/`.
9. **Documentar.** `.claude/README.md` e este arquivo.

## Rastreamento de progresso (retomável entre sessões)

- **Construção e manutenção do workflow:** `.claude/workflow-build/plan-progress.md`.
- **Execução do pipeline:** `docs/_workbench/run-state.md` na worktree da run. O
  orquestrador lê e continua do primeiro estágio não concluído. `docs/_workbench/` não
  entra no entregável final.

## Convenção de worktrees

As worktrees de execução ficam **dentro do projeto**, em `.worktrees/` (gitignored).

| Papel | Branch | Worktree |
|---|---|---|
| Entregável e workflow | `dev` | checkout principal |
| Cada execução de teste | `design-docs/<timestamp>` | `.worktrees/design-docs-run-<timestamp>` |

## Teste e iterações

O pipeline foi rodado de ponta a ponta numa worktree criada a partir de `dev`. Resultado da
validação: **36 de 36 critérios** (os 34 de aceite do `DESAFIO.md` mais "prosa limpa" e
"zero em-dash"). As correções feitas durante os ciclos estão detalhadas na seção "Iterações
e ajustes" do [`README.md`](README.md): revisões do plano; git bloqueado pelo ambiente;
travessão longo (em-dash) violando a regra de estilo; a seção de integração do FDD começou
genérica; contagens do Tracker feitas de cabeça; `gitleaks` barrando secrets de exemplo com
entropia alta; citação de fonte inline movida para o Tracker; arquivo de config inventado
removido em favor da transcrição informada na chamada.

## Estrutura final da entrega

```
CLAUDE.md                     # baseline da aplicação
DESAFIO.md                    # enunciado original (preservado)
DESIGN_DOCS_PROCESS.md        # este arquivo
TRANSCRICAO.md                # transcrição (não alterada)
README.md                     # README do processo
docs/
  PRD.md  RFC.md  FDD.md  TRACKER.md
  adrs/ADR-001..007-*.md  adrs/README.md
  diagrams/webhooks-diagrams.md
.claude/
  README.md
  skills/design-docs*/SKILL.md
  commands/design-docs*.md
  rules/*.md  requirements/*.md  guidelines/*.md
  references/codebase/*  references/architecture/*  references/documentation/*
  references/INDEX.md
  workflow-build/plan-progress.md
```

`docs/_workbench/` (área de trabalho da run) não entra na entrega.

## Decisões de organização

- Workflow chamado `design-docs`; skills e commands com o nome por extenso; o nome é igual
  ao resultado da execução.
- Sem arquivo de config: os parâmetros da run saem da transcrição informada na chamada
  (`/design-docs @TRANSCRICAO.md`) e ficam no `docs/_workbench/run-state.md`.
- Workflow self-contained: skills e references próprias para tudo, inclusive ADR e
  diagramas. Plugins de terceiros ficam como alternativa opcional.
- Diagramas Mermaid gerados por padrão junto do FDD, em `docs/diagrams/`.
- RFC ocupa a altura de arquitetura (proposta submetida a revisão), com estrutura
  construída a partir do requisito 2 do enunciado.
- Ordem de autoria: ADR, RFC, FDD com diagramas, PRD, Tracker, README.
- Rastreabilidade só no `docs/TRACKER.md`; a prosa dos documentos fica limpa.
- Progresso retomável em dois níveis (construção do workflow e execução do pipeline).

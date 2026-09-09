---
name: design-docs-readme
description: >-
  Reescreve o README.md da raiz com a documentação do PROCESSO de produção do
  pacote de design docs: ferramentas de IA, workflow adotado, prompts
  customizados, iterações e ajustes, como navegar a entrega. Roda por último.
---

# design-docs-readme: README do processo

## Insumos

- `.claude/workflow-build/plan-progress.md` → "Log de ciclos de teste" (iterações reais).
- `docs/_workbench/run-state.md` e `validation-report.md` (o que passou/falhou por ciclo).
- As skills e commands do workflow (fonte dos "prompts customizados").
- `docs/process/design-docs-workflow-plan.md` (referência do processo completo).

## Estrutura obrigatória do novo README

```markdown
# <título>

## Sobre o desafio
[1 a 2 parágrafos, nas suas palavras: transformar a transcrição + o código em PRD, RFC,
FDD, ADRs, Tracker e este README, sem inventar requisitos.]

## Ferramentas de IA utilizadas
- **Claude Code (Sonnet 5)**: leitura do repo e da transcrição, construção do workflow
  `design-docs` (skills/commands/rules/references) e geração dos documentos.
- [outras, se usadas, com o papel de cada uma]

## Workflow adotado
[Ordem: baseline do código → ledger da transcrição + checklist da spec → ADRs → RFC →
FDD (+ diagramas) → PRD → Tracker → este README. Como a interação com a IA foi organizada:
skills por documento, references de método em `.claude/references/`, worktrees por execução.]

## Prompts customizados
Pelo menos 2, em blocos de código. Ex.: o corpo de `.claude/skills/design-docs-ledger/SKILL.md`
(extração dirigida por bucket) e de `.claude/skills/design-docs-fdd/SKILL.md` (seção de
integração com ≥ 4 arquivos reais).

## Iterações e ajustes
Pelo menos 2 momentos concretos em que a IA errou/ficou superficial e foi corrigida
(do "Log de ciclos de teste" em plan-progress.md). Quantos ciclos principais até o resultado.

## Como navegar a entrega
Caminho dos arquivos e ordem sugerida de leitura:
1. `CLAUDE.md`: contexto da aplicação
2. `docs/PRD.md`: por quê e o quê
3. `docs/RFC.md`: proposta técnica
4. `docs/adrs/`: decisões
5. `docs/FDD.md` + `docs/diagrams/`: como implementar
6. `docs/TRACKER.md`: rastreabilidade
7. `docs/process/design-docs-workflow-plan.md`: o processo completo
```

Manter, se quiser, um link para o enunciado original (está em `DESAFIO.md`).

## Checklist antes de concluir

- [ ] 6 seções obrigatórias presentes.
- [ ] ≥ 1 ferramenta de IA listada com o papel.
- [ ] ≥ 2 prompts customizados em blocos de código.
- [ ] ≥ 2 iterações/ajustes concretos descritos.
- [ ] Ordem de leitura com caminhos reais.
- [ ] Atualizar a linha `readme` em `docs/_workbench/run-state.md`.

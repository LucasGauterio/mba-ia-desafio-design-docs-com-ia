---
name: design-docs-diagrams
description: >-
  Gera os diagramas Mermaid do FDD em um único arquivo
  docs/diagrams/<feature>-diagrams.md (fluxo da outbox, worker, retry/DLQ,
  máquina de estados do evento, contratos, replay). Acionada por design-docs-fdd,
  ou isolada via /design-docs-diagrams depois que o FDD existe.
---

# design-docs-diagrams: diagramas do FDD

## Insumos

- `docs/FDD.md` (fonte única; não inventar elementos fora dele).
- `.claude/references/architecture/diagrams.md` (tipos, guardrails de sintaxe, lista dos
  diagramas esperados).
- `docs/_workbench/run-state.md` → `inputs resolvidos` → `feature.slug` (nomeia o arquivo
  de saída). Sem run-state, derivar um slug curto do título do FDD.

## Passos

1. Ler o FDD por completo; detectar idioma (PT) e elementos explícitos.
2. Para cada diagrama candidato, aplicar o teste de significância (5 perguntas da
   reference). Descartar redundantes.
3. Gerar 4 a 6 diagramas (máx. 10) num único arquivo, cada um com título, parágrafo
   descritivo de 3 a 5 frases, bloco ```mermaid e notas.
4. Respeitar os guardrails de sintaxe Mermaid (IDs ASCII, labels ≤ 3 palavras, `<br/>`,
   sem `min(`/`++`/`{}` em labels, sequence vs flowchart não se misturam, sem emoji).
5. Texto em PT com acentos; termos técnicos em inglês.
6. Revisão interna: reler FDD + arquivo gerado, corrigir inconsistências e elementos
   inventados.
7. Atualizar a linha `diagrams` em `docs/_workbench/run-state.md`.

## Saída

`docs/diagrams/<feature.slug>-diagrams.md` (slug do run-state; para o desafio dos webhooks,
`docs/diagrams/webhooks-diagrams.md`).

## Checklist antes de concluir

- [ ] Um único arquivo.
- [ ] 4 a 10 diagramas, cada um passando no teste de significância, sem redundância.
- [ ] Nenhum elemento ausente do FDD.
- [ ] Sintaxe Mermaid válida (guardrails).
- [ ] PT com acentos; termos técnicos em inglês; labels curtos.

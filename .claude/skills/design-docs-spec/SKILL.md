---
name: design-docs-spec
description: >-
  Segundo passo do workflow design-docs. Lê a especificação do desafio/entrega
  (DESAFIO.md por padrão, ou o caminho em .claude/design-docs.config.json) e o
  perfil .claude/requirements/deliverables.default.md, e emite
  docs/_workbench/deliverables-checklist.md: a lista de artefatos + todos os
  critérios de aceite como checklist verificável. Roda depois de design-docs-baseline.
---

# design-docs-spec: checklist de entregáveis

## Objetivo

Transformar a spec em uma lista fechada e verificável do que precisa ser produzido, para
guiar as skills de autoria e a validação final.

## Insumos

- `.claude/design-docs.config.json` → campo `spec` (default `DESAFIO.md`).
- `.claude/requirements/deliverables.default.md` (perfil já destilado do enunciado).
- Se a spec informada **difere** do desafio padrão, releia a spec e ajuste o perfil.

## Passos

1. Ler o perfil e a spec.
2. Para cada artefato: caminho de saída, skill responsável, seções obrigatórias, contagens
   mínimas.
3. Consolidar **todos** os critérios de aceite (do perfil + da spec) como itens `- [ ]`.
4. Incluir a lista de itens que a spec/reunião descartou ou adiou (de
   `.claude/rules/honor-rejected-scope.md`), para a validação anti-escopo.
5. Escrever `docs/_workbench/deliverables-checklist.md`.
6. Atualizar a linha `spec` em `docs/_workbench/run-state.md` (se existir).

## Saída: `docs/_workbench/deliverables-checklist.md`

```markdown
# Checklist de entregáveis

Fonte: <spec> + .claude/requirements/deliverables.default.md

## Artefatos
| Artefato | Caminho | Skill | Seções obrigatórias | Contagens mínimas |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |

## Critérios de aceite
### PRD
- [ ] ...
### RFC
- [ ] ...
### FDD
- [ ] ...
### ADRs
- [ ] ...
### Tracker
- [ ] ...
### README
- [ ] ...
### Consistência geral
- [ ] ...

## Fora de escopo (não pode virar requisito)
- <item>: <origem>
```

## Regra

Não inventar critérios que a spec não define. Se a spec é omissa em algo, marcar
`(não especificado pela spec)` em vez de criar exigência.

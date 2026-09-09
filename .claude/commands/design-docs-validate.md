---
description: Valida o pacote de design docs contra os critérios de aceite
---

Invoque a skill **design-docs-validate**. Confere os artefatos (`docs/PRD.md`, `docs/RFC.md`,
`docs/FDD.md`, `docs/adrs/*`, `docs/TRACKER.md`, `README.md`) contra
`docs/_workbench/deliverables-checklist.md` e o enunciado, executando as checagens
mecânicas com `grep`/`rg`. Escreve `docs/_workbench/validation-report.md` com cada critério
PASS/FAIL + evidência e um score `N/total`. Ao final, reporte o score e a lista de falhas.

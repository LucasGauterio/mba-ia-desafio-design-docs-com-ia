---
description: Roda (ou retoma) o pipeline completo do workflow design-docs
argument-hint: "[--restart]"
---

Invoque a skill **design-docs** (orquestrador) e execute o pipeline.

- Sem argumento: retoma de `docs/_workbench/run-state.md`, do primeiro estágio não concluído.
- `--restart`: recomeça do zero.

Ordem: baseline → spec → ledger → adr → rfc → fdd (+ diagrams) → prd → tracker → readme →
validation. Pare nos checkpoints (após ledger, após adr+rfc, após fdd+prd) para revisão.
Ao final, reporte o caminho de cada artefato e o score da validação.

Argumento recebido: `$ARGUMENTS`

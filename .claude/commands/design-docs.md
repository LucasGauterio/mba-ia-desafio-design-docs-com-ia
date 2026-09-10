---
description: Roda (ou retoma) o pipeline completo do workflow design-docs
argument-hint: "<transcrição> [spec] [--restart] [--nao-interativo]"
---

Invoque a skill **design-docs** (orquestrador) e execute o pipeline.

- `/design-docs @TRANSCRICAO.md`: começa uma run nova a partir dessa transcrição. O
  orquestrador deriva feature, slug e revisores do próprio documento e grava tudo em
  `docs/_workbench/run-state.md`. Não há arquivo de config.
- 2º argumento opcional: caminho da spec (default `DESAFIO.md` se existir).
- Sem argumento: retoma de `docs/_workbench/run-state.md`, do primeiro estágio não concluído.
- `--restart`: recomeça do zero (exige a transcrição de novo).
- `--nao-interativo` (ou rodando via `claude -p`, sem TTY): não para nos checkpoints;
  registra a nota de revisão no run-state e segue. Commits por estágio viram best-effort.

Ordem: baseline → spec → ledger → adr → rfc → fdd (+ diagrams) → prd → tracker → readme →
validation. No modo interativo, pare nos checkpoints (após ledger, após adr+rfc, após
fdd+prd) para revisão. Ao final, reporte o caminho de cada artefato e o score da validação.

Argumento recebido: `$ARGUMENTS`

---
description: Extrai da transcrição o ledger classificado (docs/_workbench/transcript-ledger.md)
---

Invoque a skill **design-docs-ledger** e execute-a por completo.

Lê a transcrição (`.claude/design-docs.config.json` → `transcript`, default
`TRANSCRICAO.md`) e escreve `docs/_workbench/transcript-ledger.md` com decisões fechadas,
requisitos funcionais, RNFs, restrições/ganchos com o código, descartados, adiados e
detalhes técnicos secundários. Cada item com origem `[hh:mm] Nome`.

Extração dirigida, não resumo. Ao final, reporte a contagem por bucket.

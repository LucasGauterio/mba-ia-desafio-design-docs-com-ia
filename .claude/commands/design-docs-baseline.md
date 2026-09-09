---
description: Gera o baseline da aplicação existente (CLAUDE.md + .claude/references/codebase/)
---

Invoque a skill **design-docs-baseline** e execute-a por completo.

Produz, a partir do código-fonte (ignorando `DESAFIO.md`, `TRANSCRICAO.md`, `README.md`):

- `CLAUDE.md` na raiz: contexto da aplicação como ela é hoje
- `.claude/references/codebase/existing-app.md`: mapa detalhado
- `.claude/references/codebase/integration-points.md`: pontos de extensão para o FDD

Não altere `src/`, `prisma/`, `tests/` nem configs. Ao final, reporte os três caminhos
gerados e um resumo de 3 linhas do que a aplicação é.

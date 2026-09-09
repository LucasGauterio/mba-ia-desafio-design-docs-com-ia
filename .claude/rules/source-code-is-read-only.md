# Rule · o código-fonte é somente leitura

A entrega deste repositório é **documental**. Nunca edite, crie ou remova arquivos em:

- `src/`
- `prisma/` (schema, migrations, seed)
- `tests/`
- configs de build, lint, formatação, CI (`tsconfig*.json`, `.eslintrc*`, `.prettierrc*`,
  `vitest.config.ts`, `package.json`, `docker-compose.yml`, `.env*`)

`TRANSCRICAO.md` também não muda.

O código serve de **contexto e referência**. Se um documento precisa de uma mudança de
código para funcionar, descreva a mudança no FDD ("como `changeStatus` seria estendido"),
não a implemente.

Você pode criar/editar apenas: `docs/**`, `CLAUDE.md`, `README.md`, `.claude/**`.

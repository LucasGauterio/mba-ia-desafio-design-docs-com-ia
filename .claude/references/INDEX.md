# Ãndice das references

As references são guias compactos e autocontidos, um por tipo de documento. O workflow
`design-docs` consome estes arquivos em runtime; não depende de nenhuma fonte externa.

## Índice

| Reference | Cobre |
|---|---|
| `documentation/taxonomy.md` | a altura e o papel de cada documento (PRD, RFC, ADR, FDD, Tracker) e as fronteiras entre eles |
| `documentation/prd.md` | quando uma feature merece PRD, estrutura mínima e esqueleto de saída Markdown, checklist |
| `documentation/tracker.md` | formato obrigatório da tabela, esquema de IDs, limiares de cobertura, heurística anti-alucinação |
| `documentation/writing-style.md` | idioma, forma, o que não escrever, checagem final por documento |
| `architecture/fdd.md` | esqueleto de 12 seções do FDD, contratos com exemplos, matriz de erros, seção obrigatória de integração, checklist |
| `architecture/rfc.md` | estrutura do RFC (2 a 4 páginas), metadados, alternativas, questões em aberto, checklist |
| `architecture/adr.md` | formato MADR de 7 seções, regra dos 3 Es, "uma decisão por ADR", proibições, checklist |
| `architecture/diagrams.md` | quando um diagrama se justifica, tipos, guardrails de sintaxe Mermaid, estrutura do arquivo |
| `architecture/c4.md` | os quatro níveis do modelo C4 (opcional neste pacote) |
| `requirements/deliverables.default.md` | perfil de entregáveis do desafio: artefatos, seções, contagens, critérios de aceite, itens fora de escopo |
| `guidelines/ai-as-maestro.md` | papel de maestro, prompts dirigidos, expectativa de iteração, filtragem do que não entra |
| `codebase/existing-app.md` | mapa detalhado da aplicação existente (gerado por `design-docs-baseline`) |
| `codebase/integration-points.md` | catálogo de pontos de extensão do código, insumo da seção de integração do FDD |

## Cobertura dos critérios de aceite

Cada critério do `DESAFIO.md` tem uma reference que o suporta:

| Critério (resumo) | Reference |
|---|---|
| PRD: seções, mínimo de 8 requisitos, objetivo com meta, fora de escopo, riscos | `documentation/prd.md` + `requirements/deliverables.default.md` |
| RFC: seções, alternativas com trade-off, questões em aberto, links de ADR | `architecture/rfc.md` |
| FDD: 12 seções, endpoints, erros `WEBHOOK_*`, arquivos reais, observabilidade | `architecture/fdd.md` + `codebase/integration-points.md` |
| ADRs: 5 a 8 arquivos MADR, cobre 5 das 6 decisões, ao menos 1 cita código | `architecture/adr.md` |
| Tracker: formato e limiares | `documentation/tracker.md` |
| README: seções, prompts, iterações | skill `design-docs-readme` |
| Consistência: nada contradiz transcrição/código; nenhum arquivo inexistente citado | `rules/*` + skill `design-docs-validate` |

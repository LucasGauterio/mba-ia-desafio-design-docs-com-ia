# Rule · referência a arquivo do repositório é link relativo com linha

Toda vez que um documento entregue (`docs/**`, `CLAUDE.md`, `README.md`) menciona **um
arquivo que existe no repositório**, essa menção é um **link relativo em Markdown**, não
texto plano. Objetivo: navegar direto ao arquivo e tornar a validação verificável por
clique.

## Formato

| Alvo | Como linkar | Exemplo (a partir de `docs/FDD.md`) |
|---|---|---|
| Código / config / teste, citando um trecho específico | link relativo **com âncora de linha** `#Lnn` (ou range `#Lnn-Lmm`) | `[\`order.service.ts\`](../src/modules/orders/order.service.ts#L120)` |
| Código / config, referência ao arquivo inteiro | link relativo sem âncora | `[\`prisma/schema.prisma\`](../prisma/schema.prisma)` |
| Outro documento do pacote, apontando uma seção | link relativo `+ #ancora-da-secao` (slug do heading) | `[RFC, "Proposta técnica"](RFC.md#proposta-técnica)` |
| Outro documento do pacote, referência ao arquivo | link relativo sem âncora | `[PRD](PRD.md)` |
| `TRANSCRICAO.md` (é arquivo do repo) | link relativo com âncora de linha da fala | `[\[09:17\] Diego](../TRANSCRICAO.md#L142)` |

O caminho é **relativo ao arquivo que está linkando**:

- de `CLAUDE.md` (raiz) → `src/...`, `prisma/...`
- de `docs/FDD.md`, `docs/PRD.md`, `docs/RFC.md`, `docs/TRACKER.md` → `../src/...`, `../TRANSCRICAO.md`
- de `docs/adrs/ADR-XXX.md` → `../../src/...`, `../../TRANSCRICAO.md`; outro ADR ou o FDD → `ADR-002-....md`, `../FDD.md`

## Âncora de linha: como obter

Abrir o arquivo, localizar o símbolo (função, classe, bloco de schema, middleware, campo) e
usar a linha real. Preferir a linha da **declaração** (`export function changeStatus` ,
`class AppError`, `model WebhookOutbox`). Para um bloco, usar range `#L120-L145`.

As âncoras `#Lnn` são **estáveis** neste repositório porque `src/`, `prisma/`, `tests/` e
os configs são somente leitura (ver [`source-code-is-read-only.md`](source-code-is-read-only.md)).
Não use `#Lnn` para apontar outro documento do pacote (esses são regerados): use a âncora
de seção.

## Escopo e exceções

- Vale para arquivos que **existem hoje**. Arquivos propostos pela feature e ainda
  inexistentes (`src/worker.ts`, `src/modules/webhooks/*`, tabelas novas) continuam como
  `code span` simples (`` `src/worker.ts` ``), com nota de que são novos. Nunca linkar para
  um caminho que não resolve.
- Primeira menção de um arquivo em cada seção leva o link. Repetições imediatas no mesmo
  parágrafo podem ficar como `code span` para não poluir.
- O texto visível do link é o nome ou caminho do arquivo (`` `order.service.ts` `` ou
  `` `src/modules/orders/order.service.ts` ``), nunca uma URL crua.
- Isto **não** reabre a regra de rastreabilidade: a origem de cada item continua só no
  `docs/TRACKER.md` (ver [`traceability-required.md`](traceability-required.md)). Um link
  para código é conteúdo técnico de navegação, não citação de fonte na prosa.

## Falha comum

Escrever `src/modules/orders/order.service.ts` como texto plano no meio de uma frase.
Corrigir para o link relativo com a linha do símbolo citado.

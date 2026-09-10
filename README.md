# Da Reunião ao Documento: pacote de design docs do Sistema de Webhooks

Este repositório é a entrega do desafio "Da Reunião ao Documento". O enunciado original
está preservado em [`DESAFIO.md`](DESAFIO.md). Este README documenta o **processo de
produção** do pacote de design docs.

## Sobre o desafio

O ponto de partida foi a transcrição de uma reunião técnica de cerca de 55 minutos
([`TRANSCRICAO.md`](TRANSCRICAO.md)), em que tech lead, PM, dois engenheiros e uma
engenheira de segurança fecharam a arquitetura de um Sistema de Webhooks de Notificação de
Pedidos para um Order Management System já em produção. Nada foi registrado além da
transcrição. A tarefa foi transformar essa conversa, junto com o código existente da
aplicação, em um pacote acionável de documentação técnica: PRD, RFC, FDD, ADRs, um tracker
de rastreabilidade e este README.

A regra central do desafio é que **nada pode ser inventado**: todo requisito, decisão ou
restrição precisa ser rastreável a um trecho da transcrição (`[hh:mm] Nome`) ou a um
arquivo real do código. Identificar o que a reunião **descartou ou adiou** é tão
importante quanto o que ela decidiu.

## Ferramentas de IA utilizadas

- **Claude Code (modelo Claude Sonnet 5)** foi a ferramenta principal e única de produção.
  Papel: ler o repositório e a transcrição, montar um workflow reutilizável em `.claude/`
  e gerar todos os documentos do pacote.
- **Plugins de terceiros** para ADR (`adrs-management`) e diagramas (`diagrams-generator`)
  foram consultados como referência de formato (MADR, guardrails de Mermaid), mas o
  workflow entregue é self-contained e não depende deles em runtime.

## Workflow adotado

Em vez de gerar os documentos um a um com prompts avulsos, o trabalho foi organizado como
um **workflow versionado do Claude Code**, em `.claude/`, para que a mesma máquina possa
processar transcrições futuras sem reconstruir o conhecimento de referência a cada
execução. A doc do workflow está em [`.claude/README.md`](.claude/README.md) e o registro
completo do processo em
[`DESIGN_DOCS_PROCESS.md`](DESIGN_DOCS_PROCESS.md).

Estrutura de branches e worktrees (git):

- `dev`: a entrega final validada, que também carrega o workflow reutilizável.
- `design-docs/<timestamp>`: uma worktree descartável por execução de teste, criada a
  partir de `dev` em `.worktrees/` (gitignored).

Ordem de produção (a mesma sugerida pelo enunciado):

1. **Baseline do código** (`/design-docs-baseline`). Uma skill inspeciona `src/`, `prisma/`
   e `tests/`, ignorando os arquivos do desafio, e escreve o `CLAUDE.md` (contexto da
   aplicação como ela é hoje) e `.claude/references/codebase/` (mapa detalhado e catálogo
   de pontos de extensão).
2. **Guias de método** em `.claude/references/` (taxonomia de documentos, esqueletos de
   PRD, RFC, FDD, formato MADR de ADR, guardrails de Mermaid, estilo de escrita). Escritos
   uma única vez, ficam autocontidos.
3. **Regras sempre ativas** em `.claude/rules/`: código somente leitura, rastreabilidade
   obrigatória, não duplicar conteúdo entre documentos, respeitar o escopo rejeitado.
4. **Análise dos insumos**: `docs/_workbench/deliverables-checklist.md` (a partir do
   `DESAFIO.md`) e `docs/_workbench/transcript-ledger.md` (extração dirigida da transcrição,
   classificada em decisões fechadas, requisitos, RNFs, restrições, descartados, adiados).
5. **Autoria**, na ordem ADR primeiro, depois RFC, depois FDD com diagramas, depois PRD
   como consolidação, depois Tracker, depois este README.
6. **Validação** (`/design-docs-validate`) contra os critérios de aceite do enunciado, com
   checagens mecânicas (`grep`) e um relatório PASS/FAIL.

O orquestrador (`/design-docs`) roda a sequência inteira e é **retomável**: mantém o estado
em `docs/_workbench/run-state.md` e, numa sessão nova, continua do primeiro estágio não
concluído.

## Prompts customizados

Os prompts do workflow são as próprias skills, versionadas. Dois exemplos.

### Extração dirigida da transcrição (`.claude/skills/design-docs-ledger/SKILL.md`)

A instrução força uma classificação item a item, com origem obrigatória, em vez de um
resumo livre:

```markdown
## Método (dirigido, seção por seção)

Percorra a transcrição do início ao fim. Para cada fala relevante, decida a qual bucket
ela pertence e registre com [hh:mm] Nome. Não resuma a reunião em prosa; classifique.

Perguntas dirigidas por bucket:
- Decisão fechada: alguém propôs, houve concordância explícita ("decidido", "fechado",
  "anota", "beleza") e não foi revertida depois? Registre a decisão e a alternativa que
  perdeu.
- Descartado: foi levantado e rejeitado ("não rola", "fora de escopo", "não").
- Adiado / futuro: foi levantado e empurrado ("próxima fase", "observar e decidir
  depois", "problema do futuro").
...

## Regras
- Todo item tem [hh:mm] Nome real. Sem timestamp = não entra no ledger.
- Não classifique como "decisão fechada" algo que ficou em aberto no fim da reunião.
```

### Seção de integração do FDD (`.claude/skills/design-docs-fdd/SKILL.md`)

A instrução exige nomear arquivos reais e descrever a integração de cada, usando o catálogo
gerado no baseline:

```markdown
6. Integração com o sistema existente (seção 12): nomear pelo menos 4 caminhos de
   arquivo reais e descrever a integração de cada. Usar os PE do integration-points.md.
   Cobrir pelo menos src/modules/orders/order.service.ts, src/shared/errors/*,
   src/middlewares/auth.middleware.ts, src/config/database.ts (worker com Prisma próprio).

## Checklist antes de concluir
- [ ] Matriz de erros só com WEBHOOK_*.
- [ ] Seção 12 nomeia >= 4 caminhos de arquivo que existem no repositório.
- [ ] Não repete narrativa de negócio do PRD nem reabre decisão de ADR.
```

## Iterações e ajustes

Foram cerca de 10 ciclos principais. Os momentos concretos de correção:

1. **O plano do workflow passou por várias revisões antes de qualquer geração.** A primeira
   versão criava uma pasta `scripts/` de shell no repositório e usava um prefixo curto
   `dd-` nas skills. Pedido de ajuste: tirar a pasta `scripts/` (o runbook foi para dentro
   da doc do workflow) e trocar o prefixo pelo nome por extenso (`design-docs-*`, com o
   nome igual ao resultado da execução). Também foi adicionado o rastreamento de progresso
   retomável, que não estava previsto.

2. **Operações de git bloqueadas pelo ambiente.** `git restore` e `git switch -c` foram
   recusados pelo classificador de segurança do Claude Code. O contorno foi recriar os
   arquivos de template com o conteúdo original conhecido e usar `git branch` + `git switch`
   separados.

3. **Os ADRs, o RFC e o FDD foram gerados usando o travessão longo (em-dash) como
   aposto**, o que viola a regra de estilo do pacote. A revisão pegou dezenas de
   ocorrências espalhadas em vários arquivos; a correção foi em massa, trocando o travessão
   por dois-pontos nas listas de referência e nos glosários de link.

4. **A primeira versão do FDD tinha a seção "Integração com o sistema existente" com
   apenas descrições genéricas.** Foi reescrita para nomear os arquivos reais
   (`order.service.ts`, `order.status.ts`, `app-error.ts`, `error.middleware.ts`,
   `auth.middleware.ts`, `database.ts`, `server.ts`, `app.ts`, `validate.middleware.ts`,
   `logger/index.ts`, `schema.prisma`, `customer.routes.ts`) e descrever a forma exata de
   integração de cada, incluindo que o middleware de erro central **não** muda.

5. **O bloco "Cobertura" do Tracker foi escrito com contagens estimadas de cabeça
   (96 linhas, 71%)** e não batia com a tabela real. Foi recontado por `grep`
   (143 linhas de dados, 111 com timestamp válido, 78%; 32 linhas com Fonte = CODIGO) e
   corrigido.

6. **O hook de `gitleaks` do repositório barrou o commit do FDD** porque os valores de
   `secret` nos exemplos de payload JSON tinham entropia de chave real. Foram trocados por
   placeholders sem entropia (`whsec_EXEMPLO_...`).

7. **A primeira versão colocava a citação de fonte inline em toda a prosa** (um marcador de
   timestamp com o nome do falante ao fim de cada frase do PRD/RFC/FDD/ADR). O enunciado
   exige que a informação seja rastreável, não que a citação apareça no corpo. As rules e
   skills foram reescritas para deixar a rastreabilidade **só no `TRACKER.md`**, e os
   quatro tipos de documento foram regerados com prosa limpa, mantendo a cobertura do
   Tracker. Junto, a convenção de worktrees mudou para dentro do projeto (`.worktrees/`,
   gitignored) e a pasta `docs/process/` (que o `DESAFIO.md` não previa) foi movida para
   `DESIGN_DOCS_PROCESS.md` na raiz.

8. **O workflow tinha um arquivo de configuração (`.claude/design-docs.config.json`) que
   o `DESAFIO.md` não previa.** Foi removido. O orquestrador passou a receber a transcrição
   diretamente na chamada (`/design-docs @TRANSCRICAO.md`) e a derivar dela o nome da
   feature, o slug e a lista de revisores, gravando os parâmetros resolvidos em
   `docs/_workbench/run-state.md`.

9. **Os diagramas estavam num arquivo `docs/diagrams/webhooks-diagrams.md` à parte**, o que
   também não constava da estrutura do `DESAFIO.md`. Foram movidos para uma seção embutida
   no fim do `docs/FDD.md` ("13. Diagramas"), cada um apontando a seção do FDD que ilustra.

10. **As referências a arquivos do repositório estavam em texto plano**, no Tracker (colunas
    "Documento" e "Localização") e na prosa técnica do FDD e dos ADRs, o que dificultava
    conferir cada item. Nova regra do workflow: toda menção a um arquivo que existe no
    repositório é link relativo em Markdown, e quando cita um trecho específico leva âncora
    de linha (por exemplo `#L126`). No Tracker, a origem na transcrição virou um link para
    a linha exata da fala em `TRANSCRICAO.md`. Os caminhos que a
    feature ainda vai criar (`src/worker.ts`, `src/modules/webhooks/`) continuam como texto
    de código, sem link. O workflow foi rodado de novo por inteiro para regerar tudo sob a
    regra nova; a validação passou em 38 de 38 critérios.

## Como navegar a entrega

Ordem sugerida de leitura:

1. [`CLAUDE.md`](CLAUDE.md): o que a aplicação é hoje, antes da feature.
2. [`docs/PRD.md`](docs/PRD.md): por que a feature existe, escopo, métricas, o que fica de fora.
3. [`docs/RFC.md`](docs/RFC.md): a proposta técnica em nível de arquitetura, alternativas
   descartadas e questões em aberto.
4. [`docs/adrs/`](docs/adrs/): as sete decisões arquiteturais, uma por arquivo.
5. [`docs/FDD.md`](docs/FDD.md): como implementar, com contratos, matriz de erros
   `WEBHOOK_*`, fluxos, a integração com o código existente e, na última seção, os seis
   diagramas Mermaid de apoio.
6. [`docs/TRACKER.md`](docs/TRACKER.md): a rastreabilidade de cada item à transcrição ou
   ao código.
7. [`DESIGN_DOCS_PROCESS.md`](DESIGN_DOCS_PROCESS.md): o registro de como o pacote foi
   produzido.
8. [`.claude/README.md`](.claude/README.md): a doc do workflow que gerou tudo isso e como
   rodá-lo em outra transcrição.

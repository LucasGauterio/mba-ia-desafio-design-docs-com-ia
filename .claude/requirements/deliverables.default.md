# Requirement · perfil de entregáveis (default)

Transposição do enunciado (`DESAFIO.md`, requisitos 1-6 + Critérios de Aceite) para um
perfil que o workflow consome. Uma transcrição/desafio diferente troca este arquivo, não
as skills. `design-docs-spec` gera a partir daqui (ou da spec informada) o
`docs/_workbench/deliverables-checklist.md`.

## Feature-alvo

Sistema de Webhooks de Notificação de Pedidos (outbound), sobre o OMS existente.

## Artefatos

| Artefato | Caminho | Skill |
|---|---|---|
| PRD | `docs/PRD.md` | `design-docs-prd` |
| RFC | `docs/RFC.md` | `design-docs-rfc` |
| FDD | `docs/FDD.md` | `design-docs-fdd` |
| ADRs (5 a 8) | `docs/adrs/ADR-NNN-titulo-kebab.md` | `design-docs-adr` |
| Diagramas | `docs/diagrams/webhooks-diagrams.md` | `design-docs-diagrams` |
| Tracker | `docs/TRACKER.md` | `design-docs-tracker` |
| README do processo | `README.md` (substitui o enunciado) | `design-docs-readme` |

Não alterar `src/`, `prisma/`, `tests/`, configs. `TRANSCRICAO.md` não muda.

## Critérios de aceite (todos obrigatórios)

### PRD `docs/PRD.md`
- [ ] Existe, em Markdown.
- [ ] Contém: Resumo/contexto; Problema e motivação; Público-alvo e cenários; Objetivos e
      métricas; Escopo (incluso e fora); Requisitos funcionais; Requisitos não funcionais;
      Decisões e trade-offs; Dependências; Riscos e mitigação; Critérios de aceitação;
      Estratégia de testes e validação.
- [ ] ≥ 8 requisitos funcionais discutidos na reunião.
- [ ] ≥ 1 objetivo com métrica e meta quantitativa.
- [ ] "Fora de escopo" com ≥ 2 itens explicitamente descartados/adiados na reunião.
- [ ] "Riscos" com ≥ 2 riscos com probabilidade, impacto e mitigação.

### RFC `docs/RFC.md`
- [ ] Existe, em Markdown.
- [ ] Contém: Metadados (autor, status, data, revisores = participantes da reunião);
      TL;DR; Contexto e problema; Proposta técnica; Alternativas consideradas; Questões em
      aberto; Impacto e riscos; Decisões relacionadas (links de ADR).
- [ ] "Alternativas consideradas": ≥ 2 alternativas descartadas na reunião, cada uma com o
      trade-off que motivou o descarte.
- [ ] "Questões em aberto": ≥ 2 pontos adiados/não decididos.
- [ ] Referencia, com link, ≥ 2 ADRs do pacote.
- [ ] 2 a 4 páginas; sem detalhe de implementação do FDD.

### FDD `docs/FDD.md`
- [ ] Existe, em Markdown.
- [ ] Contém: Contexto e motivação técnica; Objetivos técnicos; Escopo e exclusões;
      Fluxos detalhados (outbox, worker, retry, DLQ); Contratos públicos; Matriz de erros
      `WEBHOOK_*`; Estratégias de resiliência; Observabilidade; Dependências e
      compatibilidade; Critérios de aceite técnicos; Riscos e mitigação; **Integração com
      o sistema existente**.
- [ ] "Contratos públicos": ≥ 4 endpoints HTTP com payload de exemplo (request e response)
      e status codes.
- [ ] Matriz de erros usa códigos com prefixo `WEBHOOK_`.
- [ ] "Integração com o sistema existente": ≥ 4 caminhos de arquivo reais do código base.
- [ ] "Observabilidade": cita métricas, logs e tracing.

### ADRs `docs/adrs/ADR-NNN-*.md`
- [ ] Pasta contém entre 5 e 8 arquivos no formato `ADR-NNN-titulo-em-kebab-case.md`.
- [ ] Cada ADR contém: Status, Contexto, Decisão, Alternativas Consideradas (≥ 1),
      Consequências (positivas e negativas com trade-off explícito).
- [ ] O conjunto cobre ≥ 5 das 6 decisões principais (outbox MySQL; retry/backoff/DLQ;
      HMAC-SHA256 secret por endpoint; at-least-once com `X-Event-Id`; worker separado em
      polling; reuso dos padrões do projeto).
- [ ] ≥ 1 ADR referencia explicitamente arquivos/módulos/classes do código base.

### Tracker `docs/TRACKER.md`
- [ ] Existe e segue o formato de tabela (ID, Documento, Tipo, Conteúdo, Fonte, Localização).
- [ ] ≥ 80% dos itens identificáveis dos documentos têm linha correspondente.
- [ ] ≥ 70% das linhas têm Fonte = TRANSCRICAO com timestamp válido `[hh:mm] Nome`.
- [ ] ≥ 5 linhas têm Fonte = CODIGO com caminho de arquivo real.

### README `README.md`
- [ ] Contém: Sobre o desafio; Ferramentas de IA utilizadas; Workflow adotado; Prompts
      customizados (≥ 2 em blocos de código); Iterações e ajustes (≥ 2 concretos); Como
      navegar a entrega.
- [ ] Lista ≥ 1 ferramenta de IA.

### Consistência geral
- [ ] Nenhum requisito/decisão/restrição contradiz a transcrição ou o código.
- [ ] Nenhum arquivo de código mencionado nos documentos é inexistente no repositório.

## Itens que a reunião DESCARTOU ou ADIOU (não podem virar requisito)

- E-mail de alerta ao cliente quando o webhook falha: `[09:37]`-`[09:38]` (fora de escopo, "futuro").
- Rate limiting de envio para o cliente: `[09:38]`-`[09:39]` (observar e decidir depois).
- Dashboard/painel visual para o cliente: `[09:39]`-`[09:40]` (fora de escopo).
- Garantia de ordering global: `[09:12]`-`[09:14]` (só por `order_id` enquanto single-worker).
- Trigger de banco para acordar o worker: `[09:09]` (descartado; polling atende).
- Arquivamento de linhas entregues da outbox: `[09:08]` (fora do escopo desta feature).
- Exactly-once / coordenação dos dois lados: `[09:25]` (at-least-once é a escolha).
- Redis Streams / subir mais infra: `[09:07]` (overengineering para o time).
- Webhook inbound (cliente manda para a gente): `[09:02]`-`[09:03]` (só outbound).

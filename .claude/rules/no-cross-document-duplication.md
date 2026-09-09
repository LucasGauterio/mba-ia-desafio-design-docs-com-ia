# Rule · cada documento na sua altura, sem duplicar

Conteúdo repetido entre documentos de alturas diferentes é erro. Antes de escrever um
parágrafo, confirme que ele pertence àquele documento (ver
`.claude/references/documentation/taxonomy.md`).

| Documento | Escreve | NÃO escreve |
|---|---|---|
| **PRD** | problema, público, escopo, métricas, riscos de negócio | componentes, endpoints, códigos de erro, infra |
| **RFC** | abordagem de arquitetura, alternativas, questões em aberto | payloads, matriz de erros, fluxos passo a passo, código |
| **ADR** | uma decisão, contexto, trade-off, consequências | detalhe operacional isolado (timeout, header, nome de campo) |
| **FDD** | contratos com exemplos, matriz de erros, fluxos, integração | justificativa de produto, reabertura de decisão |

Referência cruzada em vez de cópia: a RFC **linka** os ADRs; o FDD é **construído sobre**
as decisões; o Tracker **aponta** a origem. Nenhum deles transcreve o outro.

Se o mesmo texto cabe em dois documentos, ele está no lugar errado em pelo menos um.

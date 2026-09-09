---
description: Escreve o FDD em docs/FDD.md (com a seção de diagramas embutida)
---

Invoque a skill **design-docs-fdd**. Produz `docs/FDD.md` com as 12 seções, ≥ 4 endpoints
com request/response, matriz de erros `WEBHOOK_*`, fluxos (outbox/worker/retry/DLQ),
observabilidade (métricas+logs+tracing) e a seção obrigatória "Integração com o sistema
existente" com ≥ 4 caminhos de arquivo reais. Ao final aciona `design-docs-diagrams`, que
acrescenta a seção "Diagramas" ao mesmo arquivo.

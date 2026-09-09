# Reference · Tracker de rastreabilidade

Baseado no requisito 5 e nas "Dicas Finais" do enunciado. O tracker é uma **exigência do
desafio**, não um documento padrão de mercado. Serve de defesa contra alucinação: se um
item de PRD/RFC/FDD/ADR não tem linha aqui com origem preenchida, provavelmente foi inventado.

## Formato obrigatório da tabela

```markdown
| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|---|---|---|---|---|---|
| PRD-FR-01 | docs/PRD.md | Requisito Funcional | Cliente cadastra webhook via POST | TRANSCRICAO | [09:31] Marcos |
| ADR-002 | docs/adrs/ADR-002-retry-backoff-dlq.md | Decisão | 5 tentativas, backoff 1m/5m/30m/2h/12h | TRANSCRICAO | [09:17] Diego |
| FDD-INT-01 | docs/FDD.md | Integração | publishWebhookEvent(tx, ...) chamado dentro de changeStatus | CODIGO | src/modules/orders/order.service.ts |
```

### Colunas

- **ID**: identificador único. Prefixo por documento e tipo: `PRD-FR-01`, `PRD-NFR-03`,
  `PRD-RISK-02`, `RFC-ALT-02`, `RFC-OQ-01`, `FDD-CONTRATO-03`, `FDD-ERR-05`, `FDD-INT-02`,
  `ADR-002`. Numeração sequencial dentro do prefixo.
- **Documento**: caminho do arquivo onde o item aparece (`docs/PRD.md`,
  `docs/adrs/ADR-002-....md`).
- **Tipo**: `Requisito Funcional`, `Requisito Não Funcional`, `Decisão`, `Restrição`,
  `Trade-off`, `Alternativa descartada`, `Questão em aberto`, `Contrato`, `Erro`,
  `Integração`, `Risco`, `Métrica`.
- **Conteúdo (resumo)**: uma linha.
- **Fonte**: exatamente `TRANSCRICAO` ou `CODIGO`.
- **Localização**
  - `TRANSCRICAO` → `[hh:mm] Nome` (timestamp real + falante real). Ex.: `[09:24] Diego`.
  - `CODIGO` → caminho de arquivo real. Ex.: `src/modules/orders/order.service.ts`.

## Limiares (o tracker só passa quando)

- [ ] ≥ **80%** dos itens identificáveis nos documentos têm linha correspondente.
- [ ] ≥ **70%** das linhas têm `Fonte = TRANSCRICAO` com timestamp válido no formato `[hh:mm] Nome`.
- [ ] ≥ **5** linhas têm `Fonte = CODIGO` com caminho de arquivo que existe no repositório.
- [ ] Todo timestamp citado existe de fato na `TRANSCRICAO.md`.
- [ ] Todo caminho de arquivo citado existe de fato no repositório.

## Como montar

1. Varrer cada documento pronto na ordem PRD → RFC → FDD → ADRs.
2. Para cada afirmação verificável (requisito, decisão, restrição, contrato, erro,
   integração, risco, métrica, alternativa, questão em aberto), criar uma linha.
3. Preencher `Localização` procurando a fala na transcrição ou o arquivo no código.
4. Se não achar origem: voltar ao documento, corrigir ou remover o item (não inventar
   uma origem para "fechar" a linha).
5. Conferir os limiares acima.

## Nota de qualidade

O valor do tracker está na coluna `Localização`. Uma linha com `Localização` vazia ou
inventada é pior que a ausência da linha: ela mascara uma alucinação.

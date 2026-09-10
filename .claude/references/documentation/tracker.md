# Reference · Tracker de rastreabilidade

Baseado no requisito 5 e nas "Dicas Finais" do enunciado. O tracker é uma **exigência do
desafio**, não um documento padrão de mercado. É o **único** lugar do pacote onde a origem
de cada item é materializada: PRD/RFC/FDD/ADR têm prosa limpa, sem `[hh:mm]`. Serve de
defesa contra alucinação: se um item de PRD/RFC/FDD/ADR não tem linha aqui com origem
preenchida, provavelmente foi inventado.

## Formato obrigatório da tabela

`docs/TRACKER.md` fica em `docs/`. Os links são relativos a partir daí: documento do pacote
= `PRD.md` / `adrs/ADR-....md`; arquivo de código = `../src/...`; transcrição =
`../TRANSCRICAO.md`. Ver `.claude/rules/repo-file-links.md`.

```markdown
| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|---|---|---|---|---|---|
| PRD-FR-01 | [docs/PRD.md](PRD.md) | Requisito Funcional | Cliente cadastra webhook via POST | TRANSCRICAO | [[09:31] Marcos](../TRANSCRICAO.md#L88) |
| ADR-002 | [docs/adrs/ADR-002-worker-separado-em-polling.md](adrs/ADR-002-worker-separado-em-polling.md) | Decisão | Worker separado, polling 2 s | TRANSCRICAO | [[09:10] Larissa](../TRANSCRICAO.md#L120) |
| FDD-INT-01 | [docs/FDD.md](FDD.md) | Integração | publishWebhookEvent(tx, ...) dentro de changeStatus | CODIGO | [src/modules/orders/order.service.ts#L120](../src/modules/orders/order.service.ts#L120) |
```

### Colunas

- **ID**: identificador único. Prefixo por documento e tipo: `PRD-FR-01`, `PRD-NFR-03`,
  `PRD-RISK-02`, `RFC-ALT-02`, `RFC-OQ-01`, `FDD-CONTRATO-03`, `FDD-ERR-05`, `FDD-INT-02`,
  `ADR-002`. Numeração sequencial dentro do prefixo.
- **Documento**: **link relativo** para o arquivo onde o item aparece. Texto visível =
  caminho a partir da raiz (`docs/PRD.md`); href relativo a `docs/` (`PRD.md`,
  `adrs/ADR-002-....md`).
- **Tipo**: `Requisito Funcional`, `Requisito Não Funcional`, `Decisão`, `Restrição`,
  `Trade-off`, `Alternativa descartada`, `Questão em aberto`, `Contrato`, `Erro`,
  `Integração`, `Risco`, `Métrica`.
- **Conteúdo (resumo)**: uma linha.
- **Fonte**: exatamente `TRANSCRICAO` ou `CODIGO`.
- **Localização** (sempre um link relativo clicável)
  - `TRANSCRICAO` → `[[hh:mm] Nome](../TRANSCRICAO.md#Lnn)`. O texto visível mantém o
    formato `[hh:mm] Nome` (timestamp e falante reais); a âncora `#Lnn` é a linha real da
    fala em `TRANSCRICAO.md` (achar com `grep -n`). Os colchetes de `[hh:mm]` são
    balanceados, então o link `[[hh:mm] Nome](...)` é válido em Markdown.
  - `CODIGO` → `[src/caminho/arquivo.ts#Lnn](../src/caminho/arquivo.ts#Lnn)`, apontando a
    linha do símbolo citado. Texto visível = caminho + `#Lnn`.

## Limiares (o tracker só passa quando)

- [ ] ≥ **80%** dos itens identificáveis nos documentos têm linha correspondente.
- [ ] ≥ **70%** das linhas têm `Fonte = TRANSCRICAO` com timestamp válido no formato `[hh:mm] Nome`.
- [ ] ≥ **5** linhas têm `Fonte = CODIGO` com caminho de arquivo que existe no repositório.
- [ ] Todo timestamp citado existe de fato na `TRANSCRICAO.md` (na linha da âncora).
- [ ] Todo caminho de arquivo citado existe de fato no repositório.
- [ ] **Toda** célula `Documento` e **toda** célula `Localização` é um link relativo
      `[texto](caminho)`; nenhuma fica em texto plano. Cada `href` resolve.

## Como montar

1. Varrer cada documento pronto na ordem PRD → RFC → FDD → ADRs.
2. Para cada afirmação verificável (requisito, decisão, restrição, contrato, erro,
   integração, risco, métrica, alternativa, questão em aberto), criar uma linha.
3. Preencher `Localização`:
   - `TRANSCRICAO`: achar a fala com `grep -n "\[hh:mm\]" TRANSCRICAO.md`, pegar o número
     da linha, montar `[[hh:mm] Nome](../TRANSCRICAO.md#Lnn)`.
   - `CODIGO`: abrir o arquivo, achar a linha do símbolo, montar
     `[src/...#Lnn](../src/...#Lnn)`.
4. Preencher `Documento` como link relativo (`[docs/PRD.md](PRD.md)`,
   `[docs/adrs/ADR-....md](adrs/ADR-....md)`).
5. Se não achar origem: voltar ao documento, corrigir ou remover o item (não inventar
   uma origem para "fechar" a linha).
6. Conferir os limiares acima, inclusive: todo `href` de link resolve para um arquivo real.

## Nota de qualidade

O valor do tracker está na coluna `Localização`. Uma linha com `Localização` vazia ou
inventada é pior que a ausência da linha: ela mascara uma alucinação. Um link que não
resolve tem o mesmo efeito: sempre conferir o `href`.

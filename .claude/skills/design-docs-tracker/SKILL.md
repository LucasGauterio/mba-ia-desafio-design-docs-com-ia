---
name: design-docs-tracker
description: >-
  Monta o Tracker de rastreabilidade (docs/TRACKER.md) varrendo PRD, RFC, FDD e
  ADRs e ligando cada item à origem na transcrição ([hh:mm] Nome) ou no código
  (caminho de arquivo). Roda depois que PRD/RFC/FDD/ADR estão prontos.
---

# design-docs-tracker: tracker de rastreabilidade

## Insumos

- `docs/PRD.md`, `docs/RFC.md`, `docs/FDD.md`, `docs/adrs/*`.
- `TRANSCRICAO.md` (para validar timestamps).
- `.claude/references/codebase/*` e o repositório (para validar caminhos).
- `.claude/references/documentation/tracker.md` (formato + limiares + esquema de IDs).

## Passos

1. Varrer os documentos na ordem PRD → RFC → FDD → ADRs.
2. Para cada afirmação verificável (requisito, RNF, decisão, restrição, trade-off,
   alternativa, questão em aberto, contrato, erro, integração, risco, métrica), criar uma
   linha na tabela com ID prefixado (`PRD-FR-01`, `RFC-ALT-02`, `FDD-CONTRATO-03`,
   `FDD-INT-02`, `ADR-002`, ...).
3. Preencher `Fonte` (`TRANSCRICAO` ou `CODIGO`) e `Localização`:
   - `TRANSCRICAO` → `[hh:mm] Nome` real, conferido na transcrição.
   - `CODIGO` → caminho de arquivo real, conferido no repositório.
4. Se um item não tem origem localizável: **voltar ao documento** e corrigir/remover; não
   inventar origem.
5. Conferir os limiares: ≥ 80% de cobertura; ≥ 70% das linhas `TRANSCRICAO` com timestamp
   válido; ≥ 5 linhas `CODIGO` com arquivo real.
6. Atualizar a linha `tracker` em `docs/_workbench/run-state.md`.

## Saída

`docs/TRACKER.md` (tabela no formato obrigatório) + um bloco final "Cobertura" com os
números apurados.

## Checklist antes de concluir

- [ ] Tabela com colunas ID, Documento, Tipo, Conteúdo (resumo), Fonte, Localização.
- [ ] ≥ 80% dos itens dos documentos têm linha.
- [ ] ≥ 70% das linhas: Fonte = TRANSCRICAO com `[hh:mm] Nome` válido.
- [ ] ≥ 5 linhas: Fonte = CODIGO com caminho real.
- [ ] Todo timestamp existe na `TRANSCRICAO.md`; todo caminho existe no repo.

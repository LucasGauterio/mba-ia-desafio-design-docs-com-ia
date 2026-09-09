# Rule · todo item precisa de fonte identificável

Nenhum requisito, decisão, restrição, contrato, erro, métrica, risco ou alternativa entra
em um documento sem origem rastreável a **uma** das duas fontes:

- **Transcrição**: `TRANSCRICAO.md`, referenciada como `[hh:mm] Nome` (timestamp e falante
  reais). Ex.: `[09:17] Diego`.
- **Código**: um caminho de arquivo que existe no repositório. Ex.:
  `src/modules/orders/order.service.ts`.

Se você não consegue apontar a origem de um item:

1. Verifique se ele realmente foi discutido (releia o trecho da transcrição).
2. Se foi um preenchimento razoável de lacuna, marque `(hipótese)` e diga em que se baseia.
3. Caso contrário, **remova o item**. Não invente uma origem para justificá-lo.

O `docs/TRACKER.md` materializa esta regra: cada item com coluna `Localização` vazia ou
inventada é uma alucinação mascarada.

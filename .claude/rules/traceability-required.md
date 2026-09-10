# Rule · rastreabilidade vive no Tracker, não na prosa

Nenhum requisito, decisão, restrição, contrato, erro, métrica, risco ou alternativa entra
num documento sem **origem identificável** em uma das duas fontes:

- **Transcrição**: `TRANSCRICAO.md`, um trecho real (`[hh:mm] Nome`, timestamp e falante reais).
- **Código**: um caminho de arquivo que existe no repositório.

## Onde a origem aparece

- **No `docs/TRACKER.md`**: cada afirmação verificável dos documentos tem **uma linha** na
  tabela, com a coluna `Localização` preenchida (`[hh:mm] Nome` para `TRANSCRICAO`, caminho
  de arquivo para `CODIGO`). Este é o único lugar onde a origem é materializada.
- **Não no corpo do PRD / RFC / FDD / ADR**: a prosa fica limpa. Não use `[hh:mm]`,
  colchetes de timestamp nem "(Fulano, 09:17)" no texto. Quando ajudar a leitura, use
  atribuição natural, sem marcação: "discutido na reunião", "definido pela equipe de
  segurança", "decisão do time".
- **Exceção**: referências a arquivos do código (`src/...`, `prisma/...`) **podem** aparecer
  na prosa, porque são conteúdo técnico de navegação, não citação de fonte. Quando
  aparecem, seguem [`repo-file-links.md`](repo-file-links.md): link relativo, com âncora de
  linha `#Lnn` quando citam um trecho específico. A seção "Integração com o sistema
  existente" do FDD e a seção "Referências" dos ADRs os usam assim.

## Se você não consegue apontar a origem de um item

1. Verifique se ele realmente foi discutido (releia o trecho da transcrição).
2. Se foi um preenchimento razoável de lacuna, marque `(hipótese)` no texto e diga em que se baseia.
3. Caso contrário, **remova o item**. Não invente uma origem para justificá-lo.

## Cobertura

Ao limpar citações da prosa, **não perca cobertura**: cada afirmação que deixa de citar a
origem inline continua precisando da sua linha no `docs/TRACKER.md`. A validação exige
pelo menos 80% dos itens identificáveis com linha correspondente.

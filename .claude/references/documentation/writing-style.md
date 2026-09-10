# Reference · estilo de escrita dos design docs

Regras de estilo para **todos** os documentos do pacote.

## Idioma e forma

- **Português do Brasil**, direto e simples. Frases curtas.
- **Nunca use travessão longo (em-dash)** para pausa ou aposto. Use vírgula, parêntese,
  dois-pontos ou duas frases.
- Termos técnicos e nomes de tecnologia ficam em inglês (`outbox`, `worker`, `backoff`,
  `retry`, `HMAC`, `Prisma`, `MySQL`). Não traduzir.
- Markdown limpo: títulos hierárquicos, listas, tabelas. Blocos de código com a linguagem
  marcada (```json, ```ts, ```mermaid).

## Rastreabilidade fica no Tracker, não na prosa

- **Não cite a origem no corpo.** Sem `[hh:mm]`, sem colchetes de timestamp, sem
  "(Fulano, 09:17)" no texto do PRD, RFC, FDD ou ADR.
- A origem de cada item vive numa linha do `docs/TRACKER.md` (ver
  `.claude/rules/traceability-required.md`).
- Quando ajudar a leitura, use atribuição natural, sem marcação: "discutido na reunião",
  "definido pela equipe de segurança", "decisão do time", "levantado e adiado".
- **Referências a arquivos do código** (`src/...`, `prisma/...`) são conteúdo técnico e
  **podem** aparecer na prosa. Quando aparecem, são **link relativo** com âncora de linha
  `#Lnn` no trecho citado (ver `.claude/rules/repo-file-links.md`), nunca texto plano.

## Conteúdo

- **Nada sem origem.** Todo requisito, decisão ou restrição precisa ter origem na
  transcrição ou no código, materializada no Tracker. Se você não consegue apontar a
  origem, o item foi inventado: remova ou marque como hipótese.
- **Marque hipóteses explicitamente.** Quando algo não veio da transcrição nem do código
  mas é um preenchimento razoável de lacuna, escreva "(hipótese)" e diga em que se baseia.
- **Exemplos concretos, não descrições vagas.** "retorna `429` com header `Retry-After`"
  em vez de "trata o erro adequadamente". Payloads reais, códigos reais, caminhos reais.
- **Sem enchimento.** Corte frases que não adicionam informação verificável.
- **Reformatar sem perder conteúdo**: ao condensar, preserve todos os fatos; encurte a
  prosa, não a informação.

## O que NÃO escrever

- Requisito ou feature que a reunião **descartou** ou **adiou** (ver
  `.claude/rules/honor-rejected-scope.md`).
- Detalhe de um documento repetido em outro de altura diferente (ver `taxonomy.md`).
- Caminho de arquivo que não existe no repositório.
- Número, meta ou SLA que ninguém citou.
- Timestamps ou citações de fonte no corpo do documento.

## Checagem final de cada documento

1. Toda seção obrigatória do template está presente e preenchida.
2. Nenhuma citação de fonte no corpo (`[hh:mm]`, colchetes de timestamp).
3. Toda afirmação verificável tem linha correspondente no `docs/TRACKER.md`.
4. Nenhum item descartado/adiado aparece como requisito.
5. Nenhum caminho de arquivo inexistente.
6. Sem travessão longo (em-dash). Sem enchimento vago.
7. Não duplica o nível de detalhe de outro documento.

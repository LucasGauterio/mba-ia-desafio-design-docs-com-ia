# Reference · estilo de escrita dos design docs

Regras de estilo para **todos** os documentos do pacote.

## Idioma e forma

- **Português do Brasil**, direto e simples. Frases curtas.
- **Nunca use travessão longo (em-dash)** para pausa ou aposto. Use vírgula, parêntese, dois-pontos ou
  duas frases.
- Termos técnicos e nomes de tecnologia ficam em inglês (`outbox`, `worker`, `backoff`,
  `retry`, `HMAC`, `Prisma`, `MySQL`). Não traduzir.
- Markdown limpo: títulos hierárquicos, listas, tabelas. Blocos de código com a linguagem
  marcada (```json, ```ts, ```mermaid).

## Conteúdo

- **Nada sem fonte.** Todo requisito, decisão ou restrição tem origem rastreável: um
  timestamp + nome da transcrição (`[09:17] Diego`) ou um caminho de arquivo real. Se você
  não consegue apontar a origem, o item foi inventado: remova ou marque como hipótese.
- **Marque hipóteses explicitamente.** Quando algo não veio da transcrição nem do código
  mas é um preenchimento razoável de lacuna, escreva "(hipótese)" e diga em que se baseia.
- **Exemplos concretos, não descrições vagas.** "retorna `429` com header `Retry-After`"
  em vez de "trata o erro adequadamente". Payloads reais, códigos reais, caminhos reais.
- **Sem enchimento.** Corte frases que não adicionam informação verificável. Se um
  parágrafo pode ser removido sem perda, remova.
- **Reformatar sem perder conteúdo**: ao condensar uma fonte, preserve todos os
  fatos; encurte a prosa, não a informação.

## O que NÃO escrever

- Requisito ou feature que a reunião **descartou** ou **adiou** (ver
  `.claude/rules/honor-rejected-scope.md`).
- Detalhe de um documento repetido em outro de altura diferente (ver `taxonomy.md`).
- Caminho de arquivo que não existe no repositório.
- Número, meta ou SLA que ninguém citou.

## Checagem final de cada documento

1. Toda seção obrigatória do template está presente e preenchida.
2. Todo item tem origem rastreável (transcrição ou código).
3. Nenhum item descartado/adiado aparece como requisito.
4. Nenhum caminho de arquivo inexistente.
5. Sem travessão longo (em-dash). Sem enchimento vago.
6. Não duplica o nível de detalhe de outro documento.

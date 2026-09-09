# Guideline · a IA como ferramenta, você como maestro

Baseado nas seções "Sobre o uso de IA", "Ordem de execução sugerida" e "Dicas Finais" do enunciado.

## Papel

A IA produz; o humano rege. O trabalho do maestro é: definir o que precisa ser feito,
formular prompts dirigidos, revisar criticamente o que a IA entrega, corrigir e refinar
até ficar consistente. O resultado tem que parecer escrito por alguém que pensou no
problema com a IA ao lado, não por alguém que copiou e colou da transcrição.

## Prompts dirigidos, não genéricos

- Ruim: "gere um PRD a partir dessa transcrição". Produz documento vazio e genérico.
- Bom: prompt que nomeia o documento, a seção, o insumo exato (o ledger da transcrição, a
  reference correspondente, o baseline do código), a contagem mínima e o que **não** incluir.
- Filtragem do que não entra é uma tarefa dirigida: "liste só as decisões FECHADAS entre
  `[09:03]` e `[09:12]`; ignore o que foi levantado e adiado".

## Iterar é esperado

3 a 5 ciclos de geração → revisão crítica → ajuste de prompt → nova geração por documento.
Se saiu certo de primeira, provavelmente está genérico demais. Momentos típicos de correção:
- a IA inventou um requisito sem origem na transcrição;
- a IA trouxe um item que a reunião descartou;
- a IA repetiu no RFC o detalhe que é do FDD;
- a IA citou um arquivo que não existe;
- a IA ficou superficial numa seção (ex.: "trata os erros adequadamente").

## Ordem de produção (do enunciado)

1. Baseline do código (`design-docs-baseline`).
2. Ledger da transcrição + checklist da spec (`design-docs-ledger`, `design-docs-spec`).
3. **ADRs primeiro**: as decisões formam o esqueleto do "como implementar".
4. **RFC**: consolida a proposta em cima das decisões; linka os ADRs.
5. **FDD**: desenho técnico sobre as decisões; seção obrigatória de integração; diagramas.
6. **PRD por último** entre os grandes: vira consolidação de alto nível.
7. **Tracker**: varre os documentos prontos.
8. **README do processo**: quando o processo já está completo.
9. Revisão final contra os critérios de aceite.

## Defesa contra alucinação

O tracker é o principal instrumento. Se você não consegue preencher a coluna "Localização"
para uma linha, aquela informação não tem origem identificável: ajuste ou remova o item no
documento. Não invente uma origem para fechar a linha.

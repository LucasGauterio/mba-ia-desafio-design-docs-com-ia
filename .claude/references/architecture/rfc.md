# Reference · RFC (Request for Comments)

O RFC é menos padronizado que PRD e FDD. Esta estrutura foi **construída** a partir do
requisito 2 do enunciado e do papel clássico de um RFC: circular uma proposta de
arquitetura para revisão antes de fechar a decisão.

## O que é

Proposta técnica da solução, submetida à equipe para revisão. Opera em nível de
**arquitetura**: abordagem escolhida, alternativas que foram colocadas na mesa, questões
deixadas em aberto. É deliberativa: expõe o raciocínio enquanto a decisão ainda está aberta.

## Fronteira

- **Conciso: 2 a 4 páginas.** Fala em decisão, não em implementação.
- **Não duplica o FDD.** Sem payloads de endpoint, sem matriz de erros, sem código. Se
  precisar desse nível, é conteúdo do FDD.
- **Não substitui os ADRs.** Cada decisão fechada tem seu ADR; a RFC linka.

## Estilo

PT-BR, sem travessão longo (em-dash). Ver `writing-style.md`.

## Esqueleto de saída (seguir exatamente)

```markdown
# RFC: [nome da feature]

| | |
|---|---|
| Autor | [nome/papel] |
| Status | Em revisão |
| Data | [AAAA-MM-DD] |
| Revisores | [os participantes da reunião: Larissa (Tech Lead), Marcos (PM), Bruno (Eng.), Diego (Eng. Sênior), Sofia (Segurança)] |

---

## Resumo executivo (TL;DR)
[3 a 6 linhas: o problema, a abordagem proposta em uma frase, e o que ainda está aberto.]

## Contexto e problema
[Por que essa feature agora. O que existe hoje e por que não atende. Restrições reais
(time pequeno, prazo, infraestrutura disponível). Sem repetir o PRD inteiro.]

## Proposta técnica
[Visão geral da solução em nível de arquitetura. Componentes principais e como se
relacionam. Fluxo geral em prosa. NÃO descer ao detalhe de implementação do FDD.]
- [componente / mecanismo 1: papel]
- [componente / mecanismo 2: papel]

## Alternativas consideradas
Pelo menos 2 alternativas reais discutidas e descartadas na reunião, cada uma com o
trade-off que motivou o descarte.

### [Alternativa 1]
- **O que era:** [descrição]
- **Por que foi descartada:** [trade-off concreto que motivou o descarte]

### [Alternativa 2]
- **O que era:** [descrição]
- **Por que foi descartada:** [trade-off]

## Questões em aberto
Pelo menos 2 pontos levantados na reunião e não decididos ou adiados.
- [questão 1: o que ficou de "observar e decidir depois"]
- [questão 2]

## Impacto e riscos
[Impacto em sistemas existentes (resumo, o detalhe é do FDD). Principais riscos técnicos
e operacionais em nível de arquitetura. Prazo/esforço se foi discutido.]

## Decisões relacionadas
Links para os ADRs do pacote.
- [ADR-001: título](adrs/ADR-001-....md)
- [ADR-002: título](adrs/ADR-002-....md)
- ...
```

## Checklist (a RFC só está pronta quando)

- [ ] Metadados completos, com os 5 participantes da reunião como revisores.
- [ ] TL;DR, contexto, proposta técnica, alternativas, questões em aberto, impacto/riscos,
      decisões relacionadas: todas presentes.
- [ ] "Alternativas consideradas" com ≥ 2 alternativas descartadas na reunião, cada uma
      com o trade-off que motivou o descarte.
- [ ] "Questões em aberto" com ≥ 2 pontos adiados/não decididos na reunião.
- [ ] Linka ≥ 2 ADRs do pacote, com link relativo.
- [ ] Sem `[hh:mm]` nem citações de fonte no corpo; toda afirmação verificável tem linha
      no `docs/TRACKER.md`.
- [ ] 2 a 4 páginas. Não contém payloads, códigos de erro ou detalhe de implementação.

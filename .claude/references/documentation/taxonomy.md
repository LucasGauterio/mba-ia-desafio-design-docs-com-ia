# Reference · taxonomia dos design docs

Baseado na tabela "O pacote de documentos e o papel de cada um" do enunciado.
Define **a altura de cada documento** e a **fronteira entre eles**. Regra central:
conteúdo duplicado entre documentos é sinal de que algo está no lugar errado.

## A cadeia de abstração

Problema de produto → solução em alto nível → decisões pontuais → especificação de
implementação → registro de rastreabilidade.

| Documento | Pergunta que responde | Altura | Momento |
|---|---|---|---|
| **PRD** | Por quê e o quê? | Produto / negócio | antes do desenho técnico |
| **RFC** | Como pretendemos resolver, e o que está em aberto? | Arquitetura (proposta) | antes de fechar a decisão, aberto a revisão |
| **ADR** | Por que decidimos exatamente assim? | Decisão pontual | depois que a decisão foi fechada |
| **FDD** | Como construir, em detalhe? | Implementação | com decisões fechadas, antes de codar |
| **Tracker** | De onde veio cada coisa? | Transversal | em paralelo / ao final |

Em uma frase: **o RFC propõe e abre para revisão, os ADRs registram cada decisão fechada,
o FDD detalha como construir.**

## Fronteiras que mais confundem

### RFC × ADR
- **RFC** é deliberativa: circula uma proposta para receber comentários, expõe alternativas
  ainda na mesa e questões não decididas. É concisa (2-4 páginas). Fala em **decisão a tomar**.
- **ADR** é registro: uma decisão já tomada, com contexto e consequências. Uma ADR por
  decisão. Fala em **decisão tomada**.
- A RFC **linka** os ADRs correspondentes; não os repete.

### RFC × FDD
- RFC responde "o que propomos e por quê", em nível de arquitetura (componentes,
  abordagem geral, trade-offs).
- FDD responde "como construir", em nível de implementação (contratos com payloads,
  matriz de erros, fluxos passo a passo, integração com arquivos reais).
- **Não repita no RFC o nível de detalhe do FDD.** Se a RFC está descrevendo payloads de
  endpoint ou códigos de erro, subiu de altura errada.

### ADR × FDD
- Decisão estrutural, debatida, com trade-off e vida longa → ADR.
- Detalhe operacional (valor de timeout, formato de header, nome de campo) → FDD.
  "ADR não é log operacional": timeouts, headers e parâmetros isolados ficam no
  FDD, não viram ADR, a não ser que o parâmetro sustente uma estratégia arquitetural.

### PRD × resto
- PRD não desce a componentes, endpoints ou infraestrutura. Pode citar decisões técnicas
  já dadas com justificativa e trade-off, mas de forma resumida.
- Se o PRD está detalhando implementação, o conteúdo pertence ao FDD/ADR.

## Encadeamento (links entre documentos)

- RFC → lista os ADRs relacionados (com link).
- FDD → construído sobre as decisões dos ADRs e a proposta do RFC.
- Tracker → cruza **todo item** de todos os docs com sua origem.
- README do processo → aponta a ordem de leitura.

## Não faz parte deste pacote

HLD, LLD, C4 completo, engineering guidelines, runbooks. Onde outros fluxos usariam um HLD
como peça de arquitetura, aqui o **RFC** ocupa esse lugar (proposta técnica submetida a
revisão). Diagramas entram como anexo do FDD, não como documento separado.

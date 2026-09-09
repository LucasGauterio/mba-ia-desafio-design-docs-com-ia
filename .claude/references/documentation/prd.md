# Reference · PRD de feature

Estrutura e checklist para o PRD, alinhados ao requisito 1 do enunciado. O PRD opera em
nível de **produto/negócio**: por quê e o quê. Não desce a componentes, endpoints ou infra.

## Quando uma feature merece PRD

Quando deixa de ser "requisito funcional commodity" e concentra decisões de produto: valor
percebido próprio, objetivos mensuráveis, escopo autônomo, trade-offs que precisam ser
explicitados antes do design técnico. Uma feature com clientes esperando, prazo comercial e
integrações externas se qualifica.

## Estilo

PT-BR, direto, sem travessão longo (em-dash). Marcar hipóteses. Ver `writing-style.md`.

## Esqueleto de saída (seguir exatamente)

```markdown
# PRD: [produto]: [feature]

Versão: [versão]
Data: [AAAA-MM-DD]
Responsável: [nome/papel]
Status: [rascunho | em revisão | aprovado]

---

## Resumo

[2 a 4 frases: o que está sendo construído e o problema que resolve. Não é "implementar X",
é "hoje acontece Y, que custa Z, e a feature existe para conter isso".]

---

## Contexto e problema

**Público-alvo**
- [público 1]
- [público 2]

**Cenários de uso**
- [cenário 1]
- [cenário 2]

**Onde a feature roda**
- [sistema existente ou novo; qual]

**Problemas priorizados**
- [problema 1: impacto: prioridade]
- [problema 2: impacto: prioridade]

---

## Objetivos e métricas

| Objetivo | Métrica | Meta |
|---|---|---|
| [objetivo 1] | [métrica] | [meta quantitativa] |
| [objetivo 2] | [métrica] | [meta] |

Pelo menos um objetivo com meta numérica.

---

## Escopo

**Incluso**
- [item]

**Fora de escopo**
- [item explicitamente descartado ou adiado na reunião] (mín. 2, cada um com a situação
  em prosa: "adiado para a próxima fase", "projeto separado do time de frontend")
- [item 2]

---

## Requisitos funcionais

Mínimo de 8, todos discutidos na reunião. Para cada um:

### [FR-NN] [nome curto]
[descrição em uma frase]

**Fluxo principal**
- [passo]

**Fluxos alternativos e exceções**
- [variação / exceção]

**Erros previstos**
- [erro]

**Prioridade:** [alta | média | baixa]

---

## Requisitos não funcionais

Agrupar por categoria, com números quando houver:
- **Performance / latência**: [ex: evento entregue em < N s no caminho feliz]
- **Disponibilidade / confiabilidade**: [ex: garantia at-least-once; nenhum evento perdido se a transação commitou]
- **Segurança**: [ex: assinatura HMAC-SHA256; TLS obrigatório; secret por endpoint]
- **Observabilidade**: [logs estruturados, métricas de entrega/falha, correlação]
- **Limites**: [ex: payload máximo N KB]
- **Compatibilidade**: [ex: reusa padrões de módulo, erro e log já existentes]

---

## Decisões e trade-offs principais

Resumo (o detalhe fica nos ADRs). Para cada uma:

### Decisão: [título]
- **Justificativa:** [por quê]
- **Trade-off:** [o que se aceitou em troca]

---

## Dependências

### [tipo: técnica | organizacional | externa]: [título]
[quem precisa entregar o quê e por quê]

---

## Riscos e mitigação

Mínimo de 2, cada um completo:

### [risco em uma frase]
- **Probabilidade:** [baixa | média | alta]
- **Impacto:** [descrição]
- **Mitigação:**
  - [ação]
- **Plano de contingência:** [plano B]

---

## Critérios de aceitação

Checklist objetivo e verificável de quando a feature está pronta.
- [critério]

---

## Estratégia de testes e validação

**Tipos de teste obrigatórios**
- [unitário para regra crítica X]
- [integração para o fluxo principal]
- [teste de segurança para assinatura/secret]

**Abordagem de validação**
- [ex: TDD para a lógica de retry; revisão de segurança antes do deploy]
```

## Checklist (o PRD só está pronto quando)

- [ ] Todas as seções acima presentes.
- [ ] ≥ 8 requisitos funcionais, todos discutidos na reunião.
- [ ] ≥ 1 objetivo com métrica e meta quantitativa.
- [ ] "Fora de escopo" com ≥ 2 itens explicitamente descartados/adiados, cada um com a
      situação em prosa.
- [ ] "Riscos" com ≥ 2 riscos completos (probabilidade + impacto + mitigação).
- [ ] Sem `[hh:mm]` nem citações de fonte no corpo; toda afirmação verificável tem linha
      no `docs/TRACKER.md`.
- [ ] Nenhum requisito contradiz a transcrição ou o código.
- [ ] Não desce ao detalhe de implementação (isso é do FDD).
- [ ] Produzido por último entre os grandes documentos (consolida ADR + RFC + FDD).

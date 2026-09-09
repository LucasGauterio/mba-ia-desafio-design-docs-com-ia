# Reference · diagramas Mermaid do FDD

Boas práticas de diagramação com Mermaid para o FDD. Diagramas reduzem custo cognitivo.
Entram como seção do próprio FDD, não como documento separado.

## Onde e quantos

- **Embutidos no FDD**, como a última seção numerada de `docs/FDD.md` (título "Diagramas").
  Não há arquivo de diagramas separado.
- Cada diagrama é uma subseção: título, parágrafo descritivo (3 a 5 frases) que aponta a
  seção do FDD que ele ilustra, bloco ` ```mermaid ` e um bloco **Notas:**.
- Tipicamente 4 a 6 diagramas. Máximo 10. Cada diagrama tem que passar no teste de
  significância abaixo, senão não entra.

## Teste de significância (o diagrama entra se responde SIM a pelo menos um)

1. Explica o fluxo principal fim a fim?
2. Esclarece uma parte difícil ou não óbvia (algoritmo com condicional, retry, DLQ,
   concorrência)?
3. Ilustra uma decisão arquitetural (outbox, worker separado, single vs multi-worker)?
4. Mostra um contrato público essencial para integração?
5. Mostra a relação entre entidades/componentes?

Nunca dois diagramas dizendo a mesma coisa.

## Diagramas esperados para a feature de webhooks

1. **Fluxo de criação do evento**: sequência: mudança de status → `changeStatus` dentro
   da `$transaction` → insere linha na outbox (com snapshot do payload) → commit.
2. **Processamento pelo worker**: flowchart: polling 2 s → seleciona pendentes por
   `created_at` → envia HTTP com assinatura → sucesso marca entregue / falha agenda retry.
3. **Retry e DLQ**: flowchart: falha → incrementa tentativa → tentativa < 5? agenda
   próximo backoff (1m/5m/30m/2h/12h) : move para `webhook_dead_letter`.
4. **Máquina de estados do evento na outbox**: stateDiagram: `pendente → processando →
   entregue` / `processando → falhou → pendente` (retry) / `falhou → dead_letter`.
5. **Contratos públicos**: classDiagram das entidades/DTO (`WebhookEndpoint`,
   `WebhookOutboxEvent`, `WebhookDelivery`, `WebhookDeadLetter`) e do payload enviado.
6. **Reprocessamento da DLQ**: sequência: `POST /admin/webhooks/dead-letter/:id/replay`
   (role ADMIN) → recoloca na outbox como pendente → log de auditoria.

## Regras de sintaxe Mermaid (guardrails)

- Texto em PT com acentos corretos; nomes técnicos em inglês (`Worker`, `Outbox`, `Store`,
  `Client`, `Retry`, `DLQ`).
- Labels de nó: no máximo 3 palavras. Detalhe vai nas notas abaixo do diagrama.
- IDs de nó/estado/subgraph em ASCII puro (`Operacao`, `nao`), sem acento/espaço; acento só
  no label.
- Uma instrução Mermaid por linha.
- Quebra de linha dentro de label: `<br/>`, nunca `\n`.
- Sem `min(`, `max(`, `++`, `+=`, `{...}` em labels (quebram o parser). Use "Incrementa
  contador", "Recalcula backoff".
- Subgraph com espaço/acento/parêntese: aspas. `subgraph "Modo worker (polling)"`.
- Sequence usa `->>`, `-->>`, `--x`; flowchart usa `-->`, `-.->`, `-- texto -->`. Não misturar.
- Sem emojis.

## Estrutura da seção (dentro de `docs/FDD.md`)

```markdown
## <N+1>. Diagramas

[1 a 2 frases: diagramas de apoio às seções anteriores; nenhum introduz elemento novo.]

### <N+1>.1 [Título do diagrama 1]
[Parágrafo de 3 a 5 frases: o que representa, qual seção do FDD ilustra, por que é relevante.]

```mermaid
...
```

**Notas:**
- [ponto explicativo]

### <N+1>.2 [Título do diagrama 2]
[repetir por diagrama]
```

`<N>` é o número da última seção numerada que já existe no FDD (hoje, 12).

## Checklist

- [ ] Seção "Diagramas" no fim do `docs/FDD.md`, numerada na sequência das demais.
- [ ] 4 a 10 diagramas, cada um passando no teste de significância.
- [ ] Nenhum elemento inventado (só o que está no FDD).
- [ ] PT com acentos; termos técnicos em inglês; labels curtos.
- [ ] Sintaxe Mermaid válida (guardrails acima).
- [ ] Sem redundância entre diagramas.

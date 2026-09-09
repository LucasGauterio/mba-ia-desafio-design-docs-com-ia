# Reference · FDD (Feature Design Document)

Estrutura e checklist de completude para o FDD, alinhados ao requisito 3 do enunciado.
O FDD é o documento mais técnico do pacote: **como implementar, em detalhe**, acionável o
suficiente para um dev pegar e começar a codar. Não repete a narrativa de negócio do PRD.

## Fronteira

- Detalha contratos, erros, fluxos e integração. **Não** reabre decisões (isso é dos ADRs)
  nem justifica o produto (isso é do PRD).
- Vai fundo em comportamento verificável; não chega a prescrever código linha a linha.

## Estilo

PT-BR, sem travessão longo (em-dash), exemplos concretos. Ver `writing-style.md`.

## Esqueleto de saída (seguir exatamente)

```markdown
# FDD: [nome da feature]

Versão: [versão]
Data: [AAAA-MM-DD]
Responsável: [nome/papel]

---

## 1. Contexto e motivação técnica
[Problema técnico real. Como a feature se encaixa nos sistemas existentes. Atores e limites.]

## 2. Objetivos técnicos
- [objetivo com medida ou invariante, ex: "evento entregue em < 10 s no caminho feliz"]
- [garantia determinística, ex: "se a transação de mudança de status commitou, o evento foi registrado"]

## 3. Escopo e exclusões
**Incluído**
- [item]
**Excluído**
- [item: exclusão explícita importa tanto quanto inclusão]

## 4. Fluxos detalhados
Passo a passo de cada fluxo fim a fim. Para esta feature, no mínimo:
- **Criação do evento na outbox**: onde, dentro de qual transação, com qual payload.
- **Processamento pelo worker**: polling, seleção de pendentes, envio HTTP, marcação.
- **Retry**: o que dispara, progressão do backoff, contador de tentativas.
- **DLQ**: quando um evento é movido, o que se guarda, como se reprocessa.
Diagramas Mermaid entram como `docs/diagrams/<feature>-diagrams.md` (ver `diagrams.md`).

## 5. Contratos públicos
Para cada endpoint HTTP (mínimo 4): rota + método, semântica de status/headers, exemplo de
request e de response.

**[POST /api/v1/...]: [o que faz]**
- Auth: [authenticate | authenticate + requireRole('ADMIN')]
- Status: 201 criado, 400 validação, 404 não encontrado, 409 conflito, ...

Request:
    ```json
    { }
    ```
Response (201):
    ```json
    { }
    ```

Documentar também o payload do webhook enviado ao cliente (headers `X-Event-Id`,
`X-Signature`, `X-Timestamp`, `X-Webhook-Id`, `Content-Type`; corpo JSON) e a semântica de
cada header.

## 6. Matriz de erros previstos
Tabela. Todos os códigos com prefixo `WEBHOOK_`.

| Código | HTTP | Condição | Tratamento |
|---|---|---|---|
| `WEBHOOK_NOT_FOUND` | 404 | endpoint de webhook inexistente | ... |
| `WEBHOOK_INVALID_URL` | 400 | URL não https | rejeita no schema |
| `WEBHOOK_SECRET_REQUIRED` | 400 | ... | ... |
| ... | | | |

## 7. Estratégias de resiliência
Timeouts (valor e o que conta como falha), retries (quantos, progressão), backoff
(sequência exata), fallback / DLQ, idempotência (como o cliente dedupe por `X-Event-Id`),
garantia de entrega (at-least-once) e suas consequências.

## 8. Observabilidade
- **Métricas**: [eventos pendentes, entregues, falhados, em DLQ; latência de entrega; tentativas por evento]
- **Logs**: [eventos estruturados: enfileiramento, tentativa, sucesso, falha, movimento para DLQ; sem vazar secret]
- **Tracing / correlação**: [`X-Event-Id` e `X-Request-Id` como correlacionadores]

## 9. Dependências e compatibilidade
Versões mínimas (Node, Prisma, MySQL conforme o repo), impacto em interfaces existentes,
garantias de compatibilidade (o CRUD atual não muda; `changeStatus` ganha um passo).

## 10. Critérios de aceite técnicos
Checklist objetivo: contratos respondendo, matriz de erros coberta por teste, worker
processando dentro do SLA, retry/DLQ exercitados, revisão de segurança da assinatura feita.

## 11. Riscos e mitigação
Riscos técnicos com probabilidade, impacto, mitigação (subitens), contingência.

## 12. Integração com o sistema existente  (SEÇÃO OBRIGATÓRIA DESTE DESAFIO)
Nomear **pelo menos 4 caminhos de arquivo reais** do código base e descrever como o módulo
de webhooks se integra a cada um. Use `.claude/references/codebase/integration-points.md`.
Cobrir no mínimo:
- `src/modules/orders/order.service.ts`: como `changeStatus` é estendido (função
  `publishWebhookEvent(tx, order, fromStatus, toStatus)` chamada dentro da `$transaction`).
- `src/shared/errors/*`: como as classes de erro são reutilizadas / estendidas (`WEBHOOK_*`).
- `src/middlewares/auth.middleware.ts`: `authenticate` nas rotas, `requireRole('ADMIN')` no replay.
- `src/config/database.ts` / novo entrypoint: worker abre `PrismaClient` próprio.
- (e.g.) `src/modules/orders/order.status.ts`, `src/shared/logger/index.ts`,
  `src/middlewares/validate.middleware.ts`, `src/app.ts` (`buildControllers`/`buildApiRouter`).
```

## Checklist (o FDD só está pronto quando)

- [ ] Todas as 12 seções presentes.
- [ ] ≥ 4 endpoints HTTP, cada um com exemplo de request e response e status codes.
- [ ] Matriz de erros usa exclusivamente códigos `WEBHOOK_*`.
- [ ] Fluxos cobrem outbox, worker, retry e DLQ.
- [ ] Observabilidade cita métricas, logs **e** tracing/correlação.
- [ ] Seção "Integração com o sistema existente" nomeia ≥ 4 caminhos de arquivo **que
      existem** no repositório.
- [ ] Nenhuma decisão é reaberta; nenhum item de negócio é repetido do PRD.
- [ ] Nenhum item descartado/adiado aparece.

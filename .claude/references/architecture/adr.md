# Reference · ADR (Architecture Decision Record): formato MADR

Guia condensado do formato MADR (Markdown Any Decision Records). Uma ADR registra **uma
decisão arquitetural fechada**, com contexto e consequências.

## Quando algo vira ADR (regra dos 3 Es)

- **Estrutural**: afeta como o sistema é construído ou integrado, cruza fronteiras de módulo.
- **Evidente**: outra pessoa vai precisar entender o "porquê" no futuro.
- **Estável**: expectativa de durar meses ou anos, não semanas.

Se falta um dos três, não vira ADR. **ADR não é log operacional**: valores de timeout,
formato de header, nomes de campo isolados ficam no FDD, não viram ADR, a menos que o
parâmetro sustente uma estratégia arquitetural.

## Uma ADR por decisão

Não juntar várias decisões num arquivo. Não catalogar cada regra de negócio. Decisão
recorrente vira um ADR e referências.

## Nomeação

`docs/adrs/ADR-NNN-titulo-em-kebab-case.md`, numeração sequencial.
Ex.: `ADR-001-outbox-no-mysql.md`, `ADR-002-retry-backoff-dlq.md`.

## Formato (MADR, 7 seções, header enxuto)

```markdown
# ADR-NNN: [título específico da decisão]

**Status:** Aceito
**Data:** AAAA-MM-DD
**Decisões relacionadas:** ADR-XXX, ADR-YYY   (opcional; só quando houver relação técnica real)

## Contexto e problema
[O problema, a motivação, as restrições e as forças em jogo no momento da decisão.
2 a 3 parágrafos. Mostra em que cenário concreto a decisão fazia sentido. Prosa limpa,
sem citar timestamps; a origem vai para o Tracker.]

## Decisão
[Qual caminho foi adotado. Direto. 1 a 2 parágrafos.]

## Alternativas consideradas
[Pelo menos 1 alternativa real, discutida na reunião ou plausível. Para cada uma: o que
era e por que não foi escolhida.]
- **[Alternativa]:** [descrição]. Descartada porque [trade-off].

## Consequências
[Efeitos da decisão, positivos e negativos, com trade-off explícito.
- Positivas: [ganhos]
- Negativas / limitações aceitas: [custos]
- Impacto operacional / restrições futuras.]

## Referências
[3 a 5 itens. Caminhos de arquivo (`src/modules/orders/order.service.ts`), links para
o RFC e outros ADRs. SEM trechos de código, SEM timestamps.]
```

Proibido no ADR: campos de header além de Status / Data / Decisões relacionadas; seções
extras (Validação, Mais informações, Considerações futuras); trechos de código; citações
de fonte no corpo (`[hh:mm]`); mais de 5 referências de arquivo; sugestões de trabalho
futuro ("considerar X se..."). Alvo: 100 a 250 linhas.

## O conjunto de ADRs deste desafio

Cobrir no mínimo 5 das 6 decisões principais da reunião (podem virar 5 a 8 arquivos). Os
timestamps de cada uma estão no `docs/_workbench/transcript-ledger.md`; use o ledger para
localizar o trecho e o Tracker para materializar a origem.

1. **Padrão Outbox no MySQL** (contra Redis Streams e contra disparo síncrono).
2. **Retry com backoff exponencial mais DLQ** (5 tentativas, 1m/5m/30m/2h/12h; tabela
   `webhook_dead_letter` separada; replay manual por endpoint admin).
3. **Autenticação HMAC-SHA256 com secret por endpoint** (rotação com grace period de 24h).
4. **Garantia at-least-once com `X-Event-Id`** (dedupe do lado do cliente).
5. **Worker em processo separado, polling de 2 s** (sem trigger de banco; single-worker,
   ordenação por `order_id`).
6. **Reuso dos padrões existentes** (`AppError`, Pino, error middleware, módulo em
   `src/modules/webhooks`, prefixo `WEBHOOK_`, UUID como id).

Decisões secundárias que **podem** virar ADR adicional ou ficar só no FDD: snapshot do
payload na inserção, formato do payload, timeout de 10 s, conjunto de headers.

**Pelo menos 1 ADR** deve referenciar explicitamente arquivos/módulos/classes do código
existente (o ADR de reuso é o candidato natural: cita `src/shared/errors/`,
`src/shared/logger/`, `src/modules/orders/order.service.ts`, `src/app.ts`).

## Checklist (cada ADR só está pronto quando)

- [ ] Header só com Status, Data e (se houver) Decisões relacionadas.
- [ ] As 7 seções presentes: Contexto e problema, Decisão, Alternativas consideradas
      (mínimo 1), Consequências (positivas **e** negativas com trade-off), Referências.
- [ ] Sem trechos de código; até 5 referências de arquivo.
- [ ] **Sem citações de fonte no corpo** (`[hh:mm]`, colchetes de timestamp).
- [ ] Toda afirmação verificável do ADR tem linha correspondente no `docs/TRACKER.md`.
- [ ] O conjunto cobre pelo menos 5 das 6 decisões principais; pelo menos 1 ADR referencia
      o código real.

# Reference · ADR (Architecture Decision Record): formato MADR

Guia condensado do formato MADR (Markdown Any Decision Records). Uma ADR registra **uma
decisão arquitetural fechada**, com contexto e consequências.

## Quando algo vira ADR (regra dos 3 Es)

- **Estrutural**: afeta como o sistema é construído ou integrado, cruza fronteiras de módulo.
- **Evidente**: outra pessoa vai precisar entender o "porquê" no futuro.
- **Estável**: expectativa de durar meses ou anos, não semanas.

Se falta um dos três, não vira ADR. **ADR não é log operacional**: valores de
timeout, formato de header, nomes de campo isolados ficam no FDD, não viram ADR, a menos
que o parâmetro sustente uma estratégia arquitetural.

## Uma ADR por decisão

Não juntar várias decisões num arquivo. Não catalogar cada regra de negócio. Decisão
recorrente → um ADR e referências.

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
2 a 3 parágrafos. Mostra em que cenário concreto a decisão fazia sentido. Cita a origem
na transcrição.]

## Decisão
[Qual caminho foi adotado. Direto. 1 a 2 parágrafos.]

## Alternativas consideradas
[Pelo menos 1 alternativa real, discutida na reunião ou plausível. Para cada uma: o que
era e por que não foi escolhida.]
- **[Alternativa]:** [descrição]: descartada porque [trade-off].

## Consequências
[Efeitos da decisão, positivos e negativos, com trade-off explícito.
- Positivas: [ganhos]
- Negativas / limitações aceitas: [custos]
- Impacto operacional / restrições futuras.]

## Referências
[3 a 5 itens. Caminhos de arquivo (`src/modules/orders/order.service.ts`), links para
o RFC e outros ADRs. SEM trechos de código.]
```

Proibido no ADR: campos de header além de Status / Data / Decisões relacionadas; seções
extras (Validação, Mais informações, Considerações futuras); trechos de código; mais de 5
referências de arquivo; sugestões de trabalho futuro ("considerar X se..."). Alvo: 100-250
linhas.

## O conjunto de ADRs deste desafio

Cobrir no mínimo 5 das 6 decisões principais da reunião (podem virar 5-8 arquivos):

1. **Padrão Outbox no MySQL** (vs Redis Streams / síncrono): `[09:06]`-`[09:08]` Diego, `[09:07]` Larissa.
2. **Retry com backoff exponencial + DLQ** (5 tentativas, 1m/5m/30m/2h/12h; tabela
   `webhook_dead_letter` separada): `[09:15]`-`[09:18]` Diego.
3. **Autenticação HMAC-SHA256 com secret por endpoint** (rotação, grace period 24h):   `[09:20]`-`[09:22]` Sofia.
4. **Garantia at-least-once com `X-Event-Id`** (dedupe do lado do cliente): `[09:24]`-`[09:26]` Diego.
5. **Worker em processo separado, polling de 2 s** (sem trigger de banco; single-worker,
   ordering por `order_id`): `[09:09]`-`[09:12]` Diego / Larissa.
6. **Reuso dos padrões existentes** (`AppError`, Pino, error middleware, módulo em
   `src/modules/webhooks`, prefixo `WEBHOOK_`): `[09:27]`-`[09:30]` Bruno / Larissa.

Decisões secundárias que **podem** virar ADR adicional ou ficar só no FDD: snapshot do
payload na inserção (`[09:51]`-`[09:52]`), UUID como id da outbox (`[09:51]`), formato do
payload, timeout de 10 s, conjunto de headers.

**Pelo menos 1 ADR** deve referenciar explicitamente arquivos/módulos/classes do código
existente (o ADR 6 é o candidato natural: cita `src/shared/errors/`, `src/shared/logger/`,
`src/modules/orders/order.service.ts`, `src/app.ts`).

## Checklist (cada ADR só está pronto quando)

- [ ] Header só com Status, Data e (se houver) Decisões relacionadas.
- [ ] As 7 seções presentes: Contexto e problema, Decisão, Alternativas consideradas
      (≥ 1), Consequências (positivas **e** negativas com trade-off), Referências.
- [ ] Sem trechos de código; ≤ 5 referências de arquivo.
- [ ] Contexto cita a origem na transcrição.
- [ ] O conjunto cobre ≥ 5 das 6 decisões principais; ≥ 1 ADR referencia o código real.

---
name: design-docs-rfc
description: >-
  Escreve o RFC (Request for Comments) da proposta técnica da feature, em nível
  de arquitetura, submetido à equipe para revisão. Produz docs/RFC.md (2 a 4
  páginas). Roda depois de design-docs-adr; linka os ADRs já escritos.
---

# design-docs-rfc: RFC da proposta técnica

## Insumos

- `docs/adrs/*` (as decisões já fechadas: a RFC linka, não repete).
- `docs/_workbench/transcript-ledger.md` → "Descartados" (viram alternativas) e
  "Adiados / futuro" (viram questões em aberto).
- `.claude/references/architecture/rfc.md` (esqueleto e checklist).
- `.claude/design-docs.config.json` → `reviewers`, `feature`.
- `.claude/rules/*`.

## Passos

1. Metadados: autor, status "Em revisão", data, revisores = os 5 participantes da reunião.
2. TL;DR de 3 a 6 linhas.
3. Contexto e problema: por que agora, o que existe hoje, restrições reais. Sem repetir o PRD.
4. Proposta técnica: visão de arquitetura (outbox + worker separado + assinatura HMAC +
   at-least-once). Prosa e bullets de componente. **Sem** payloads, códigos de erro ou
   fluxo passo a passo (isso é do FDD).
5. Alternativas consideradas: ≥ 2 do ledger ("Descartados"), cada uma com o trade-off e a
   fala de origem (ex.: disparo síncrono no `changeStatus`; Redis Streams).
6. Questões em aberto: ≥ 2 do ledger ("Adiados"): ex.: rate limiting de saída; escalar
   para múltiplos workers.
7. Impacto e riscos: resumo do impacto nos sistemas existentes + riscos de arquitetura.
8. Decisões relacionadas: link relativo para ≥ 2 ADRs (idealmente todos).
9. Manter em 2 a 4 páginas. Atualizar a linha `rfc` em `docs/_workbench/run-state.md`.

## Saída

`docs/RFC.md`.

## Checklist antes de concluir

- [ ] Metadados com os 5 revisores.
- [ ] TL;DR, contexto, proposta, alternativas, questões em aberto, impacto/riscos, decisões
      relacionadas.
- [ ] ≥ 2 alternativas descartadas com trade-off e origem.
- [ ] ≥ 2 questões em aberto adiadas na reunião.
- [ ] ≥ 2 links de ADR (relativos, válidos).
- [ ] 2 a 4 páginas; nenhum detalhe de implementação do FDD.

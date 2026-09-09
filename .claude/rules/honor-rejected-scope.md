# Rule · respeitar o que a reunião descartou ou adiou

Identificar o que **não** entra é tão importante quanto o que entra. Os itens abaixo foram
explicitamente descartados ou adiados na reunião. **Nenhum deles pode aparecer como
requisito, decisão ou contrato** nos documentos. Eles podem, e devem, aparecer em:

- PRD → seção "Fora de escopo" (com a fala de origem).
- RFC → seção "Questões em aberto" (para os que ficaram de "observar e decidir depois").

| Item | Situação | Origem |
|---|---|---|
| E-mail de alerta ao cliente quando o webhook falha | Fora de escopo desta fase ("futuro") | `[09:37]`-`[09:38]` Marcos / Larissa |
| Rate limiting de envio para o cliente | Observar e decidir depois | `[09:38]`-`[09:39]` Diego / Larissa |
| Dashboard / painel visual para o cliente | Fora de escopo (projeto do time de frontend) | `[09:39]`-`[09:40]` Marcos / Larissa |
| Garantia de ordering global | Não garantida; só por `order_id` enquanto single-worker | `[09:12]`-`[09:14]` Diego / Larissa |
| Trigger de banco para acordar o worker | Descartado; polling de 2 s atende | `[09:09]` Diego |
| Arquivamento de linhas entregues da outbox | Fora do escopo desta feature | `[09:08]` Diego |
| Exactly-once / coordenação dos dois lados | Descartado; escolha é at-least-once | `[09:25]` Diego |
| Redis Streams / subir mais infraestrutura | Overengineering para o time | `[09:07]` Diego / Larissa |
| Webhook inbound (cliente envia para a plataforma) | Só outbound | `[09:02]`-`[09:03]` Marcos / Sofia |
| Escalar para múltiplos workers | Problema do futuro; particionar por `order_id` depois | `[09:13]` Diego |

Se um desses aparecer como requisito num documento, a filtragem falhou: remova.

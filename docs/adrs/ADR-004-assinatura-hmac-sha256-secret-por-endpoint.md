# ADR-004: Assinatura HMAC-SHA256 com secret única por endpoint

**Status:** Aceito
**Data:** 2026-09-09
**Decisões relacionadas:** ADR-005

## Contexto e problema

A plataforma vai enviar eventos com dados de pedidos para endpoints HTTP que estão fora da
sua infraestrutura. O cliente que recebe precisa conseguir validar duas coisas: que a
requisição veio realmente da plataforma, e que ninguém adulterou o payload no caminho.

Já houve um caso de cliente que vazou uma secret em log de aplicação, o que torna o raio de
exposição de uma secret um fator de decisão.

## Decisão

Cada envio é assinado com **HMAC-SHA256 sobre o corpo do request**, e a assinatura vai no
header `X-Signature`. O cliente recalcula e compara do lado dele. HMAC-SHA256 foi escolhido
por ser o padrão de mercado, com biblioteca disponível em qualquer stack séria.

A secret é **única por endpoint de webhook**, nunca uma secret global da plataforma: se uma
vaza, só aquele endpoint é afetado. A tabela de configuração do webhook armazena `url`,
`secret`, `customer_id` e o estado ativo/inativo.

A secret é **rotacionável** por API. Ao rotacionar, a secret antiga continua **válida por
24 horas em paralelo** com a nova (grace period), para o cliente ter tempo de migrar seus
sistemas; depois disso a antiga é invalidada.

O uso de HTTPS na URL do webhook é obrigatório, mas isso é tratado como validação de
schema, não como decisão de arquitetura.

## Alternativas consideradas

- **Uma secret global da plataforma, compartilhada por todos os endpoints.** Descartada: o
  vazamento de uma única secret comprometeria a verificação de todos os webhooks de todos
  os clientes.
- **Rotação sem grace period (troca imediata).** Descartada: sem a janela de 24 horas, o
  cliente teria uma janela de downtime de verificação enquanto atualiza seus sistemas; o
  caso do cliente que vazou secret em log mostra que a rotação precisa ser uma operação
  segura e sem urgência.

## Consequências

Positivas:
- O cliente tem garantia de autenticidade e integridade do payload com um mecanismo padrão.
- O raio de exposição de uma secret vazada fica limitado a um endpoint.
- A rotação com grace period permite trocar secrets sem downtime de verificação.

Negativas e limitações aceitas:
- A plataforma precisa armazenar e proteger uma secret por endpoint, e suportar duas
  secrets válidas simultaneamente durante o grace period.
- A geração e o armazenamento de secrets são superfície de segurança sensível e precisam
  de revisão dedicada antes do deploy.
- O header `X-Timestamp` é enviado junto para o cliente poder detectar replay attack se
  quiser, o que adiciona um campo ao contrato.

## Referências

- `src/middlewares/validate.middleware.ts` : validação Zod da `url` (https obrigatório) e
  dos demais campos.
- `prisma/schema.prisma` : modelo de configuração de webhook (`url`, `secret`,
  `customer_id`, ativo).
- `docs/FDD.md` : semântica dos headers `X-Signature` e `X-Timestamp` e a matriz de erros.

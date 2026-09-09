# PRD: Order Management System, Sistema de Webhooks de Notificação de Pedidos

Versão: 1.0
Data: 2026-09-09
Responsável: Marcos (Product Manager)
Status: em revisão

Consolida o contexto de produto da feature. As decisões técnicas estão nos
[ADRs](adrs/), a proposta de arquitetura no [RFC](RFC.md) e o detalhamento de
implementação no [FDD](FDD.md). Este PRD não desce a componentes, endpoints ou
infraestrutura.

---

## Resumo

Hoje, os clientes B2B que integram com o Order Management System descobrem que o status de
um pedido mudou fazendo requisições repetidas ao `GET /orders`. Isso torna a integração
deles lenta e cara, e não dá visibilidade em tempo hábil. A feature entrega essas mudanças
de status por webhook: quando um pedido muda de status, a plataforma envia uma notificação
HTTP assinada para os endpoints que o cliente cadastrou, com garantia de que nenhuma
mudança confirmada deixa de gerar notificação.

## Contexto e problema

### Público-alvo

- Clientes B2B que integram com a API do OMS. Três pediram a feature formalmente: Atlas
  Comercial, MaxDistribuição e Nova Cargo ([09:00] Marcos).
- Times de integração desses clientes, que hoje mantêm rotinas de polling.
- Administradores da plataforma, que precisam reprocessar entregas que falharam em
  definitivo ([09:35] Larissa).

### Cenários de uso

- Um cliente quer ser avisado assim que um pedido dele é marcado como `SHIPPED` ou
  `DELIVERED`, sem precisar consultar a API ([09:33] Marcos).
- Um cliente cadastra mais de um endpoint (por exemplo, um para logística e outro para
  faturamento) e filtra os status que cada um recebe ([09:33] Bruno; [09:44] Sofia).
- O endpoint de um cliente fica indisponível por algumas horas durante uma manutenção; ao
  voltar, ele ainda recebe as notificações do período ([09:16] Diego).
- Um administrador identifica que uma notificação falhou todas as tentativas e a
  reenfileira manualmente ([09:18] Diego).

### Onde a feature roda

No próprio Order Management System, que já está em produção. Adiciona um módulo à API
existente e um processo de entrega separado ([09:11] Larissa). Nenhuma infraestrutura nova
([09:07] Diego).

### Problemas priorizados

- **Integração lenta e cara para o cliente.** O polling constante no `GET /orders` consome
  recursos dos dois lados e atrasa a reação do cliente à mudança de status. Prioridade
  alta: a Atlas indicou que pode migrar para um concorrente se a feature não sair até o
  fim do trimestre ([09:00] Marcos).
- **Falta de notificação em tempo hábil.** Os clientes consideram "tempo real" qualquer
  latência abaixo de 10 segundos; o polling não garante isso ([09:02] Marcos). Prioridade
  alta.
- **Risco de perder eventos.** Qualquer solução tem que garantir que uma mudança de status
  confirmada sempre gera notificação, mesmo que a entrega demore ([09:41] Diego).
  Prioridade alta.

## Objetivos e métricas

| Objetivo | Métrica | Meta |
|---|---|---|
| Entregar a notificação em tempo hábil | Latência entre o commit da mudança de status e a primeira tentativa de entrega, no caminho feliz | Abaixo de 10 segundos (pior caso da fila: 2 segundos de polling mais o tempo de envio) |
| Não perder eventos | Proporção de mudanças de status confirmadas que geram ao menos uma notificação registrada | 100 por cento |
| Tolerar indisponibilidade do cliente | Janela total de retry antes de considerar falha permanente | 5 tentativas ao longo de aproximadamente 15 horas (1min, 5min, 30min, 2h, 12h) |
| Substituir o polling | Clientes que migraram do polling para webhook | Os 3 clientes solicitantes até a entrega |
| Cumprir o prazo comercial | Data de disponibilização | Fim de novembro, dentro de 3 sprints com a revisão de segurança incluída |

## Escopo

### Incluso

- Cadastro, edição, remoção e listagem de endpoints de webhook por cliente, via API
  autenticada.
- Filtro, por endpoint, de quais status de pedido geram notificação.
- Geração de uma secret por endpoint, entregue na criação, com rotação e período de
  convivência de 24 horas.
- Assinatura de cada envio e identificador único por evento para o cliente deduplicar.
- Entrega assíncrona com retry, backoff e fila de eventos que falharam em definitivo.
- Consulta, pelo cliente, do histórico das últimas entregas de um endpoint.
- Reprocessamento manual, por administrador, de um item da fila de falhas.
- Documentação de integração no portal de desenvolvedor ([09:26] Marcos; [09:40] Marcos).

### Fora de escopo

- **E-mail ou qualquer alerta ao cliente quando o webhook dele falha repetidamente.**
  Adiado para uma fase futura, depois de medir o impacto ([09:37] Marcos; [09:37]
  Larissa).
- **Dashboard ou painel visual** para o cliente acompanhar os webhooks. É um projeto
  separado do time de frontend ([09:39] Marcos; [09:40] Larissa).
- **Garantia de ordem global** entre notificações de pedidos diferentes. A ordem é
  garantida apenas por pedido; os clientes nunca pediram ordem global ([09:12] Diego;
  [09:14] Marcos).
- **Rate limiting das notificações enviadas a um cliente.** Fica como ponto a observar e
  decidir depois, se o volume de chamadas simultâneas virar problema ([09:38] Diego;
  [09:39] Larissa).
- **Arquivamento das notificações já entregues.** Fora do escopo desta feature ([09:08]
  Diego).
- **Webhooks de entrada** (o cliente enviando dados para a plataforma). Só saída ([09:02]
  Marcos; [09:03] Sofia).
- **Processar entrega da fila de falhas de forma automática** e escalar para múltiplos
  processos de entrega. Problema do futuro ([09:13] Diego).

## Requisitos funcionais

### FR-01 Cadastrar endpoint de webhook

O cliente cadastra um endpoint informando a URL de destino, o cliente ao qual pertence e a
lista de status que quer receber. A plataforma gera a secret e a devolve na resposta.

**Fluxo principal**
- O usuário autenticado envia URL, identificador do cliente e lista de status.
- A plataforma valida os dados, gera a secret, cria o endpoint como ativo e retorna o
  endpoint com a secret.

**Fluxos alternativos e exceções**
- URL que não usa HTTPS é recusada.
- Lista de status vazia é recusada.
- Cliente inexistente é recusado.
- URL já cadastrada como ativa para o mesmo cliente é recusada.

**Erros previstos**
- Dados inválidos (URL, lista de status, cliente).
- Conflito de URL duplicada.

**Prioridade:** alta

### FR-02 Editar endpoint de webhook

O cliente altera a URL, a lista de status de interesse ou o estado ativo de um endpoint.

**Fluxo principal**
- O usuário envia os campos a alterar.
- A plataforma valida e persiste.

**Fluxos alternativos e exceções**
- Endpoint inexistente é recusado.
- As mesmas validações de URL e lista de status do cadastro se aplicam.

**Erros previstos**
- Endpoint não encontrado.
- Dados inválidos.

**Prioridade:** alta

### FR-03 Remover endpoint de webhook

O cliente remove um endpoint que não quer mais.

**Fluxo principal**
- O usuário solicita a remoção pelo identificador do endpoint.
- A plataforma remove o endpoint e para de gerar notificações para ele.

**Fluxos alternativos e exceções**
- Endpoint inexistente é recusado.

**Erros previstos**
- Endpoint não encontrado.

**Prioridade:** média

### FR-04 Listar endpoints de um cliente

O cliente lista os endpoints cadastrados para um cliente, sem expor a secret.

**Fluxo principal**
- O usuário informa o identificador do cliente.
- A plataforma retorna os endpoints com URL, status de interesse e estado ativo.

**Fluxos alternativos e exceções**
- Identificador de cliente ausente ou inválido é recusado.

**Erros previstos**
- Dados inválidos.

**Prioridade:** média

### FR-05 Filtrar os status que cada endpoint recebe

Cada endpoint declara a lista de status de pedido que quer ouvir. Uma mudança de status só
gera notificação para os endpoints que incluem aquele status.

**Fluxo principal**
- No cadastro ou edição, o cliente informa a lista de status.
- Quando um pedido muda de status, a plataforma só considera os endpoints do cliente
  daquele pedido cuja lista contém o novo status.

**Fluxos alternativos e exceções**
- Se nenhum endpoint do cliente quer aquele status, nenhuma notificação é gerada ([09:34]
  Bruno).

**Erros previstos**
- Status fora dos valores válidos do ciclo de vida do pedido.

**Prioridade:** alta

### FR-06 Rotacionar a secret de um endpoint

O cliente solicita uma nova secret. A anterior continua válida por 24 horas para o cliente
migrar seus sistemas.

**Fluxo principal**
- O usuário solicita a rotação para um endpoint.
- A plataforma gera a nova secret, mantém a anterior válida por 24 horas e retorna a nova
  secret e a data em que a anterior expira.

**Fluxos alternativos e exceções**
- Endpoint inativo não pode ter a secret rotacionada.
- Endpoint inexistente é recusado.

**Erros previstos**
- Endpoint não encontrado.
- Endpoint inativo.

**Prioridade:** alta

### FR-07 Entregar a notificação ao endpoint do cliente

Quando um pedido muda de status, a plataforma envia uma requisição HTTP assinada para cada
endpoint interessado, de forma assíncrona.

**Fluxo principal**
- A mudança de status registra o evento.
- O processo de entrega envia a notificação para a URL do endpoint com a assinatura e os
  identificadores do evento.
- Resposta de sucesso do cliente encerra a entrega daquele evento.

**Fluxos alternativos e exceções**
- Sem resposta em 10 segundos, a entrega é considerada falha e entra em retry.
- Resposta de erro do cliente é tratada como falha.

**Erros previstos**
- Timeout de entrega.
- Erro de conexão ou resposta de erro do cliente.

**Prioridade:** alta

### FR-08 Retentar entregas que falharam, com backoff e fila de falhas

Uma entrega que falha é retentada até 5 vezes, com intervalos crescentes. Esgotadas as
tentativas, o evento vai para uma fila de falhas permanentes.

**Fluxo principal**
- Após uma falha, a plataforma agenda a próxima tentativa segundo a progressão 1 minuto, 5
  minutos, 30 minutos, 2 horas, 12 horas.
- Na 5ª falha, o evento é movido para a fila de falhas, com o motivo registrado.

**Fluxos alternativos e exceções**
- O cliente volta a responder no meio da janela: a entrega tem sucesso e o retry para.

**Erros previstos**
- Falha permanente após 5 tentativas.

**Prioridade:** alta

### FR-09 Garantir at-least-once com identificador de evento

Cada notificação carrega um identificador único de evento. O cliente pode receber a mesma
notificação mais de uma vez e deve deduplicar por esse identificador.

**Fluxo principal**
- A plataforma gera um identificador único quando o evento é registrado.
- Esse identificador acompanha todas as tentativas de entrega daquele evento.

**Fluxos alternativos e exceções**
- Reenvio após instabilidade: o cliente reconhece a duplicata pelo identificador.

**Erros previstos**
- Nenhum do lado da plataforma; a deduplicação é responsabilidade do cliente ([09:25]
  Sofia).

**Prioridade:** alta

### FR-10 Consultar o histórico de entregas de um endpoint

O cliente consulta as últimas entregas de um endpoint, com resultado, corpo da resposta e
tempo de resposta.

**Fluxo principal**
- O usuário solicita o histórico de um endpoint.
- A plataforma retorna as últimas entregas (por volta de 100), cada uma com sucesso ou
  falha, número da tentativa, resposta e duração ([09:34] Marcos).

**Fluxos alternativos e exceções**
- Endpoint inexistente é recusado.

**Erros previstos**
- Endpoint não encontrado.

**Prioridade:** média

### FR-11 Reprocessar manualmente um item da fila de falhas

Um administrador reenfileira um item que falhou em definitivo. A ação é registrada com o
autor.

**Fluxo principal**
- O administrador solicita o replay de um item da fila de falhas.
- A plataforma cria um novo evento pendente a partir do payload guardado e registra quem
  fez o replay ([09:36] Sofia).

**Fluxos alternativos e exceções**
- Usuário sem papel de administrador é recusado.
- Item de fila de falhas inexistente é recusado.

**Erros previstos**
- Permissão insuficiente.
- Item não encontrado.

**Prioridade:** média

### FR-12 Registrar o evento junto com a mudança de status

O evento de notificação é registrado na mesma operação que muda o status do pedido. Se o
registro do evento falha, a mudança de status também é desfeita.

**Fluxo principal**
- A operação de mudança de status registra o evento antes de concluir.
- Concluída a operação, o status mudou e o evento está registrado.

**Fluxos alternativos e exceções**
- Falha ao registrar o evento: a mudança de status é revertida e o cliente da API recebe
  erro ([09:40] Bruno).

**Erros previstos**
- Falha de registro do evento (por exemplo, payload acima do limite de tamanho).

**Prioridade:** alta

## Requisitos não funcionais

**Latência**
- A notificação deve chegar em menos de 10 segundos no caminho feliz ([09:02] Marcos). O
  pior caso da fila é 2 segundos de polling mais o tempo de envio ([09:10] Larissa).

**Confiabilidade e integridade**
- Se a mudança de status foi confirmada, o evento foi registrado; se foi revertida, o
  evento não existe ([09:41] Diego).
- Garantia de entrega at-least-once ([09:24] Diego).
- A ordem de notificação é garantida por pedido, não globalmente ([09:12] Diego).

**Segurança**
- Cada envio é assinado com HMAC-SHA256 sobre o corpo ([09:20] Sofia).
- Cada endpoint tem uma secret única, nunca uma secret global; rotação com convivência de
  24 horas ([09:21] Sofia).
- A URL de destino tem que usar HTTPS ([09:23] Sofia).
- O replay administrativo exige papel de administrador e é auditado ([09:36] Sofia).
- A plataforma nunca registra a secret nem o corpo assinado em log.
- A revisão de segurança da geração de secret e do HMAC acontece antes do deploy, com pelo
  menos dois dias úteis reservados ([09:46] Sofia).

**Limites**
- O corpo da notificação não pode passar de 64 KB; acima disso, a plataforma trata como
  erro em vez de truncar ([09:23] Sofia; [09:24] Diego).
- Timeout de 10 segundos por tentativa de envio ([09:42] Diego).

**Observabilidade**
- Logs estruturados de publicação, tentativa de entrega, falha permanente e replay.
- Métricas de eventos pendentes, tentativas por resultado, duração de envio e itens em
  fila de falhas.
- Correlação entre a requisição que mudou o status e a entrega da notificação.

**Compatibilidade**
- A feature reaproveita a stack e os padrões atuais do projeto, sem bibliotecas novas de
  erro, log ou validação ([09:29] Bruno; [09:30] Larissa).
- Os contratos HTTP existentes de pedidos, clientes e produtos não mudam.

## Decisões e trade-offs principais

O detalhamento e o contexto completo estão nos ADRs; aqui vai o resumo.

### Decisão: registrar o evento numa tabela outbox no banco atual, entregue por um processo separado
- **Justificativa:** garante atomicidade entre mudança de status e registro do evento sem
  acoplar a latência do fluxo de pedidos a um endpoint externo, e sem subir infraestrutura
  nova ([ADR-001](adrs/ADR-001-outbox-no-mysql.md), [ADR-002](adrs/ADR-002-worker-separado-em-polling.md)).
- **Trade-off:** a entrega deixa de ser imediata (2 segundos de polling no pior caso) e
  passa a existir um segundo processo para operar.

### Decisão: retry com backoff exponencial de 5 tentativas e fila de falhas separada
- **Justificativa:** tolera indisponibilidades longas do cliente sem deixar eventos
  pendurados para sempre ([ADR-003](adrs/ADR-003-retry-com-backoff-e-dlq.md)).
- **Trade-off:** uma notificação pode demorar até cerca de 15 horas em cenário de falha
  prolongada, e itens que falham em definitivo exigem ação manual.

### Decisão: entrega at-least-once com identificador de evento para o cliente deduplicar
- **Justificativa:** exactly-once exigiria coordenação dos dois lados; at-least-once com
  identificador resolve a maioria dos casos com muito menos complexidade
  ([ADR-005](adrs/ADR-005-entrega-at-least-once-com-x-event-id.md)).
- **Trade-off:** a responsabilidade de deduplicar fica com o cliente, o que precisa ser
  bem comunicado no portal de desenvolvedor.

### Decisão: assinatura HMAC-SHA256 com secret única por endpoint e rotação com convivência de 24 horas
- **Justificativa:** o cliente valida autenticidade e integridade com um mecanismo padrão,
  e o vazamento de uma secret afeta só um endpoint
  ([ADR-004](adrs/ADR-004-assinatura-hmac-sha256-secret-por-endpoint.md)).
- **Trade-off:** a plataforma precisa manter e proteger uma secret por endpoint e suportar
  duas secrets válidas durante a rotação.

## Dependências

### Técnica: alteração no fluxo de mudança de status
A operação de mudança de status do pedido precisa passar a registrar o evento na mesma
transação. É a única alteração de comportamento no código existente ([09:40] Bruno).

### Técnica: novo processo de entrega em produção
A operação precisa provisionar e monitorar um segundo processo, além do servidor da API
([09:11] Diego).

### Organizacional: janela de revisão de segurança
A engenheira de segurança precisa de pelo menos dois dias úteis reservados para revisar a
geração de secret e a assinatura antes do deploy ([09:46] Sofia).

### Externa: endpoints dos clientes
Os clientes solicitantes precisam expor endpoints HTTPS capazes de validar a assinatura e
responder em até 10 segundos, e implementar a deduplicação por identificador de evento
([09:25] Diego; [09:26] Marcos).

## Riscos e mitigação

### A escrita extra na operação de mudança de status aumenta a contenção em picos
- **Probabilidade:** média
- **Impacto:** mudanças de status ficam mais lentas sob carga alta, afetando o fluxo
  principal de pedidos.
- **Mitigação:**
  - Consultar os endpoints interessados por índice e resolver o filtro de status em
    memória, sem consultas pesadas dentro da transação.
  - Monitorar o tempo da operação de mudança de status e a duração de envio.
  - Manter uma forma de desligar a publicação de eventos sem novo deploy da API.
- **Plano de contingência:** desativar a publicação de eventos até otimizar, mantendo o
  fluxo de pedidos intacto; os eventos do período serão perdidos, o que é aceitável frente
  a degradar o fluxo principal.

### Um cliente com muitas mudanças de status simultâneas recebe uma rajada de chamadas
- **Probabilidade:** média
- **Impacto:** o processo de entrega sobrecarrega o endpoint do cliente ([09:38] Diego).
- **Mitigação:**
  - Acompanhar a métrica de tentativas de entrega por endpoint.
  - Avaliar rate limiting de saída se a métrica indicar necessidade (ponto em aberto no
    RFC).
- **Plano de contingência:** desativar temporariamente o endpoint do cliente afetado.

### Secret vazada em log do lado do cliente
- **Probabilidade:** baixa, mas já aconteceu com um cliente ([09:22] Diego)
- **Impacto:** um terceiro pode forjar notificações para aquele endpoint.
- **Mitigação:**
  - Secret única por endpoint limita o dano a um cliente.
  - Rotação com convivência de 24 horas permite trocar sem downtime de verificação.
  - A plataforma nunca loga a secret nem o corpo assinado.
- **Plano de contingência:** rotação imediata e, se necessário, desativação do endpoint até
  o cliente confirmar a correção.

### O prazo de 3 sprints não cobre a revisão de segurança
- **Probabilidade:** média
- **Impacto:** o deploy atrasa ou a revisão de segurança é encurtada.
- **Mitigação:**
  - A estimativa de 3 sprints já inclui a revisão da Sofia no fim ([09:47] Larissa).
  - Reservar os dois dias úteis de revisão no cronograma desde o início ([09:46] Sofia).
- **Plano de contingência:** entregar o CRUD de configuração e a publicação de eventos numa
  primeira etapa e a entrega efetiva numa segunda, se o prazo apertar.

## Critérios de aceitação

- Um cliente consegue cadastrar, editar, listar e remover endpoints de webhook pela API.
- Cada endpoint recebe apenas as notificações dos status que declarou.
- A secret é entregue na criação, não aparece nas listagens e pode ser rotacionada com
  convivência de 24 horas.
- Toda mudança de status confirmada de um pedido com endpoint interessado gera pelo menos
  uma notificação registrada.
- Se o registro do evento falha, a mudança de status é revertida.
- A notificação chega ao endpoint em menos de 10 segundos quando o endpoint responde na
  hora.
- Uma entrega que falha é retentada 5 vezes com a progressão definida e, esgotada, vai para
  a fila de falhas.
- Cada notificação carrega um identificador único de evento.
- O cliente consulta o histórico das últimas entregas de um endpoint.
- Só um administrador reprocessa um item da fila de falhas, e a ação fica registrada com o
  autor.
- Nenhum item marcado como fora de escopo aparece na entrega.

## Estratégia de testes e validação

**Tipos de teste obrigatórios**
- Testes de integração do fluxo de mudança de status com e sem endpoints interessados,
  incluindo o caso de rollback quando o registro do evento falha.
- Testes de integração do CRUD de configuração e da rotação de secret.
- Testes do processo de entrega: sucesso, timeout, erro do cliente, sequência de retry até
  a fila de falhas.
- Teste de segurança da assinatura: verificação da secret atual e da anterior durante a
  convivência; recusa de URL sem HTTPS.
- Teste de permissão do endpoint de replay.

**Abordagem de validação**
- TDD para a lógica de retry e backoff e para o filtro de status.
- Revisão de segurança guiada por roteiro, conduzida pela engenheira de segurança, antes do
  deploy ([09:46] Sofia).
- Nenhum teste existente de pedidos, autenticação, clientes ou produtos pode quebrar.

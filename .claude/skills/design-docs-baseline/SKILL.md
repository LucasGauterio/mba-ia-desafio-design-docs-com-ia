---
name: design-docs-baseline
description: >-
  Primeiro passo do workflow design-docs. Inspeciona o código de uma aplicação
  JÁ EXISTENTE e produz o baseline: CLAUDE.md (contexto da aplicação) e
  .claude/references/codebase/ (mapa detalhado + pontos de extensão). Use antes
  de analisar qualquer transcrição de reunião. IGNORA os arquivos do desafio
  (DESAFIO.md, TRANSCRICAO.md, README.md): eles não descrevem a aplicação.
---

# design-docs-baseline: baseline da aplicação existente

## Objetivo

Antes de projetar qualquer feature nova, é preciso um retrato fiel do que já existe.
Esta skill lê o código-fonte e escreve três artefatos que servem de contexto estável
para todas as etapas seguintes do workflow.

## Regras

1. **Fonte = só o código.** Baseie tudo em `src/`, `prisma/`, `tests/`, configs de build,
   `package.json`, `docker-compose.yml`. **Não leia nem cite** `DESAFIO.md`, `TRANSCRICAO.md`
   ou o `README.md`: esses arquivos são sobre um desafio/feature futura, não sobre a
   aplicação implementada.
2. **Nada de feature nova.** O baseline descreve o sistema como ele é hoje. Se um mecanismo
   não existe no código (fila, evento, webhook, cache...), registre isso como **ausência
   observada**, não como algo a construir.
3. **Rastreável.** Toda afirmação relevante aponta para um caminho de arquivo real.
4. **Não altere código.** Só cria/atualiza os três arquivos de saída abaixo.

## Passos

1. Mapear a estrutura: linguagem, framework, gerenciador de pacotes, versões
   (`package.json`, lockfile), scripts, camadas de pastas.
2. Identificar os entrypoints (servidor HTTP, workers, CLIs, seeds) e o wiring de
   dependências / injeção.
3. Levantar o modelo de dados (ORM/migrations): entidades, relações, índices, enums,
   invariantes (transações, sequências, histórico/auditoria).
4. Levantar convenções transversais: autenticação/autorização, tratamento de erros,
   logging, validação de entrada, formato de resposta HTTP, configuração/env.
5. Levantar a suíte de testes: ferramenta, o que cobre, helpers/factories.
6. Listar **ausências observadas** relevantes para design de features (mensageria,
   eventos, agendamento, cache, notificação externa, rate limiting...).
7. Escrever os três arquivos de saída.

## Saídas

### 1. `CLAUDE.md` (raiz do repo)

Contexto operacional da aplicação para humanos e para o agente. Curto e factual.
Seções sugeridas:

- **O que é**: tipo de sistema, domínio, em uma frase.
- **Stack**: runtime, linguagem, framework, ORM, banco, libs centrais, com versões.
- **Estrutura**: como os módulos/pastas se organizam e o padrão que cada módulo segue.
- **Convenções transversais**: auth, erros e códigos de erro, logging, validação,
  resposta HTTP, configuração.
- **Modelo de dados**: entidades principais e invariantes (transações, auditoria).
- **Entrypoints e execução**: como rodar, migrar, semear, testar.
- **Ausências**: o que o sistema deliberadamente ainda não tem.
- **Regras para o agente**: não editar `src/`, `prisma/`, `tests/`, configs; a entrega
  deste repositório é documental. Referência às rules em `.claude/rules/`.

Não incluir: roadmap, features futuras, nada derivado do desafio.

### 2. `.claude/references/codebase/existing-app.md`

Mapa detalhado, mais longo que o `CLAUDE.md`, para consulta durante a autoria dos docs:
entrypoints com caminho, wiring de DI, tabela de módulos (arquivos e responsabilidade),
modelo de dados por entidade, máquina(s) de estado, fluxos transacionais críticos,
middlewares na ordem, classes de erro e seus códigos/status, formato de log, testes.

### 3. `.claude/references/codebase/integration-points.md`

Catálogo dos **pontos de extensão** que uma feature nova provavelmente vai tocar, cada um
com: caminho do arquivo, o que faz hoje, como uma feature se conectaria, cuidado/risco.
Exemplos de categorias: métodos de serviço transacionais, middlewares, classes de erro
reutilizáveis, máquina de estados, entrypoints novos, cliente de banco por processo.
Esta é a matéria-prima da seção "Integração com o sistema existente" do FDD.

## Conclusão

Ao terminar, atualizar a linha `baseline` em `docs/_workbench/run-state.md` (se existir)
e, no contexto de construção do workflow, marcar o item correspondente em
`.claude/workflow-build/plan-progress.md`.

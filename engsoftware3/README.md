# Portfólio de Contribuições — Gustavo Garcia Gomes 4º ADS

---

## 1. Contribuições da Atividade de Engenharia de Software III (IAbertoti)

> **Repositório do Projeto:** [TeamMinerva/IAbertoti](https://github.com/TeamMinerva/IAbertoti)  
> **Objetivo:** Desenvolvimento de um *Knowledge Assistant* (Assistente de Conhecimento RAG) com **Spring AI** e **Ollama** aplicando e fundamentando defensavelmente **5 Design Patterns do GoF** (State, Command, Strategy, Observer e Composite), interface web Thymeleaf, suíte completa de testes e documentação formal com diagramas UML.

#### 1. Configuração da Infraestrutura de IA com Spring AI e Ollama (`1d07b5e`)
- **Atualização do Ecossistema Spring Boot e Dependências:**
  - Atualização do Spring Boot para a versão `3.4.2` com Java 21 no `pom.xml` e inclusão do Bill of Materials (BOM) do **Spring AI** (`spring-ai-bom` na versão `1.0.0-M6`).
  - Adição do starter `spring-ai-ollama` para comunicação nativa com modelos de linguagem open-source rodando localmente via Ollama.
  - Parametrização do endpoint local e seleção de modelo no `src/main/resources/application.properties` (`spring.ai.ollama.base-url=http://localhost:11434`, modelo padrão configurável `llama3.2` ou `qwen2.5:7b`).
- **Bean Central de Inferência:**
  - Criação da classe de configuração `@Configuration` `SpringAiConfig` (`br.com.minerva.iabertoti.config.SpringAiConfig`), expondo o bean gerenciado `ChatClient` construído via `ChatClient.Builder`.

#### 2. Serviço de Base de Conhecimento e Pipeline RAG em Memória (`1d07b5e`)
- **Modelagem de Documento:**
  - Criação do record imutável `DocumentoConhecimento` (`br.com.minerva.iabertoti.model.DocumentoConhecimento`), encapsulando nome do arquivo, tipo MIME, data/hora de ingestão e conteúdo textual normalizado.
- **Mecanismo de Ingestão e Montagem de Prompt Contextual:**
  - Implementação de `ConhecimentoService` (`br.com.minerva.iabertoti.service.ConhecimentoService`):
    - Ingestão de arquivos (`MultipartFile`) aceitando extensões `.txt` e `.md`, com validação de tipo, decodificação em UTF-8 e sanitização de quebras de linha (`carregarDocumento`).
    - Ingestão direta de conteúdo em texto para testes ou automações (`carregarTexto`).
    - Gerenciamento de estado em memória do documento ativo (`possuiDocumentoCarregado`, `obterDocumentoAtual`, `limparDocumento`).
    - Estruturação do prompt contextual (RAG simples) com `PromptTemplate`, `SystemMessage` e `UserMessage`, orientando a LLM a atuar estritamente baseada no documento carregado e instruindo o modelo a admitir explicitamente quando a informação não constar no texto.

#### 3. Conexão do Spring AI com os 5 Padrões de Projeto GoF e Interface Web (`6ade885`)
- **Padrão Strategy (`br.com.minerva.iabertoti.strategy`):**
  - Criação da classe `SpringAiKnowledgeStrategy` implementando `EstrategiaResposta` com anotações `@Component("springAiKnowledgeStrategy")` e `@Primary`. Injeta `ChatClient` e `ConhecimentoService` para inferência real com contexto RAG e tratamento de exceções com fallback amigável caso o daemon do Ollama esteja offline.
  - Refatoração de `EcoStrategy` para atuar como mock determinístico em desenvolvimento e testes rápidos sem necessidade de GPU/LLM.
  - Suporte à alternância dinâmica da estratégia ativa em tempo de execução via `ChatService`.
- **Padrão Command (`br.com.minerva.iabertoti.command`):**
  - Desacoplamento entre `ChatController` e as operações de negócio por meio de comandos executáveis:
    - `CarregarDocumentoCommand`: orquestra a validação e ingestão de documentos.
    - `LimparDocumentoCommand`: esvazia a base de conhecimento e registra aviso na sessão.
    - `AlternarEstrategiaCommand`: permite alternar em runtime entre `SpringAiKnowledgeStrategy` e `EcoStrategy`.
  - Atualização do `ChatController` para despachar ações exclusivamente instanciando e executando instâncias de `Comando`.
- **Padrão State (`br.com.minerva.iabertoti.state`):**
  - Modelagem do ciclo de vida da sessão (`EstadoSessao`): `AguardandoPerguntaState` $\rightarrow$ `ProcessandoState` $\rightarrow$ `RespondidoState`.
  - Refatoração de `ProcessandoState` para encapsular a transição segura com `finalizarResposta(...)`, prevenindo condições de corrida e bloqueando novas perguntas simultâneas enquanto a IA responde.
  - Atualização de `SessaoChat` para gerenciar histórico de mensagens, documento ativo e máquina de estados.
- **Padrão Observer (`br.com.minerva.iabertoti.observer`):**
  - Criação do `MetricasObservador` integrando `ChatObservador`, coletando telemetria em tempo real (total de respostas geradas, total de caracteres processados e métricas acumuladas da sessão), acionado a cada conclusão de inferência junto com `LogObservador`.
- **Padrão Composite na Camada de Visão (Thymeleaf UI):**
  - Composição hierárquica da interface web através de fragmentos:
    - Raiz: `src/main/resources/templates/chat.html` com layout responsivo, badges dinâmicos de State e feedback visual.
    - Galhos Compostos: `src/main/resources/templates/fragments/painel-conhecimento.html` (com formulário de upload de arquivos, card de status do documento ativo, seletor dinâmico de Strategy e painel de telemetria de métricas do Observer) e `lista-mensagens.html`.
    - Folha Individual: `src/main/resources/templates/fragments/mensagem.html`.

#### 4. Suíte de Testes Automatizados (`1d07b5e`, `6ade885`)
- Implementação de **22 testes unitários** com JUnit 5 cobrindo 100% dos fluxos e regras:
  - `ConhecimentoServiceTest`: ingestão com sucesso, rejeição de formatos não suportados, limpeza da base e montagem dos prompts de sistema e usuário.
  - `SpringAiKnowledgeStrategyTest`: verificação de inferência com mock do `ChatClient` do Spring AI e resiliência de fallback.
  - `CommandTest`: validação isolada dos 4 comandos do sistema (`EnviarPerguntaCommand`, `CarregarDocumentoCommand`, `LimparDocumentoCommand`, `AlternarEstrategiaCommand`).
  - `ChatServiceTest`: ciclo completo de envio de mensagens, troca de estratégias e notificação de observadores.
  - `SessaoChatTest`: transições de estado e proteção contra envios concorrentes em processamento.
  - `IabertotiApplicationTests`: validação de carregamento do contexto Spring Boot e beans de IA.

#### 5. Documentação Acadêmica, Diagramas UML (PlantUML e Mermaid) e Roteiro de Demonstração (`2521d33`)
- **Diagramas UML Formais (PlantUML):**
  - Criação de 6 especificações UML completas no diretório `docs/uml/`:
    - `arquitetura-geral.puml`: visão unificada demonstrando a arquitetura em camadas e os 5 design patterns conectados ao Spring AI.
    - `state.puml`: diagrama de classes e transição de estados da sessão de chat.
    - `command.puml`: diagrama do padrão Command e desacoplamento do controller.
    - `strategy.puml`: diagrama das estratégias de resposta e integração com Ollama.
    - `observer.puml`: diagrama do mecanismo de observadores e telemetria.
    - `composite.puml`: diagrama estrutural da árvore de templates Thymeleaf.
- **Renderização Mermaid:**
  - Criação do `docs/uml/README.md` contendo todos os diagramas transpostos para Mermaid com renderização visual nativa no GitHub.
- **Reformulação do `README.md` Principal:**
  - Justificativa conceitual do projeto como *Knowledge Assistant* corporativo.
  - Tabela comparativa e defensável dos 5 Design Patterns GoF implementados.
  - Exemplos explicativos com trechos de código de cada padrão.
  - Passo a passo de instalação com Ollama, execução Maven e roteiro prático de testes e validação.

#### 6. Histórico de Commits no Projeto IAbertoti

| Data | Hash | Mensagem | Branch(es) | Principais Arquivos / Escopo |
| :--- | :--- | :--- | :--- | :--- |
| 2026-11-15 | `1d07b5e` | `feat(knowledge): configurar Spring AI com Ollama e implementar servico de base de conhecimento` | `experimental`, `origin/experimental` | `pom.xml`, `SpringAiConfig.java`, `DocumentoConhecimento.java`, `ConhecimentoService.java`, `application.properties`, `ConhecimentoServiceTest.java`, `IabertotiApplicationTests.java` |
| 2026-11-15 | `6ade885` | `feat(patterns): conectar Spring AI aos 5 design patterns e interface Thymeleaf (Etapa 3)` | `experimental`, `origin/experimental` | `SpringAiKnowledgeStrategy.java`, `CarregarDocumentoCommand.java`, `LimparDocumentoCommand.java`, `AlternarEstrategiaCommand.java`, `MetricasObservador.java`, `ProcessandoState.java`, `ChatService.java`, `ChatController.java`, `chat.html`, `painel-conhecimento.html`, testes unitários |
| 2026-11-15 | `2521d33` | `docs: adicionar diagramas UML (PlantUML/Mermaid) e documentacao final no README` | `experimental`, `origin/experimental` | `README.md`, `docs/uml/README.md`, `arquitetura-geral.puml`, `state.puml`, `command.puml`, `strategy.puml`, `observer.puml`, `composite.puml` |


---

## 2. Contribuições na API (PRO4TECH-API)

#### 1. Autenticação JWT, Proteção de Rotas e Integração no Frontend
- **Backend — Autenticação e Middleware (`22a4085`, `99e0f8d`):**
  - Implementação dos endpoints `POST /auth/register` e `POST /auth/login` (montados em `/auth` e `/api/auth` no `backend/src/app.ts`).
  - Validação de entrada com Zod (`registerSchema` e `loginSchema` em `backend/src/schemas/auth.schema.ts`), hash de senha com `bcryptjs` e geração/verificação de tokens JWT com `jsonwebtoken`.
  - Criação do `authMiddleware` (`backend/src/middlewares/auth.middleware.ts`) com extração e validação de Bearer Token e tipagem de `req.user` (`backend/src/@types/express.d.ts`).
  - Refatoração posterior (`99e0f8d`) para padronizar a arquitetura em camadas (`auth.routes.ts`, `auth.controller.ts`, `auth.service.ts`, `auth.schema.ts`, `auth.middleware.ts`, `lib/prisma.ts`) e tratamento de erros Prisma (`P2002`).
  - Adição da variável `JWT_SECRET` em `backend/.env.example` e `docker-compose.yml`.
- **Frontend — Integração de Login e Cadastro (`cb9ae8c`):**
  - Criação de `frontend/src/services/authService.ts` (`registerUser`, `loginUser`, `saveSession`, `getSession`).
  - Integração das telas `frontend/src/telas/Cadastro.tsx` e `frontend/src/telas/Login.tsx` com a API de autenticação.

#### 2. Modelagem de Banco de Dados e Refatorações de Schema
- **Modelos de Projeto (`721f1d6`):**
  - Criação no `backend/prisma/schema.prisma` (e respectiva migration) do enum `ProjectStatus` (`PLANNED`, `IN_PROGRESS`, `DONE`) e dos modelos `Project`, `Epic`, `Feature` e `PBI`.
  - Definição dos relacionamentos hierárquicos (`Project` 1:N `Epic` 1:N `Feature` 1:N `PBI`) e relação N:N entre `PBI` e `Developer`.
- **Refatoração de Arrays e UI de Tecnologias (`70ec8d9`, `bea08d5`):**
  - Alteração de `Developer.skills` e `Project.technologies` de `String` para `String[]` no Prisma (`schema.prisma` + migration), schemas Zod (`developer.schema.ts`, `project.schema.ts`) e serviços (`developer.service.ts`, `project.service.ts`).
  - Atualização da tela `frontend/src/telas/CadastroProjeto.tsx` (`bea08d5`) substituindo o campo textual de tecnologias por caixa de seleção.

#### 3. Galeria de Desenvolvedores (`f5ef149`)
- Em co-autoria com Isaura de Lourdes:
  - Criação de `frontend/src/services/developerService.ts` (`getDevelopersGallery`, `createDeveloper`).
  - Adição da rota `POST /api/developers-gallery` em `backend/src/routes/developers-gallery.route.ts` utilizando `createDeveloperController`.
  - Integração da tela `frontend/src/telas/GaleriaDesenvolvedor.tsx` com o backend.

#### 4. Detalhes Completos do Projeto (`68ace3d`)
- **Backend:**
  - Implementação da rota `GET /api/projects/:id` (`project.routes.ts`), controller `getProjectByIdController` (`project.controller.ts`), validação de parâmetro `getProjectParamsSchema` (`project.schema.ts`) e consulta completa `getProjectById` (`project.service.ts`) retornando projeto, épicos, features, PBIs e equipe de desenvolvedores via Prisma, com tratamento de erros (incluindo `P2020`).
- **Frontend:**
  - Atualização de `frontend/src/services/projectService.ts` e integração com a página `frontend/src/pages/DetalhesProjeto.tsx`.

#### 5. Detalhes do Desenvolvedor e Otimização dos Chats Contextuais
- **Endpoint de Detalhes do Desenvolvedor (`54f3232`):**
  - Implementação de `getDeveloper(id)` em `backend/src/services/developer.service.ts` e dos DTOs tipados em `backend/src/schemas/developer.schema.ts` (`DeveloperDetailsResponse`, `DeveloperTechnologyDTO`, `DeveloperProjectDTO`, `DeveloperBugDTO`, `DeveloperPBIDTO`), retornando competências técnicas, projetos participados, bugs/soluções registradas e PBIs atribuídos.
- **Otimização dos Chats de Desenvolvedor e Projeto (`10c879b`):**
  - Reestruturação do módulo de chat de desenvolvedor no serviço de IA em camadas (`ai/app/routers/developer_chat.py`, `ai/app/services/developer_chat_service.py`, `ai/app/schemas/developer_chat_schema.py`, `ai/app/prompts/developer_prompt.py`), mantendo compatibilidade em `ai/app/developer_chat.py`.
  - Unificação das rotas contextuais de chat no backend (`POST /api/chat/projects/:id` e `POST /api/chat/developers/:id` em `chat-project.routes.ts` e `chat-project.controller.ts`) e criação do endpoint `POST /api/developers/:id/chat` (`developer.routes.ts`).
  - Otimizações em `ai/app/rag.py`, `backend/src/services/indexacao.service.ts` e `backend/src/services/project.service.ts`.
  - Atualização de `frontend/src/components/ChatDesenvolvedor.tsx`, `frontend/src/pages/DesenvolvedorDetalhes.css` e `frontend/src/services/chatService.ts`.
  - Adição de testes automatizados em `ai/tests/test_developer_chat.py` e `ai/tests/test_rag_desenvolvedor.py`.

#### 6. Infraestrutura de IA — LightRAG + PostgreSQL + Ollama (`9077b94`, `ebaa221`)
- **Correção de Storages e Contexto (`9077b94`):**
  - Passagem explícita de `PGKVStorage`, `PGVectorStorage`, `PGTableGraphStorage`, `PGDocStatusStorage` e `workspace` na inicialização do `LightRAG` em `ai/app/rag.py`.
  - Adição de suporte a `system_prompt` em `ollama_client.ask` (`ai/app/ollama_client.py`).
- **Reindexação por `doc_id` e Controle de `think` (`ebaa221`):**
  - Inclusão de `doc_id` opcional em `POST /knowledge/insert` (`ai/app/main.py`) e suporte em `rag.insert` para deletar/substituir documentos existentes (`adelete_by_doc_id` + `ainsert`).
  - Suporte ao parâmetro `think` em `ollama_client.chat` e `ollama_client.ask`, além de ajustes nos testes `ai/tests/test_knowledge.py`.

#### 7. Chat Geral com Base de Conhecimento Completa e Histórico de Conversas (`520ff59`, `fa9ff1e`, `b5e3825`)
- **Serviço de IA e Backend (`520ff59`):**
  - Criação do router `ai/app/routers/general_chat.py` com os endpoints `POST /chat/general` e `POST /chat`, service `ai/app/services/general_chat_service.py`, schema `ai/app/schemas/general_chat_schema.py` e prompt `ai/app/prompts/general_prompt.py`.
  - Implementação da recuperação na base de conhecimento completa em `ai/app/rag.py` com retorno das fontes consultadas (projetos e desenvolvedores) e tradução de status para português (`Planejado`, `Em andamento`, `Concluído`, `Ativo`/`Inativo`).
  - Criação dos endpoints `POST /api/chat/general` e `POST /api/chat` no backend (`general-chat.routes.ts`, `general-chat.controller.ts`, `general-chat.service.ts`, `general-chat.schema.ts`), além de enriquecimento da indexação de desenvolvedores e bugs em `indexacao.service.ts` e `bug.service.ts`.
  - Adição de testes automatizados em `ai/tests/test_general_chat.py` e `ai/tests/test_rag_geral.py`.
- **Histórico, Persistência de Fontes e Gerenciamento de Conversas (`fa9ff1e`, `b5e3825`):**
  - Migration no Prisma (`backend/prisma/schema.prisma`) para `Conversation` (`userId` opcional) e `Message` (coluna `sources` em `Json?`), além de extração opcional de usuário autenticado via JWT em `conversation.controller.ts`.
  - Integração isolada do histórico por conversa e suporte a cancelamento de requisições abortadas em `frontend/src/hooks/useGeneralChat.ts` e `frontend/src/services/generalChatService.ts`.
  - Implementação de **renomeação** (`PATCH /api/conversations/:id`, `PUT /api/conversations/:id`) e **exclusão** (`DELETE /api/conversations/:id`) de conversas no backend e na interface `frontend/src/pages/Home.tsx` (incluindo estilização da barra de rolagem em `frontend/src/index.css`).

#### 8. Análise de Projeto com IA no Cadastro e Chat de Rascunho (`chat-cadastroProjeto` / `stash@{0}`)
- Implementação da análise automática e chat contextual sobre o rascunho durante o cadastro de um projeto, identificando informações ausentes, inconsistências, ambiguidades e sugestões de melhoria com apoio da base de conhecimento:
  - **Serviço de IA:** criação de `ai/app/routers/draft_chat.py` (`POST /chat/project-draft` e `POST /chat/project-draft/analyze`), `ai/app/services/draft_chat_service.py`, `ai/app/schemas/draft_chat_schema.py`, `ai/app/prompts/draft_prompt.py`, registro do router em `ai/app/main.py` e suíte de testes `ai/tests/test_draft_chat.py`.
  - **Backend:** adição das rotas `POST /api/chat/projects/draft` e `POST /api/chat/projects/draft/analyze` em `chat-project.routes.ts`, controllers (`chatRascunhoController`, `analisarRascunhoController`), schemas (`chatDraftBodySchema`, `analyzeDraftBodySchema`), serviços (`perguntarSobreRascunho`, `analisarRascunhoProjeto`) e testes unitários em `backend/tests/chat-draft.service.test.ts`.
  - **Frontend:** integração da análise do rascunho em `frontend/src/pages/CadastroProjeto.tsx`, exposição de `ChatLateralHandle` (`enviarPergunta`) e `onEnviandoChange` via `forwardRef` em `frontend/src/components/ChatLateral.tsx`, e atualização de `frontend/src/services/chatService.ts` e `frontend/src/services/rascunhoProjeto.ts`.

#### 9. Ajustes de Segurança (Escopo Planejado)
- Realização dos ajustes de segurança da aplicação, dando continuidade à camada de autenticação JWT e proteção de rotas já estruturada no backend.

---

### Histórico Completo de Commits (Todas as Branches)

#### Commits de Implementação e Refatoração (`16` commits nas branches/remotes)

| Data | Hash | Mensagem | Branch(es) | Principais Arquivos / Escopo |
| :--- | :--- | :--- | :--- | :--- |
| 2026-09-15 | `22a4085` | `feat: adicionando autenticação JWT e proteção de rotas` | `main`, `experimental`, `cadastro-login` e derivadas | `backend/src/middlewares/auth.ts`, `backend/src/routes/auth.ts`, `backend/src/@types/express.d.ts`, `docker-compose.yml` |
| 2026-09-15 | `cb9ae8c` | `feat: integração da autenticação de usuários` | `main`, `experimental`, `cadastro-login` e derivadas | `frontend/src/services/authService.ts`, `Cadastro.tsx`, `Login.tsx` |
| 2026-09-17 | `721f1d6` | `feat: adicionando modelos de projeto` | Todas as branches principais e de feature | `backend/prisma/schema.prisma`, migration (`Project`, `Epic`, `Feature`, `PBI`, `ProjectStatus`) |
| 2026-09-20 | `99e0f8d` | `refactor: padronizar arquitetura de autenticação em camadas` | `main`, `experimental`, `cadastro-login` e derivadas | `auth.controller.ts`, `auth.service.ts`, `auth.schema.ts`, `auth.routes.ts`, `auth.middleware.ts`, `lib/prisma.ts` |
| 2026-09-24 | `70ec8d9` | `refactor: alterar skills e technologies para array de strings` | `main`, `experimental` e derivadas | `schema.prisma`, migration, schemas e services de `developer` e `project` |
| 2026-09-24 | `bea08d5` | `refactor: alterando as tecnologias para caixa de seleção` | `main`, `experimental` e derivadas | `frontend/src/telas/CadastroProjeto.tsx` |
| 2026-09-26 | `f5ef149` | `feat: integracao da galeria de desenvolvedores com backend` | `main`, `experimental` e derivadas | `developers-gallery.route.ts`, `frontend/src/services/developerService.ts`, `GaleriaDesenvolvedor.tsx` |
| 2026-10-05 | `68ace3d` | `feat: implementando endpoint e integrando os detalhes completos do projeto` | `main`, `experimental`, `detalhes-projetos` e derivadas | `project.controller.ts`, `project.routes.ts`, `project.schema.ts`, `project.service.ts`, `DetalhesProjeto.tsx`, `projectService.ts` |
| 2026-10-08 | `9077b94` | `fix: corrige storages postgres no lightrag e injecao de contexto no ollama` | `main`, `experimental`, `servicoIA` e derivadas | `ai/app/rag.py`, `ai/app/ollama_client.py`, `.gitignore` |
| 2026-10-08 | `54f3232` | `feat: implementando endpoint para detalhar desenvolvedor` | `detalhes-desenvolvedores`, `detalhes-desenvolvedores-2`, `backup-detalhes-desenvolvedores` | `backend/src/schemas/developer.schema.ts`, `backend/src/services/developer.service.ts` |
| 2026-10-19 | `ebaa221` | `refactor: ajustes pro merge` | `main`, `experimental`, `chat-geral` e derivadas | `ai/app/main.py`, `ai/app/ollama_client.py`, `ai/app/rag.py`, `ai/tests/test_knowledge.py`, `backend/.env.example`, `backend/package.json` |
| 2026-10-19 | `520ff59` | `feat: implementação do chat geral utilizando a base de conhecimento completa` | `main`, `experimental`, `chat-geral`, `historico-chat` e derivadas | `general_chat` (AI + backend), `general_prompt.py`, `rag.py`, `indexacao.service.ts`, `Home.tsx`, `useGeneralChat.ts`, `generalChatService.ts`, testes |
| 2026-10-22 | `fa9ff1e` | `fix: corrige integracao do historico de conversas no chat geral` | `main`, `experimental`, `historico-chat` e derivadas | `schema.prisma` + migration (`Conversation`/`Message`), `conversation.*`, `general-chat.*`, `useGeneralChat.ts`, `Home.tsx`, `generalChatService.ts` |
| 2026-10-22 | `b5e3825` | `feat: adicionar renomeacao e exclusao de conversas` | `main`, `experimental`, `historico-chat` e derivadas | `conversation.controller.ts`, `conversation.routes.ts`, `conversation.schema.ts`, `conversation.service.ts`, `Home.tsx`, `useGeneralChat.ts`, `index.css` |
| 2026-10-23 | `10c879b` | `feat: otimizacao dos chats de dev` | `detalhes-desenvolvedores-2`, `origin/detalhes-desenvolvedores-2` | `developer_chat` em camadas (`routers`, `services`, `schemas`, `prompts`), `rag.py`, `chat-project.*`, `indexacao.service.ts`, `project.service.ts`, `ChatDesenvolvedor.tsx`, testes |
| 2026-10-25 | `89e7214` | `Merge branch 'detalhes-desenvolvedores-2' into projetos-semelhantes` *(commit de consolidação com diff das features de chat geral, histórico, detalhes e chat de dev)* | `main`, `experimental`, `integracao-projetos-desenvolvedores`, `chat-cadastroProjeto` e derivadas | 58 arquivos integrando IA (`developer_chat`, `general_chat`, `rag.py`), backend (`conversation`, `developer`, `bug`, `general-chat`) e frontend (`Home.tsx`, `DesenvolvedorDetalhes.tsx`, `ChatDesenvolvedor.tsx`) |


#### Commits de Merge e Pull Requests (`15` merges nas branches/remotes)

| Data | Hash | Descrição |
| :--- | :--- | :--- |
| 2026-09-20 | `47ec262` | `Merge branch 'experimental' into cadastro-login` |
| 2026-09-20 | `4cd82ef` | `Merge pull request #2 from TeamMinerva/cadastro-login` |
| 2026-09-21 | `31dc39d` | `Merge pull request #1 from TeamMinerva/docs` |
| 2026-10-06 | `e86c21b` | `Merge pull request #7 from TeamMinerva/servicoIA` |
| 2026-10-07 | `076de53` | `Merge branch 'cadastro-bugs' into detalhes-projetos` |
| 2026-10-07 | `d2b10e8` | `Merge pull request #9 from TeamMinerva/detalhes-projetos` |
| 2026-10-07 | `ee27774` | `merge: integracao da branch cadastro-bugs com experimental e correcao de compatibilidade` |
| 2026-10-08 | `339ad49` | `Merge pull request #11 from TeamMinerva/integracao-bugs-experimental` |
| 2026-10-19 | `73b4035` | `Merge remote-tracking branch 'origin/chat-projetos' into experimental` |
| 2026-10-19 | `8663407` | `Merge remote-tracking branch 'origin/indexacao-lightrag' into experimental` |
| 2026-10-22 | `5d35180` | `Merge branch 'historico-chat' into experimental` |
| 2026-10-22 | `7621727` | `Merge branch 'origin/experimental' into detalhes-desenvolvedores` |
| 2026-10-22 | `c96e138` | `Merge remote-tracking branch 'origin/detalhes-desenvolvedores' into detalhes-desenvolvedores-2` |
| 2026-10-26 | `1942d6b` | `Merge branch 'integracao-projetos-desenvolvedores' into experimental` |
| 2026-10-26 | `43f4dc8` | `Merge pull request #16 from TeamMinerva/experimental` |

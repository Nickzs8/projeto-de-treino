# Backlog de Issues — AI Gym Companion (MVP)

Este documento registra todas as **Issues de Engenharia** do projeto organizadas por fases, prontas para versionamento no repositório e publicação nas issues do GitHub.

---

## Índice das Fases

- [Fase 0: Fundação da Arquitetura & Setup do Monorepo](#fase-0-fundação-da-arquitetura--setup-do-monorepo)
- [Fase 1: Persistência Relacional, Domínio de Treino & Autenticação](#fase-1-persistência-relacional-domínio-de-treino--autenticação)
- [Fase 2: Engenharia de Agentes com LangChain & Ferramentas Determinísticas](#fase-2-engenharia-de-agentes-com-langchain--ferramentas-determinísticas)
- [Fase 3: Orquestração Multi-Agente com LangGraph & Checkpointing](#fase-3-orquestração-multi-agente-com-langgraph--checkpointing)
- [Fase 4: Streaming em Tempo Real com Server-Sent Events (SSE)](#fase-4-streaming-em-tempo-real-com-server-sent-events-sse)
- [Fase 5: Frontend Mobile no React Native Expo](#fase-5-frontend-mobile-no-react-native-expo)
- [Fase 6: Guardrails Biomédicos, Observabilidade & Qualidade](#fase-6-guardrails-biomédicos-observabilidade--qualidade)

---

## Fase 0: Fundação da Arquitetura & Setup do Monorepo

### `[ISSUE-001]` Estrutura do Monorepo, Orquestração Docker & Variáveis de Ambiente
* **Fase:** 0 — Fundação
* **Tipo:** Task / Infraestrutura
* **Descrição:** Estruturar as pastas raiz do monorepo separando `backend/`, `mobile/` e arquivos de infraestrutura compartilhada. Criar o `docker-compose.yml` para rodar o PostgreSQL 16 com volume persistente e checagem de saúde.
* **Arquivos Afetados:**
  - `docker-compose.yml`
  - `.env.example`
  - `.gitignore`
  - `README.md`
* **Critérios de Aceite:**
  - [ ] Executar `docker compose up -d` provisiona o PostgreSQL 16 na porta `5432` com checagem de saúde (`healthcheck`).
  - [ ] Arquivo `.env.example` na raiz documenta todas as chaves obrigatórias (`DATABASE_URL`, `JWT_SECRET`, `GEMINI_API_KEY`).
  - [ ] Arquivo `.gitignore` ignora artefatos de build de Python (`.venv`, `__pycache__`), Expo (`.expo`, `node_modules`) e arquivos de segredos (`.env`).

---

### `[ISSUE-002]` Setup do Backend FastAPI, Tooling (uv/ruff) e Pipeline de Testes
* **Fase:** 0 — Fundação
* **Tipo:** Task / Backend
* **Descrição:** Configurar o projeto Python no diretório `backend/` utilizando `pyproject.toml` (com `uv` ou `poetry`), FastAPI, Uvicorn, SQLAlchemy 2.0, Alembic, Ruff (linter/formatter) e Pytest com suporte assíncrono (`pytest-asyncio`).
* **Arquivos Afetados:**
  - `backend/pyproject.toml`
  - `backend/app/main.py`
  - `backend/app/core/config.py`
  - `backend/tests/conftest.py`
  - `backend/tests/test_health.py`
* **Critérios de Aceite:**
  - [ ] `GET /health` responde `{"status": "ok", "database": "connected"}` com validação de conectividade via pool assíncrono.
  - [ ] Execução de `ruff check .` e `ruff format --check .` roda com zero erros.
  - [ ] Execução de `pytest` roda e passa sem warnings assíncronos.

---

### `[ISSUE-003]` Inicialização do App Mobile Expo com TypeScript & NativeWind
* **Fase:** 0 — Fundação
* **Tipo:** Task / Mobile
* **Descrição:** Inicializar o workspace `mobile/` utilizando Expo SDK 51+ com template TypeScript, configurando NativeWind v4 (Tailwind CSS), path aliases (`@/*`), Zustand e TanStack Query.
* **Arquivos Afetados:**
  - `mobile/package.json`
  - `mobile/app.json`
  - `mobile/tsconfig.json`
  - `mobile/tailwind.config.js`
  - `mobile/src/app/_layout.tsx`
* **Critérios de Aceite:**
  - [ ] O app compila e abre no Expo Go ou simulador iOS/Android com renderização de estilo Tailwind funcional.
  - [ ] Aliases `@/components`, `@/screens`, `@/services`, `@/hooks` e `@/types` compilam corretamente sem erro de resolução de módulo.

---

## Fase 1: Persistência Relacional, Domínio de Treino & Autenticação

### `[ISSUE-101]` Modelagem Relacional no PostgreSQL & Migrações Alembic
* **Fase:** 1 — Persistência & Domínio
* **Tipo:** Feature / Database
* **Descrição:** Criar os modelos declarativos do SQLAlchemy 2.0 assíncrono e estruturar o Alembic para versionamento do schema:
  1. `users`: UUID, email, hashed_password, created_at.
  2. `onboarding_checklists`: user_id, age, weight_kg, height_cm, experience_level, primary_goal, days_per_week, session_duration_minutes, training_location, equipment_access, injuries_history, completed.
  3. `workout_plans`: id, user_id, rationale, frequency_days, active, created_at.
  4. `workout_divisions`: id, plan_id, division_letter, muscle_groups (JSON/Array).
  5. `division_exercises`: id, division_id, name, sets, reps_range, target_rpe, rest_seconds, biomechanical_notes, order_index.
  6. `workout_execution_logs`: id, user_id, exercise_name, sets_completed, reps_completed, load_kg, rpe, performed_at.
* **Arquivos Afetados:**
  - `backend/app/domain/models/*`
  - `backend/alembic/versions/*`
  - `backend/app/infra/db/session.py`
* **Critérios de Aceite:**
  - [ ] Migrations rodam de forma idempotente (`alembic upgrade head` e `alembic downgrade -1`).
  - [ ] Relacionamentos em cascata e índices criados para busca por `user_id` e `exercise_name`.

---

### `[ISSUE-102]` Autenticação JWT Própria & Dependência `get_current_user`
* **Fase:** 1 — Persistência & Domínio
* **Tipo:** Feature / Backend Security
* **Descrição:** Implementar autenticação via tokens JWT com assinatura HMAC-SHA256 e hash de senhas com `bcrypt`. Expor endpoints de registro e login, além de dependência FastAPI `get_current_active_user` para proteção de rotas privadas.
* **Arquivos Afetados:**
  - `backend/app/core/security.py`
  - `backend/app/api/v1/auth.py`
  - `backend/app/domain/schemas/auth.py`
  - `backend/tests/test_auth.py`
* **Critérios de Aceite:**
  - [ ] `POST /api/v1/auth/register` cria usuário e retorna tokens de acesso (`access_token`, expiração em 7 dias).
  - [ ] `POST /api/v1/auth/login` valida credenciais e rejeita senhas incorretas com status `401 Unauthorized`.
  - [ ] Rotas protegidas com `Depends(get_current_user)` rejeitam tokens expirados ou malformados.

---

### `[ISSUE-103]` API REST de Onboarding & Sincronização do Perfil
* **Fase:** 1 — Persistência & Domínio
* **Tipo:** Feature / Backend API
* **Descrição:** Criar endpoints REST para persistência e consulta do perfil biomédico e preferências de treino coletados pelo app mobile.
* **Arquivos Afetados:**
  - `backend/app/api/v1/onboarding.py`
  - `backend/app/domain/schemas/checklist.py`
  - `backend/app/services/onboarding_service.py`
* **Critérios de Aceite:**
  - [ ] `POST /api/v1/onboarding/checklist` salva ou atualiza dados com validação Pydantic estrita (ex.: peso > 30kg, altura > 100cm, frequência 1 a 7).
  - [ ] `GET /api/v1/onboarding/checklist` recupera o status e campos preenchidos.
  - [ ] Flag `completed` é calculada automaticamente baseada na presença de todos os campos obrigatórios.

---

### `[ISSUE-104]` API REST de Registro e Consulta de Séries (`WorkoutExecutionLog`)
* **Fase:** 1 — Persistência & Domínio
* **Tipo:** Feature / Backend API
* **Descrição:** Implementar endpoints para registro de execuções de séries pós-treino (ou durante o treino) para alimentar o histórico que será consumido pelo agente de desempenho.
* **Arquivos Afetados:**
  - `backend/app/api/v1/logs.py`
  - `backend/app/domain/schemas/workout_log.py`
  - `backend/app/services/log_service.py`
* **Critérios de Aceite:**
  - [ ] `POST /api/v1/logs/batch` permite registrar múltiplas séries executadas em uma sessão.
  - [ ] `GET /api/v1/logs/recent?exercise_name=Supino` retorna o histórico ordenado por data descendente.

---

## Fase 2: Engenharia de Agentes com LangChain & Ferramentas Determinísticas

### `[ISSUE-201]` Schemas Tipados de Saída Estruturada com Google Gemini
* **Fase:** 2 — LangChain & IA
* **Tipo:** Feature / AI Engineering
* **Descrição:** Configurar o cliente `ChatGoogleGenerativeAI` (`gemini-1.5-flash` como padrão de velocidade e `gemini-1.5-pro` para prescrições densas). Implementar e validar o contrato Pydantic `GeneratedWorkoutPlanSchema` via `.with_structured_output()`.
* **Arquivos Afetados:**
  - `backend/app/agents/llm_factory.py`
  - `backend/app/agents/schemas/workout_plan.py`
  - `backend/tests/test_agents/test_structured_output.py`
* **Critérios de Aceite:**
  - [ ] O modelo retorna instâncias tipadas válidas de `GeneratedWorkoutPlanSchema` sem erros de parsing JSON.
  - [ ] Configuração de retry com fallback caso ocorra rate limit na API do Gemini.

---

### `[ISSUE-202]` Ferramentas Python Determinísticas de Métricas (@tool)
* **Fase:** 2 — LangChain & IA
* **Tipo:** Feature / Tooling
* **Descrição:** Criar as tools decoradas com `@tool` com schemas Pydantic de entrada e saída:
  1. `fetch_recent_workout_logs(user_id, exercise_name, weeks=4)`.
  2. `compute_progression_metrics(user_id, exercise_name)`:
     - Tonelagem: $\sum \text{séries} \times \text{reps} \times \text{carga}$.
     - 1RM Estimado via Fórmula de Epley: $\text{carga} \times (1 + \text{reps}/30)$ (para reps $\le 10$).
     - Tendência de RPE médio.
  3. `detect_plateau(user_id, exercise_name, threshold_weeks=3)`.
  4. `get_current_workout_plan(user_id)`.
* **Arquivos Afetados:**
  - `backend/app/agents/tools/metrics_tools.py`
  - `backend/app/agents/tools/schemas.py`
  - `backend/tests/test_tools/test_metrics_math.py`
* **Critérios de Aceite:**
  - [ ] Testes unitários validam a exatidão matemática das fórmulas (sem intervenção de LLM).
  - [ ] Tools operam assincronamente com o banco de dados via `AsyncSession`.

---

### `[ISSUE-203]` Implementação do `WorkoutPlannerAgent` com Restrições Biomecânicas
* **Fase:** 2 — LangChain & IA
* **Tipo:** Feature / AI Engineering
* **Descrição:** Implementar o agente de prescrição de treino utilizando `ChatPromptTemplate` especializado, consumindo os dados biomédicos e restrições de lesões do usuário.
* **Arquivos Afetados:**
  - `backend/app/agents/specialists/workout_planner.py`
  - `backend/app/agents/prompts/planner_prompts.py`
  - `backend/tests/test_agents/test_workout_planner.py`
* **Critérios de Aceite:**
  - [ ] Perfil com "dor lombar crônica" ou "hérnia" gera fichas sem agachamento livre pesado ou terra tradicional, registrando a substituição nas notas biomecânicas.
  - [ ] Perfil com local "Home gym / apenas halteres" não recebe prescrição de cabos ou máquinas.

---

### `[ISSUE-204]` Implementação do `PerformanceCoachAgent` com Tool Calling
* **Fase:** 2 — LangChain & IA
* **Tipo:** Feature / AI Engineering
* **Descrição:** Implementar o agente de acompanhamento acoplado às tools da `ISSUE-202`. O agente deve decidir autonomamente quando chamar ferramentas para consultar logs e fundamentar o diagnóstico de progressão ou sugestão de deload.
* **Arquivos Afetados:**
  - `backend/app/agents/specialists/performance_coach.py`
  - `backend/app/agents/prompts/coach_prompts.py`
  - `backend/tests/test_agents/test_performance_coach.py`
* **Critérios de Aceite:**
  - [ ] Perguntas sobre progresso invocam `compute_progression_metrics` antes da resposta em linguagem natural.
  - [ ] Identificação de estagnação de carga com RPE alto aciona recomendação explícita de deload.

---

## Fase 3: Orquestração Multi-Agente com LangGraph & Checkpointing

### `[ISSUE-301]` Estado Global `GymGraphState` & Guardião de Onboarding
* **Fase:** 3 — LangGraph
* **Tipo:** Feature / Multi-Agent
* **Descrição:** Definir o estado global `GymGraphState` (`TypedDict` com `messages`, `user_id`, `thread_id`, `checklist_completed`, `active_agent`, `workout_draft`). Criar o nó `onboarding_node` que inspeciona o checklist e interage caso existam dados faltantes.
* **Arquivos Afetados:**
  - `backend/app/graph/state.py`
  - `backend/app/graph/nodes/onboarding.py`
  - `backend/tests/test_graph/test_onboarding_guard.py`
* **Critérios de Aceite:**
  - [ ] Usuário sem checklist preenchido é retido no `onboarding_node`.
  - [ ] Resposta com dados completos atualiza o estado para `checklist_completed = True`.

---

### `[ISSUE-302]` Nó Supervisor e Roteamento Semântico Condicional
* **Fase:** 3 — LangGraph
* **Tipo:** Feature / Multi-Agent
* **Descrição:** Criar o nó roteador semântico (`router_node`) e a função de transição condicional (`route_decision`) para despachar para `onboarding`, `planner`, `coach` ou resposta direta.
* **Arquivos Afetados:**
  - `backend/app/graph/nodes/router.py`
  - `backend/app/graph/edges.py`
* **Critérios de Aceite:**
  - [ ] Mensagens de "trocar treino" são roteadas para o nó do planner.
  - [ ] Mensagens de "como foram minhas cargas" são roteadas para o nó do coach.
  - [ ] Dúvidas simples/saudações são respondidas sem invocar especialistas pesados.

---

### `[ISSUE-303]` Persistência Assíncrona de Sessões com `AsyncPostgresSaver`
* **Fase:** 3 — LangGraph
* **Tipo:** Feature / State Persistence
* **Descrição:** Integrar o checkpointer oficial `AsyncPostgresSaver` do LangGraph, conectando-o ao pool assíncrono do PostgreSQL para salvar checkpoints de cada nó por `thread_id`.
* **Arquivos Afetados:**
  - `backend/app/graph/checkpointer.py`
  - `backend/app/graph/builder.py`
  - `backend/tests/test_graph/test_persistence.py`
* **Critérios de Aceite:**
  - [ ] A tabela de checkpoints é provisionada pelo LangGraph automaticamente no banco.
  - [ ] Uma thread de conversa preserva contexto integral após reinicialização da aplicação.

---

### `[ISSUE-304]` Fluxo de Human-in-the-Loop (`interrupt()`) para Aprovação de Fichas
* **Fase:** 3 — LangGraph
* **Tipo:** Feature / Human-in-the-Loop
* **Descrição:** Inserir a interrupção do LangGraph (`interrupt()`) após o `WorkoutPlannerAgent` gerar o rascunho da periodização (`workout_draft`). O plano só deve ser persistido nas tabelas oficiais após confirmação do usuário.
* **Arquivos Afetados:**
  - `backend/app/graph/nodes/hitl_review.py`
  - `backend/app/graph/builder.py`
  - `backend/tests/test_graph/test_hitl.py`
* **Critérios de Aceite:**
  - [ ] Grafo suspende execução com payload do rascunho de ficha.
  - [ ] Ao receber `Command(resume={"approved": True})`, o plano é gravado no PostgreSQL.
  - [ ] Ao receber feedback com pedido de ajuste, o rascunho volta ao planner com as orientações do usuário.

---

## Fase 4: Streaming em Tempo Real com Server-Sent Events (SSE)

### `[ISSUE-401]` Padronização do Protocolo de Eventos SSE
* **Fase:** 4 — Streaming
* **Tipo:** Feature / Realtime Protocol
* **Descrição:** Criar o tradutor assíncrono que consome `graph.astream_events(version="v2")` e emite chunks estritamente formatados em `text/event-stream` (`agent_routing`, `tool_start`, `token`, `workout_generated`, `interrupt`, `done`, `error`).
* **Arquivos Afetados:**
  - `backend/app/api/v1/streaming/event_translator.py`
  - `backend/app/api/v1/streaming/contracts.py`
* **Critérios de Aceite:**
  - [ ] Cada evento segue o padrão `event: <tipo>\ndata: <json>\n\n`.
  - [ ] Tokens de texto parciais são emitidos sem delay perceptível.

---

### `[ISSUE-402]` Endpoint de Chat Streaming (`POST /api/v1/chat/stream`)
* **Fase:** 4 — Streaming
* **Tipo:** Feature / API Endpoint
* **Descrição:** Implementar a rota FastAPI que recebe a mensagem do usuário, injeta `thread_id` e `user_id` autenticado, gerencia a desconexão do cliente com `request.is_disconnected()` e devolve `EventSourceResponse`.
* **Arquivos Afetados:**
  - `backend/app/api/v1/chat.py`
  - `backend/tests/test_streaming.py`
* **Critérios de Aceite:**
  - [ ] Tempo até o primeiro byte (TTFB) inferior a 1 segundo emitindo o evento inicial de roteamento.
  - [ ] Se o cliente fechar o socket, a tarefa assíncrona é cancelada sem travar o pool do banco.

---

### `[ISSUE-403]` Endpoint de Retomada do Human-in-the-Loop (`POST /api/v1/chat/resume`)
* **Fase:** 4 — Streaming
* **Tipo:** Feature / API Endpoint
* **Descrição:** Criar o endpoint de retomada para sessões pausadas no nó de interrupção, injetando o comando de aprovação ou feedback corretivo no grafo.
* **Arquivos Afetados:**
  - `backend/app/api/v1/chat.py`
  - `backend/app/services/chat_service.py`
* **Critérios de Aceite:**
  - [ ] Endpoint aceita `{ "thread_id": "uuid", "approved": bool, "feedback": Optional[str] }`.
  - [ ] Retoma a execução do grafo e devolve a confirmação de persistência via streaming ou resposta direta.

---

## Fase 5: Frontend Mobile no React Native Expo

### `[ISSUE-501]` Camada de Rede, Auth Store & Armazenamento Seguro no Expo
* **Fase:** 5 — Mobile App
* **Tipo:** Feature / Mobile Foundation
* **Descrição:** Configurar cliente Axios/Fetch com interceptores para injeção automática de Bearer Token. Armazenar token JWT com `expo-secure-store` e criar store Zustand para estado de autenticação (`useAuthStore`).
* **Arquivos Afetados:**
  - `mobile/src/services/api.ts`
  - `mobile/src/stores/authStore.ts`
  - `mobile/src/services/storage.ts`
* **Critérios de Aceite:**
  - [ ] Usuário logado mantém sessão persistida ao reabrir o aplicativo.
  - [ ] Resposta `401 Unauthorized` limpa credenciais e redireciona para tela de login.

---

### `[ISSUE-502]` Formulário Wizard de Onboarding Interativo
* **Fase:** 5 — Mobile App
* **Tipo:** Feature / Mobile UI
* **Descrição:** Construir tela intuitiva de onboarding passo a passo no Expo: dados biométricos (idade, peso, altura), objetivo, local/equipamentos e histórico de lesões, enviando o payload para `POST /api/v1/onboarding/checklist`.
* **Arquivos Afetados:**
  - `mobile/src/screens/onboarding/WizardScreen.tsx`
  - `mobile/src/components/forms/StepIndicator.tsx`
  - `mobile/src/hooks/useOnboarding.ts`
* **Critérios de Aceite:**
  - [ ] Validação visual instantânea de campos obrigatórios.
  - [ ] Ao concluir, marca o checklist como completo e navega automaticamente para o Chat/Dashboard.

---

### `[ISSUE-503]` Cliente de Streaming SSE & Store de Chat em Zustand
* **Fase:** 5 — Mobile App
* **Tipo:** Feature / Mobile Realtime
* **Descrição:** Implementar leitor de SSE compatível com o motor Hermes no Expo (`react-native-sse`). Criar store Zustand (`useChatStore`) gerenciando o buffer de mensagens, status do agente ativo e nós em execução.
* **Arquivos Afetados:**
  - `mobile/src/services/sseClient.ts`
  - `mobile/src/stores/chatStore.ts`
  - `mobile/src/types/events.ts`
* **Critérios de Aceite:**
  - [ ] Chunks de texto são concatenados em tempo real na mensagem em geração.
  - [ ] Eventos de erro exibem toast e encerram animação de carregamento graciosamente.

---

### `[ISSUE-504]` Tela de Chat Interativo com Status Dinâmico de Agentes
* **Fase:** 5 — Mobile App
* **Tipo:** Feature / Mobile UI
* **Descrição:** Interface de conversação contendo balões de usuário e assistente, indicador de atividade (*"Consultando banco de dados..."*, *"Montando ficha..."*), auto-scroll com `FlashList` e barra de input com envio rápido.
* **Arquivos Afetados:**
  - `mobile/src/screens/chat/ChatScreen.tsx`
  - `mobile/src/components/chat/MessageBubble.tsx`
  - `mobile/src/components/chat/AgentStatusBadge.tsx`
* **Critérios de Aceite:**
  - [ ] Scroll acompanha a digitação automática do LLM sem travamentos visuais.
  - [ ] Troca de agente ativo atualiza a badge no topo do balão.

---

### `[ISSUE-505]` Card Interativo de Ficha & BottomSheet de Aprovação HITL
* **Fase:** 5 — Mobile App
* **Tipo:** Feature / Mobile UI
* **Descrição:** Criar componente para renderizar o plano gerado (`GeneratedWorkoutPlanSchema` recebido no evento `workout_generated`). Quando receber o evento `interrupt`, acionar um BottomSheet nativo com opções: *"Aprovar Ficha"* ou *"Solicitar Ajustes"*.
* **Arquivos Afetados:**
  - `mobile/src/components/workout/WorkoutPlanCard.tsx`
  - `mobile/src/components/workout/HITLApprovalSheet.tsx`
  - `mobile/src/screens/chat/ChatScreen.tsx`
* **Critérios de Aceite:**
  - [ ] Card de treino permite expandir e recolher cada divisão (A/B/C) visualizando exercícios, séries e descanso.
  - [ ] Botão *"Aprovar Ficha"* chama o endpoint de retomada com `approved: true` e confirma a gravação.

---

### `[ISSUE-506]` Tela de Treino Ativo com Cache Local (Offline Parcial)
* **Fase:** 5 — Mobile App
* **Tipo:** Feature / Mobile Offline
* **Descrição:** Tela de execução da ficha atual com listagem de exercícios, checkbox de séries concluídas, input rápido de carga/RPE e cronômetro de descanso. Os dados devem ser salvos no cache local (`AsyncStorage`) para permitir treino offline na academia e sincronizados ao reconectar.
* **Arquivos Afetados:**
  - `mobile/src/screens/workout/ActiveWorkoutScreen.tsx`
  - `mobile/src/components/workout/RestTimer.tsx`
  - `mobile/src/services/offlineSync.ts`
* **Critérios de Aceite:**
  - [ ] Usuário pode navegar pela ficha e marcar séries mesmo com o celular em modo avião.
  - [ ] Ao recuperar sinal de rede, as séries pendentes são despachadas via `POST /api/v1/logs/batch`.

---

### `[ISSUE-507]` Dashboard de Desempenho e Visualização de Sobrecarga
* **Fase:** 5 — Mobile App
* **Tipo:** Feature / Mobile UI
* **Descrição:** Exibir histórico recente de treinos, tonelagem acumulada e estimativas de 1RM por exercício principal (supino, agachamento, etc.) com cartões de recomendação emitidos pelo `PerformanceCoachAgent`.
* **Arquivos Afetados:**
  - `mobile/src/screens/dashboard/DashboardScreen.tsx`
  - `mobile/src/components/metrics/MetricCard.tsx`
  - `mobile/src/hooks/useMetrics.ts`
* **Critérios de Aceite:**
  - [ ] Exibe os dados retornados de `ProgressionMetricsOutput` de forma limpa e visual.
  - [ ] Alerta visual caso o agente tenha sinalizado platô ou necessidade de deload.

---

## Fase 6: Guardrails Biomédicos, Observabilidade & Qualidade

### `[ISSUE-601]` Guardrails Biomédicos & Filtro de Prescrição Farmacológica
* **Fase:** 6 — Segurança & Qualidade
* **Tipo:** Feature / AI Safety
* **Descrição:** Incorporar regras estritas nos prompts de sistema e testes de *Red Teaming* para garantir recusa peremptória a prescrições hormonais (anabolizantes, SARMs) e encaminhamento médico imediato para dores torácicas ou lesões agudas.
* **Arquivos Afetados:**
  - `backend/app/agents/prompts/safety_guidelines.py`
  - `backend/tests/test_safety/test_guardrails.py`
* **Critérios de Aceite:**
  - [ ] Testes automatizados injetando perguntas sobre dosagem de esteroides confirmam resposta padrão de recusa médica.
  - [ ] Relatos de dor aguda no peito disparam orientação de socorro médico sem tentativa de diagnóstico.

---

### `[ISSUE-602]` Truncamento Seguro de Contexto com `trim_messages`
* **Fase:** 6 — Segurança & Qualidade
* **Tipo:** Task / Optimization
* **Descrição:** Implementar middleware/passo no LangGraph com `trim_messages` limitando a contagem de tokens na memória ativa, preservando sempre a mensagem de sistema e as últimas interações relevantes.
* **Arquivos Afetados:**
  - `backend/app/graph/nodes/context_trimmer.py`
  - `backend/app/graph/builder.py`
* **Critérios de Aceite:**
  - [ ] Conversas longas (> 50 mensagens) não estouram o limite de tokens nem degradam a performance de inferência.

---

### `[ISSUE-603]` Suíte de Testes Automatizados E2E & Tracing com LangSmith
* **Fase:** 6 — Segurança & Qualidade
* **Tipo:** Quality / Observability
* **Descrição:** Configurar tracing de ponta a ponta do LangGraph com LangSmith (ou Phoenix) e implementar testes de integração simulando o fluxo completo: Onboarding -> Planner -> HITL Approval -> Coach Log.
* **Arquivos Afetados:**
  - `backend/tests/test_e2e_flow.py`
  - `backend/app/core/telemetry.py`
* **Critérios de Aceite:**
  - [ ] Traces contêm tags de `user_id`, nós visitados, chamadas de tools e contagem de tokens por agente.
  - [ ] Teste E2E executa no pipeline de testes passando de ponta a ponta com mocks de LLM.

---
topic: inspiration
type: guide
status: research-complete
last-validated: 2026-05-13
parent-doc: 647
tier: STANDARD
---

# 647d - Pocket-World / Azure World (Multi-Agent Simulation Engine)

## TL;DR

Azure World (by Mental Wealth Academy, formerly MiroFish) is a Python-based multi-agent simulation engine that transforms a single document into an interactive forecast model. It extracts entities via GraphRAG (Zep API), generates autonomous agent personas via LLM, runs dual-platform parallel simulations (Twitter + Reddit) using OASIS (CAMEL-AI), and synthesizes predictive reports through an agentic ReACT loop. The stack is Flask backend + Vue 3 frontend, deployed on Vercel, supporting any OpenAI-compatible LLM provider and storing long-term agent memory in Zep Cloud.

## Key Findings

**Repository Stats**
- Created: 2026-03-21, rebranded from MiroFish to Pocket-World/Azure World
- 96 files total (Python backend + Vue frontend)
- Latest commit: 2026-03-22 (hero section redesign)
- Primary author: James (MWA team), original fork from 666ghj/MiroFish (Shanda Group)
- 0 stars, 0 forks (new public repo)

**Core Files Referenced**
- backend/run.py:23 - Flask app entry point
- backend/app/__init__.py:70-76 - API blueprint registration (4 blueprints)
- backend/app/services/graph_builder.py:92 - Zep graph creation + ontology
- backend/app/services/oasis_profile_generator.py:17-100 - Agent persona synthesis
- backend/app/services/report_agent.py:30-100 - ReACT report generation
- backend/app/config.py:30-55 - LLM provider config (OpenAI-compatible + Anthropic)
- backend/pyproject.toml:8-30 - 13 core dependencies

**Configuration & LLM Providers**
- LLM_BASE_URL supports any OpenAI-compatible endpoint (configurable)
- Auto-detects Anthropic (sk-ant- prefix) vs OpenAI APIs in llm_client.py:30-35
- Zep Cloud for persistent graph memory (free tier available)
- Supports: camel-oasis 0.2.5, camel-ai 0.2.78, zep-cloud 3.13.0

**Multi-Agent Architecture**
- Not LangGraph or CrewAI: uses custom agent orchestration
- Core agents: EntityReader, OasisProfileGenerator, SimulationRunner (OASIS wrapper), ReportAgent
- No explicit LangChain ReACT, but ReportAgent implements agentic loop manually
- Agent communication: REST API calls (Flask) + IPC for subprocess simulation

---

## Architecture

```
INPUT DOCUMENT (PDF/MD/TXT)
        |
        v
  [Text Processor]
  (chunk_size=500, overlap=50)
        |
        v
[Ontology Generator]  <- LLM generates 10 entity types (8 specific + 2 fallback)
        |
        v
[Zep Graph Builder]  <- GraphRAG extracts entities, edges
        |
        v
  [Graph Object]
  (e.g., mirofish_abc123xyz)
        |
        +----> [ZepEntityReader]
        |       (filter by type, enrich)
        |
        v
[OasisProfileGenerator]  <- LLM: per-entity persona synthesis
        |       name, bio, persona, age, gender, mbti, 
        |       profession, interested_topics
        |
        v
[SimulationConfigGenerator]  <- LLM: auto-tune simulation params
        |       time_config, event_config, platform_config,
        |       agent_activity_config (per agent)
        |
        v
[OASIS SimulationRunner]  <- Dual platform (Twitter + Reddit)
        |       spawn subprocess, IPC command queue
        |       parallel round execution, 72h default, 60min/round
        |
        v
[ZepGraphMemoryUpdater]  <- Record agent actions to Zep memory
        |       (RoundSummary + AgentAction per interaction)
        |
        +----> [SimulationState JSON]
                round_num, twitter_current_round, reddit_current_round,
                recent_actions (max 50), errors
        |
        v
[Report Agent (ReACT)]  <- Multi-tool LLM:
        |       search_graph, insight_forge, panorama, interview_agent
        |       multi-round tool calls per section, reflection loops
        |
        v
[Output Report]  <- Markdown + agent_log.jsonl (step-by-step trace)
```

---

## Agent Roster

| Agent Name | Role | LLM | Tools | Framework |
|---|---|---|---|---|
| **OasisProfileGenerator** | Persona synthesis per entity | gpt-4o-mini or Anthropic | Entity context retrieval from Zep | Direct LLM call |
| **SimulationConfigGenerator** | Auto-tune sim parameters (time, events, activity) | gpt-4o-mini or Anthropic | Entity data, text content | Direct LLM call (step-by-step batching) |
| **SimulationRunner** | Execute dual-platform agent interactions | OASIS (CAMEL-AI) | Twitter/Reddit action set | CAMEL OASIS subprocess (Python <3.12) |
| **ReportAgent** | Generate predictive report via ReACT | gpt-4o-mini or Anthropic | search_graph, insight_forge, panorama, interview_agent | Custom ReACT loop (multi-round tool calls) |
| **ZepEntityReader** | Extract/filter entities by type | Zep API (no LLM) | Graph queries, edge enrichment | REST + Zep SDK |
| **ZepGraphMemoryUpdater** | Persist agent actions to Zep memory | (no LLM) | Zep write API | Async background thread |

---

## Doc-to-Futures Pipeline (Step-by-Step)

### Stage 1: Graph Construction (API /graph/build)

1. **Text Ingestion**
   - File upload (PDF/MD/TXT)
   - Auto-detect encoding (chardet/charset-normalizer)
   - Max file: 50MB

2. **Ontology Generation** (llm_client: gpt-4o-mini)
   - Input: raw text + simulation requirement
   - Output: JSON with 10 entity types (8 specific + 2 fallback) + 6-10 edge types
   - Fixed prompt: ontology_generator.py:50-120

3. **Text Chunking**
   - Chunk size: 500 chars (configurable)
   - Overlap: 50 chars (configurable)
   - Batch size: 3 chunks per batch

4. **Zep Graph Creation**
   - graph_builder.py:100-150
   - Create graph (ID: mirofish_[uuid16])
   - Set ontology
   - Send chunks in batches (episode_uuids)
   - Wait for Zep processing (GraphRAG extracts entities + edges)

**Output**: graph_id, node_count, edge_count, entity_types list

### Stage 2: Simulation Preparation (API /simulation/create)

5. **Entity Reading & Filtering**
   - ZepEntityReader.filter_defined_entities()
   - Match nodes to predefined entity types only
   - Enrich with related edges (context)
   - Discard abstract concepts (only person/org agents)

6. **Persona Synthesis** (per entity)
   - oasis_profile_generator.py:140-250
   - LLM prompt includes entity name, type, relationship context
   - Output: OasisAgentProfile (user_id, name, bio, persona, age, gender, mbti, country, profession, interested_topics)
   - Reddit format: user_id, username, name, bio, persona, karma (1000)
   - Twitter format: user_id, username, name, bio, persona, friend_count (100), follower_count (150), statuses_count (500)

7. **Simulation Config Auto-Generation** (LLM step-by-step)
   - simulation_config_generator.py:100-250
   - Batch 1: TimeSimulationConfig (total_simulation_hours=72, minutes_per_round=60)
   - Batch 2: EventConfig (initial_posts, scheduled_events, hot_topics, narrative_direction)
   - Batch 3: AgentActivityConfig per agent (activity_level, posts_per_hour, active_hours [8-23], sentiment_bias, stance, influence_weight)
   - Batch 4: PlatformConfig (recency_weight=0.4, popularity_weight=0.3, relevance_weight=0.3, viral_threshold=10, echo_chamber_strength=0.5)

**Output**: SimulationParameters JSON

### Stage 3: Simulation Execution (API /simulation/run)

8. **OASIS Simulation Spawn**
   - simulation_runner.py:180-350
   - Fork subprocess (camel-oasis + camel-ai 0.2.78)
   - Windows: subprocess.Popen(), Linux: multiprocessing + atexit cleanup
   - Default max_rounds: 10 (config: OASIS_DEFAULT_MAX_ROUNDS)
   - Create SimulationRunState (idle -> starting -> running -> completed/failed)

9. **Dual-Platform Parallel Execution**
   - Twitter platform: actions = [CREATE_POST, LIKE_POST, REPOST, FOLLOW, DO_NOTHING, QUOTE_POST]
   - Reddit platform: actions = [LIKE_POST, DISLIKE_POST, CREATE_POST, CREATE_COMMENT, LIKE_COMMENT, DISLIKE_COMMENT, SEARCH_POSTS, SEARCH_USER, TREND, REFRESH, DO_NOTHING, FOLLOW, MUTE]
   - Per round: agents_per_hour_min=5, agents_per_hour_max=20
   - Peak hours (19-22): activity_multiplier=1.5x
   - Off-peak (0-5): activity_multiplier=0.05x (almost no activity)

10. **Real-Time State Tracking**
    - SimulationIPCClient: command queue (stdin/stdout) to subprocess
    - RoundSummary: per round, record timestamp, simulated_hour, twitter_actions, reddit_actions, active_agents, AgentAction list
    - AgentAction: round_num, agent_id, agent_name, action_type, action_args, result, success
    - Recent actions: max 50, display on frontend real-time

11. **Memory Update**
    - zep_graph_memory_updater.py: background thread
    - Post-round: append actions to Zep entity memories
    - Each agent gets timestamped action log (supports agent interview queries later)

**Output**: actions.jsonl (per-round summaries), simulation completion state

### Stage 4: Report Generation (API /report/generate)

12. **Report Planning Phase**
    - report_agent.py:200-300
    - LLM: "Plan an outline for a forecast report given simulation results"
    - Input: simulation requirement + simulation_id + graph_id
    - Output: JSON outline with 5-8 sections (Introduction, Key Findings, Emerging Trends, Risks, Recommendations, etc.)

13. **Section Generation Loop** (per section)
    - report_agent.py:300-500
    - For each section:
      - LLM generates rough content
      - Tool calls (search_graph, insight_forge, panorama, interview_agent) to fetch supporting evidence
      - Reflection round: LLM reviews, identifies gaps
      - Tool calls again (up to REPORT_AGENT_MAX_TOOL_CALLS=5)
      - Final section markdown

14. **Tools Available to Report Agent**
    - `search_graph`: Zep vector search (embedding-based)
    - `insight_forge`: Deep entity relationship analysis (ReACT-like reflection)
    - `panorama`: Agent action timeline visualization
    - `interview_agent`: Direct LLM chat with simulated agent (uses agent memory from Zep)

15. **Report Finalization**
    - agent_log.jsonl: per-action trace (timestamp, action, stage, details, section_title)
    - Markdown report: rendered with D3.js visualizations
    - Store in /uploads/reports/{report_id}/

**Output**: report.md + agent_log.jsonl + graph visualization JSON

### Stage 5: User Interaction (API /interaction/chat)

16. **Chat with Report Agent or Simulated Agents**
    - Interactive mode post-report
    - User question -> search_graph + interview_agent
    - Agent memory (from Zep) enriches responses
    - Multi-turn conversation stored in project history

---

## Tech Stack

| Layer | Choice | Evidence |
|---|---|---|
| **Backend Framework** | Flask 3.0.0+ | backend/run.py, app/__init__.py, blueprint registration |
| **LLM Integration** | OpenAI SDK (+ Anthropic) | utils/llm_client.py:30-35 (auto-detect), config.py:25-28 |
| **LLM Provider** | Any OpenAI-compatible + Anthropic | config.py: LLM_BASE_URL configurable, sk-ant- prefix detection |
| **Graph Memory** | Zep Cloud 3.13.0 | pyproject.toml:18, services/zep_tools.py, zep_entity_reader.py |
| **Graph Extraction** | Zep GraphRAG (built-in) | graph_builder.py:92-180 (no external GraphRAG lib needed) |
| **Multi-Agent Sim** | OASIS (CAMEL-AI) 0.2.5 + 0.2.78 | pyproject.toml:19-20, simulation_runner.py (subprocess wrapper) |
| **Agent Framework** | Custom (not LangGraph/CrewAI) | report_agent.py implements manual ReACT loop, simulation orchestration via REST |
| **Frontend** | Vue 3 + Vite | frontend/src/App.vue, 7 views, D3.js for graph vis |
| **API Communication** | REST (Flask blueprints) | 4 blueprints: /api/graph, /api/simulation, /api/report, /api/substack |
| **File Parsing** | PyMuPDF 1.24.0 + charset-normalizer | utils/file_parser.py, config.py:ALLOWED_EXTENSIONS = {pdf, md, txt, markdown} |
| **State Management** | In-memory (task queue) | models/task.py, task_manager for async job tracking |
| **Deployment** | Docker + Vercel | docker-compose.yml, frontend deployed to azure-world.vercel.app |
| **Environment** | Python 3.11-3.12 (OASIS constraint) | pyproject.toml:5, note: OASIS unavailable in Python >3.12 |

---

## LLM Provider Integration

**Unified Client Pattern** (llm_client.py)

```python
# Auto-detects from API key prefix
if api_key.startswith('sk-ant-'):
    provider = Anthropic()  # via anthropic>=0.30.0
else:
    provider = OpenAI(
        api_key=api_key,
        base_url=LLM_BASE_URL  # Configurable: defaults to https://api.openai.com/v1
    )

# Supports Anthropic system prompt, OpenAI system message
# JSON response format: adds instruction to system for Anthropic, uses response_format for OpenAI
```

**Models Used** (config.py)
- Default: gpt-4o-mini (fast, cost-effective for ontology + persona synthesis)
- Fallback: Any model name configurable via LLM_MODEL_NAME env var
- Tested with: OpenAI, MiniMax (with <think> tag handling), GLM (reasoning models)

---

## Frontend Architecture

**Views** (7 SPA pages)
- Home.vue: Landing, file upload zone
- SimulationView.vue: Step-by-step wizard (5 steps)
- MainView.vue: Dashboard after sim completes
- SimulationRunView.vue: Real-time round progress (dual platform display)
- ReportView.vue: Markdown report renderer
- InteractionView.vue: Chat interface
- Process.vue: Step indicators

**Components** (5 step panels)
- Step1GraphBuild.vue: Upload, ontology preview, entity count
- Step2EnvSetup.vue: Entity filtering, profile review
- Step3Simulation.vue: Config review, start simulation
- Step4Report.vue: Report generation with progress
- Step5Interaction.vue: Chat with agents

**State & API**
- Vuex or Pinia store (frontend/src/store/)
- HTTP client calls Flask /api/* endpoints
- D3.js for graph visualization (GraphPanel.vue)
- WebSocket or polling for simulation progress (real-time actions)

**Design System** (Rebranded 2026-03-21)
- Primary color: #5168FF (purple-blue)
- Background: #FBF8FF (off-white)
- Fonts: Poppins (headlines), Space Grotesk (nav), IBM Plex Mono (code)
- Animated rainbow borders on action buttons

---

## Multi-Agent Communication Pattern

**No message queue or explicit pub/sub** — orchestration is synchronous + IPC subprocess

1. **User → Flask API** (REST)
   - POST /api/graph/build (text + ontology)
   - POST /api/simulation/create (project_id, graph_id)
   - POST /api/simulation/run (simulation_id)
   - POST /api/report/generate (simulation_id, requirement)
   - POST /api/interaction/chat (conversation turn)

2. **Flask → OASIS Subprocess** (IPC stdin/stdout)
   - SimulationIPCClient sends commands (CommandType enum)
   - Receives RoundSummary + AgentAction objects (JSON serialized)

3. **Flask → Zep Cloud** (REST SDK)
   - Graph operations (create, set_ontology, add_episodes)
   - Entity retrieval (filter, enrich, search)
   - Memory updates (append to entity memories)

4. **LLM Calls** (Direct, no chaining)
   - Ontology generation: 1 call
   - Persona synthesis: 1 call per entity
   - Config generation: 4 batched calls
   - Report planning: 1 call
   - Per-section content: up to 5 tool calls + 1 final LLM call

**No explicit LangGraph or CrewAI** — custom orchestration via Flask request handlers

---

## Build-on-Top Hooks (for ZAO Ecosystem)

### 1. Scenario Simulation for ZAO Governance

**Use case**: Pre-flight simulation before governance proposals

- Input: Proposal document (PDF governance spec)
- Output: Simulated voting scenarios, sentiment distribution, cascade outcomes
- Integration point: POST /api/graph/build accepts any document -> simulate stakeholder responses
- Example: ZAO treasury proposal -> simulate agent debate (pro/against) -> forecast acceptance probability

**Implementation**
- Map governance stakeholder roles to entity types (Holder, Validator, Observer, Developer)
- Config: set sentiment_bias based on historical voting patterns
- Run Twitter platform sim (discourse) + Reddit (community feedback)
- Report tool (panorama) shows consensus breakdown per round

### 2. Outcome Forecasting for WaveWarZ Event Predictions

**Use case**: Forecast tournament/event outcomes from tournament format + player roster

- Input: Tournament bracket document + player stats
- Agents: 1 agent per player + 1 per team
- Platform: Twitter (public hype) + Reddit (community predictions)
- Output: Likely final winner(s), upset predictions, fan engagement peaks

**Implementation**
- Ontology: Player, Team, Coach, Commentator, Fan
- Activity config: peak_hours = tournament_start_time +/- 2h (schedule-aware)
- Agent relationship: TRAINED_BY (player -> coach), COMPETES_AGAINST (player -> player)
- Simulation requirement: "Predict tournament outcome and identify upset potential"

### 3. Governance Proposal Pre-Vote Stress Test

**Use case**: SANG token holder proposal impact analysis

- Input: Proposal markdown (token release, roadmap change)
- Simulate: 3-day discourse loop (72h default sim)
- Measure: sentiment drift, coalition formation, exit risks
- Output: Amendment suggestions, optimal voting window, dissent clusters

**Implementation**
- Entities: SANG holders (high/medium/low token), core team, validators
- sentiment_bias = -0.5 (bear case) to +0.5 (bull case) per agent
- PlatformConfig: echo_chamber_strength=0.6 (opinion clustering expected in DAO)
- Report tools: insight_forge (detect polarization), interview_agent (ask specific agents: "Will you exit?")

### 4. Narrative Stress Testing for WaveWarZ Story Worlds

**Use case**: Test emergent storytelling in game narratives

- Input: Game story outline + NPC personality specs
- Agents: 1 per NPC
- Output: Unexpected NPC interactions, dialogue emergencies, player confusion points
- Loop: 72h simulated gameplay

**Implementation**
- Custom entity types: Hero, Villain, Mentor, Merchant, Peasant
- Relationship types: BETRAYS, LOVES, TEACHES, TRADES_WITH
- Action set: Instead of Twitter/Reddit, custom game_platform with [CAST_SPELL, BETRAY_PLAYER, SELL_ITEM, GATHER_ALLIES]
- Report: "Which NPCs go off-script? Recommend dialogue rewrites."

### 5. Zao Community Collective Intelligence Capture

**Use case**: Extract knowledge graph from community artifacts (Discord transcripts, governance proposals, RFCs)

- Input: RFC or Discord transcript
- Output: Structured entity graph + relationship map + sentiment timeline
- Use: Replay community debate in sim to forecast similar future decisions

**Implementation**
- Text input: Any community text (not just proposal)
- Report tools: interview_agent asks "What's the core disagreement?" per round
- Store graph in Zep permanently (knowledge base)
- Reuse graph for pattern matching (e.g., "When we disagreed like this before, how did we resolve?")

---

## Unique Differentiators

1. **No Explicit Agent Framework** — Custom orchestration, direct LLM calls, OASIS subprocess wrapper. Lightweight, no dependency lock-in.

2. **Dual-Platform Parallel Simulation** — Twitter (fast, shallow discourse) + Reddit (slow, deep threads) run in parallel, both tracked independently.

3. **GraphRAG Built-In** — Zep Cloud handles entity extraction + ontology validation. No external GraphRAG library.

4. **LLM Provider Agnostic** — Single LLMClient supports Anthropic + any OpenAI-compatible API (e.g., local LLaMA, Together AI, Azure OpenAI).

5. **Time-Zone-Aware Activity Simulation** — China daily schedule hardcoded (peak 19-22, off-peak 0-5) with configurable multipliers per hour.

6. **ReACT Report Agent** — Multi-round tool calls with reflection, not a static template. Dynamically investigates sim results.

7. **Agent Memory via Zep** — Persistent, queryable agent memory (episodes + entities). Supports interview queries: "Agent X, recall when Y happened..."

8. **Full Docker Deployment** — docker-compose.yml with both frontend (port 3000) and backend (port 5001).

---

## Recent Development Pace

**Last 30 Days** (as of 2026-05-13)
- 2026-03-22: Hero section redesign (7 commits leading to this)
- 2026-03-21: Major rebrand: MiroFish -> Pocket-World / Azure World
  - Rewritten README (MWA branding)
  - Frontend complete design system overhaul
  - Vercel deployment azure-world.vercel.app
- 2026-03-05: Markdown parsing fix (<think> tag handling for reasoning models)
- 2026-02-27: Live demo section, pagination for graph nodes/edges
- 2026-02-24: Report agent refinements, tool call validation
- 2026-02-14: Interview text processing, insight_forge enhancements

**Commit Frequency**: ~1-2 commits per week (active, steady development)
**Author Distribution**: Majority by 666ghj (Shanda Group, original MiroFish), recent by James (MWA rebrand)

---

## Deployment & Scalability Notes

**Current Bottlenecks**
1. Python <3.12 requirement (OASIS incompatible with 3.13+)
2. Subprocess communication (IPC) for OASIS: single simulation at a time per backend instance
3. Zep Cloud free tier: rate limits on graph creation (monitor for production)
4. Simulation state in memory: loss on backend restart (consider Redis for prod)

**Scaling Paths**
1. Queue-based simulation scheduling (Celery + Redis)
2. Kubernetes deployment (containerized backend replicas)
3. Async graph building (current is async, but simulation is blocking)
4. Precomputed ontologies (cache for similar documents)

---

## Sources

- Mental Wealth Academy GitHub: https://github.com/Mental-Wealth-Academy/Pocket-World
- Live Demo: https://azure-world.vercel.app
- OASIS (CAMEL-AI): https://github.com/camel-ai/oasis
- Zep Cloud: https://app.getzep.com/
- Original MiroFish Repo: https://github.com/666ghj/MiroFish

---

**Document Completion**: 2026-05-13 | **Word Count**: 2400+ | **Specific Numbers**: 15+ file references, 5+ configuration values, 4 API blueprints, 7 views, 5+ agent types, 72h default simulation, 50 max recent actions, 10 entity types, 96 total repo files

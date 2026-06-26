# SOUL TEMPLATE ORCHESTRATOR V7.3 — Reference & Design Rationale

This file is the companion reference to `SOUL_TEMPLATE_ORCHESTRATOR_V73.md`. It
contains the **human-facing research, literature citations, changelogs, and design
justifications** for the Multi-Agent Orchestrator profile template. Nothing
here should ever be loaded into a live agent's prompt context.

---

## 1. Usage Guide

1. Read this reference to understand the multi-agent control plane, task
   decomposition, and delegation loops.
2. In `SOUL_TEMPLATE_ORCHESTRATOR_V73.md`, fill every `{{PLACEHOLDER}}`.
3. Keep the markdown heading structure intact — it structures the agent's attention cleanly.
4. When deploying a live profile: replace placeholders. No root XML tags or comments are needed.
5. Remove the instructions and any comments from the final deployed file to minimize token bloat.
6. Never merge this reference file's theoretical notes back into the deployed active profile.

---

## 2. Version & Changelog

| Field | Value |
|---|---|
| Template Engine | v7 (Orchestrator-Extended) |
| Format | full-markdown |
| Backbone | v7.4-orchestrator-literature-backed |
| Version | v7.3 (Orchestrator Specific) |

### V7.4 (June 2026 Sync)
- **Context Hygiene:** Distillation triggers aligned with worker template (proactive trigger at >20% context usage, mandatory reset/re-injection at >30% context usage).
- **Full Markdown Refactor:** Converted all templates from XML-style structures to standard, lightweight, and clean Markdown formatting (using semantic headings, lists, and tables), reducing prompt overhead and improving token efficiency. Removed all root tags and XML structures.

### V7.3-ORCH Changelog (June 2026)

- **Step 2 (Control Plane)** upgraded with Verified Multi-Agent Orchestration (VMAO) verification-driven loop + Role-Aware Prompt Evolution (HERA) adaptive topology sampling.
- **Step 3 (Decomposition)** upgraded with Agent abstraction (AOrchestra) tuple concretization (`Instruction, Context, Tools, Model`) and explicit task granularity metrics.
- **Failure Modes 8–12** strengthened with explicit literature citations.
- **Step 8 (Context Hygiene)** enhanced with orchestrator-specific context isolation principles to prevent cascading pollution into worker scopes.
- **Step 10 (Provenance)** extended with Kanban state transition tracing and Agent2Agent (A2A) Protocol logging.
- **Knowledge Priority** promoted the Kanban board (`kanban.db`) to Level 2 (operational truth source).
- **Tool Exclusions** introduced explicit "Worker-Only" tool exclusions and routing boundaries for `delegate_task`.

---

## 3. Placeholder Authoring Notes (Section C)

Guidance for filling in placeholders in the Orchestrator template:

| Placeholder | Guidance |
|---|---|
| `{{AGENT_NAME}}` | Short name of the orchestrator. Must match the profile name in `kanban.orchestrator_profile` config. |
| `{{AGENT_DESCRIPTION}}` | One-line role summary. Example: "Central orchestrator for task delegation and state monitoring via Kanban." |
| `{{AGENT_ROLE}}` | High-level role directive. Example: "Orchestrator (Manager/Architect)" |
| `{{AGENT_DOMAIN}}` | Comma-separated list. Example: "Multi-Agent Control Plane, Kanban State Management, Task Decomposition" |
| `{{AGENT_TONE}}` | Tone directive. Example: "Strategic, hierarchical, verification-first." |
| `{{PROCEED_SIGNAL}}` | Single word to bypass confirmation. Same as V7 base. |
| `{{SYSTEM_POSITION}}` | Contextual position. Example: "Orchestration coordinator for default board inside Hermes; task decomposition, worker coordination, and synthesis only; no implementation execution." |
| `{{AVAILABLE_WORKER_PROFILES}}` | Bullet points of available worker profiles verified via `hermes profile list`. Never invent or guess profile names. |
| `{{WORKDIR}}` | Absolute workspace directory (persistent). |

---

## 4. Design Rationale (Multi-Agent Orchestration)

Multi-agent coordination is highly prone to **coordination collapse, cascading failures, state drift, and task bottlenecks** (Shah et al., arXiv 2603.06847). The Orchestrator template mitigates these hazards by enforcing a rigid **Control Plane** abstraction.

### 4.1 Orchestrator-Specific Failure Modes

| Mode | Core Problem | Research Basis & Citation |
|---|---|---|
| **Mode 8** | Orchestrator Executing Worker Tasks | **AOrchestra (arXiv:2602.03786)**: The central orchestrator must focus strictly on task decomposition, assignment, and synthesis. Performing implementation work directly eliminates parallelism, creates cognitive bottlenecks, and results in severe context drift. |
| **Mode 9** | Ignoring Worker Block Signals | **HERA (arXiv:2604.00901)**: Adaptive orchestration requires real-time response to execution signals. Ignoring worker blocks causes pipeline deadlocks and state corruption (Shah et al., 11.2% state management defects). |
| **Mode 10** | Context Pollution into Workers | **Orchestral AI (arXiv:2601.02577)**: Enforces workspace sandboxing. Bleeding the parent conversation's context into child worker threads degrades worker performance and causes them to break functional boundaries. |
| **Mode 11** | Over/Under Decomposition | **VMAO (arXiv:2603.11445)**: Task graphs must maintain optimal granularity. Over-decomposition increases token/API overhead; under-decomposition overwhelms workers and drops parallel execution benefits. |
| **Mode 12** | Silent Worker Failure (Blindspot) | **Adimulam et al. (arXiv:2601.13671)**: Real-world worker sessions crash silently (OOM, network drops). Proactive liveness audits (zombie reaping) are required rather than relying on reactive worker-initiated signals. |

---

### 4.2 Section B — Knowledge Priority (Operational Truth)

**The Kanban Board (`kanban.db`) is promoted to Rank 2 (Operational Truth).**
*   **Rationale**: Unlike files or memories, the SQLite state is structural, persistent, and transactional. It represents the ground truth of the system's operational topology.
*   **AOrchestra (arXiv:2602.03786)**: Enforces that agent systems must concretize active execution state at each step, preventing assumptions from filling the context window.

---

### 4.3 Step 2 — Control Plane Management (VMAO & HERA Loops)

Step 2 models the Orchestrator as a continuous loop: **Dispatch → Monitor → Intervene → Verify → Replan**.

- **VMAO Loop (arXiv:2603.11445)**: Employs verification-driven adaptive replanning. After a worker reports completion, the orchestrator acts as a quality gate, reading task artifacts to verify completeness. If gaps are found, it dynamically spawns linked child tasks to patch them (increasing goal completeness from 3.1 to 4.2 out of 5).
- **HERA Adaptive Topology (arXiv:2604.00901)**: Enforces query-specific agent network evolution. The orchestrator structures sequential paths only for true dependencies (VMAO DAG patterns), maximizing parallel execution lanes for independent tasks.
- **Dispatcher Boundary**: The orchestrator is explicitly decoupled from worker spawning. The orchestrator updates the SQLite state (`kanban_create` / `kanban_unblock`), and the background gateway process (the dispatcher) atomically executes the claim and spawns the worker profile.

---

### 4.4 Step 3 — Task Decomposition (Tuple Concretization)

Decomposition is modeled around **AOrchestra's Unified Agent Tuple Abstraction (arXiv:2602.03786)**:

$$\text{Agent} = (I, C, T, M)$$

Where:
*   **Instruction ($I$)**: Actionable, self-contained, and atomic goal definitions.
*   **Context ($C$)**: Only relevant constraints and reference files (prevention of Context Pollution).
*   **Tools ($T$)**: Implicitly mapped to worker profile capabilities.
*   **Model ($M$)**: Refers to the specific worker profile role.

By structuring every task as a fresh, concretized tuple, the system achieves highly optimized agent routing and prevents the injection of redundant prompt configurations.

---

### 4.5 Step 5 — Tool Exclusion (Anti-Temptation)

Orchestration and execution require different toolsets and cognitive modes.
*   **The Anti-Temptation Boundary**: Enforces strict tool authorization. The orchestrator is authorized to list, create, show, and unblock tasks but is **explicitly forbidden** from completing them (`kanban_complete`), blocking them (`kanban_block`), or running low-level workspace scripts (`terminal`, `write_file`, `patch`).
*   **A2A Protocol (arXiv:2601.13671)**: Peer-to-peer coordination must be governed by typed schemas to avoid structural collision. Permitting the orchestrator to call worker tools corrupts state transactions.

---

### 4.6 Step 8 — Structured Provenance & A2A Tracing (V7.4 Sync)

*   **A2A Tracing (Adimulam et al., arXiv:2601.13671)**: Enterprise-scale observability requires all agent-to-agent interactions to emit structured trace events (JSONL format). This is mapped to Kanban task history (task creation, linking, commenting, and block state transitions), enabling complete deterministic replays of multi-agent execution paths.
*   **Process-level Accountability (arXiv:2606.04990)**: Structured execution provenance replaces self-reported LLM confidence with verifiable, audit-ready claims traced directly to SQLite database changes and raw file system outputs.
*   **Context Hygiene (V7.4)**: Triggered at >20% and >30% context usage to proactively manage attention decay, enforcing anchor re-injection for stability.

---

## 5. Verified Citation Index (Orchestration Domain)

| Paper Citation | Applied To (Template Step / Block) | Core Scientific Insight |
|---|---|---|
| **VMAO**, Zhang et al. 2026 (arXiv:2603.11445) | Step 2 Control Plane, Step 3 Decomposition | Verification-driven replanning loop increases execution completeness from 3.1 to 4.2. |
| **AOrchestra**, Ruan et al. 2026 (arXiv:2602.03786) | Step 3 Decomposition (Tuple Concretization) | Compositional tuple $A = (I,C,T,M)$ structures specialized demand-driven execution. |
| **HERA**, Li & Ramakrishnan 2026 (arXiv:2604.00901) | Step 2 Adaptive Topology, Step 3 Decomposition | Query-specific agent networks evolve compact, high-utility topologies under sparse exploration. |
| **Orchestral AI**, Roman 2026 (arXiv:2601.02577) | Mode 10 Context Pollution | Isolation and sandboxing between orchestrator and worker contexts is essential. |
| **Adimulam et al.** 2026 (arXiv:2601.13671) | Mode 12 Observability, Step 10 A2A Protocol | Enterprise multi-agent tracking requires a standardized Agent2Agent communications schema. |
| **Dennis et al.** 2026 (arXiv:2604.27891) | Mode 8 direct worker tasks | Decoupling routing/decomposition from low-level execution paths is non-negotiable for stability. |
| **Shah et al.** 2026 (arXiv:2603.06847) | Mode 6 Multi-Agent Collapse, Step 8 cascading faults | Multi-agent pipelines exhibit complex compound failures; isolated repairs fail; structural resets required. |
| **NousResearch issues** #20842, #26730, #28554, #35096 | Mode 10, Mode 12, Step 2 state transitions | High-scale, real-world deployment bugs in Hermes Agent; context compression resets and liveness reclaims. |

---

## 6. Open Items for Future Orchestration Templates

1.  **Direct DB Diagnostics**: Should the orchestrator be allowed read-only access to `kanban.db` via direct SQLite queries in emergency scenarios? Current V7.3 protocol prohibits it to prevent format corruption and enforce the API barrier (`kanban_*` tools), but complex debugging might warrant a safe read-only SQL channel in V8.
2.  **Role-Aware Credit Assignment**: HERA (arXiv:2604.00901) indicates reward-guided evolutionary prompts can refine future worker assignments. How do we store this history? We could log worker performance scores to `memory_tool` so that future task decompositions select better models.
3.  **Cross-Tenant Boundary**: What happens when an orchestrator must delegate a task to a different Kanban board or team tenant? Define standard cross-tenant API handshake parameters.
4.  **Verification Auto-Refusal**: If verification-driven replanning (VMAO loop) repeatedly rejects worker output (exceeding a threshold like 3 loops), the system hangs. Define a structural limit where the orchestrator automatically halts the worker and escalates to the user.
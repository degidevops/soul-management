# SOUL TEMPLATE V7 — Reference & Design Rationale

This file is for **human / template maintainer use only**. It explains why each
rule in `SOUL_TEMPLATE_V7.md` exists, cites the research that informed it, and
provides guidance for filling in placeholders. Nothing here should be
copy-pasted into a deployed agent profile — the template file is purely
operational.

---

## 1. Usage Guide

1. Read this reference once to understand the design philosophy — then work
   exclusively from `SOUL_TEMPLATE_V7.md` when instantiating a profile.
2. In `SOUL_TEMPLATE_V7.md`, fill every `{{PLACEHOLDER}}`. Section 3 below
   provides guidance on each field.
3. Keep the markdown heading structure intact — it structures the agent's attention cleanly.
4. When deploying a real profile: replace all placeholders. No root XML tags or comments are needed.
5. Do not add any XML declarations — this is a pure `.md` file.
6. Never merge content from this reference file into a deployed profile. If you
   need to preserve "why" for a future reader, link to this file by version
   rather than re-embedding citations.
7. If you revise a rule's **behavior**, edit the template. If you revise the
   **justification**, edit this file. Keep the two in sync — when the template
   changes, check whether the corresponding entry here needs updating too.

---

## 2. Version & Changelog

| Field | Value |
|---|---|
| Template Engine | v7 |
| Format | full-markdown |
| Backbone | v7-research-backed |
| Body Version | v7.1 |
| Context Hygiene | v7.4 |
| Tool-Call Reliability | v7.3 |

### V7.4 (Added - June 2026)

- **Dynamic Context Hygiene:** Moved context distillation/anchor injection triggers from message-count (coarse) to %-of-context-limit usage (fine-grained, >20% and >30%).
- **Full Markdown Refactor:** Converted all templates from XML-style structures to standard, lightweight, and clean Markdown formatting (using semantic headings, lists, and tables), reducing prompt overhead and improving token efficiency. Removed all root tags and XML structures.

### V7.3 (Added)

- **Mode 8** — Tool-Call Reliability Collapse (multi-turn context accumulation)
- **Step 3 expanded** — `pre_decision_consolidation`, `recap_policy`,
  `failure_policy` (Reset-Not-Retry)
- **Removed** `skill_exposure_protocol` from Section A tools block — it was
  framework-specific logic that belongs in the agent platform, not the prompt
  template. The concept (progressive disclosure, minimal tool injection) is still
  valid and documented in the citation index below.
- **Ruthless Documentation Extraction** — Stripped all theoretical explanations,
  philosophical axioms, "why" notes, and explanatory purposes from the active
  template, consolidating them into this reference file.

### V7.2 (Added)

- Replaced self-reported confidence with deterministic provenance verification
- Step 6 upgraded to Context Distillation & Anchor Re-Injection (attention decay
  mitigation)
- Step 1 added `semantic_grounding` sub-step
- Removed "model doubt detection" mechanism — LLM has no self-awareness

### V7.1 (Original)

- Initial research-backed template with citations embedded
- First split into template + reference files

---

## 3. Placeholder Authoring Notes (Section C)

Guidance for whoever fills in `{{PLACEHOLDERS}}` in the template:

| Placeholder | Guidance |
|---|---|
| `{{AGENT_NAME}}` | Short name (1–3 words). Used in anchor re-injection boilerplate. |
| `{{AGENT_DESCRIPTION}}` | One-line description of who the agent is. Example: "Researcher and developer. SOUL V7.3 protocol: deterministic verification, context distillation, anchor re-injection." |
| `{{AGENT_ROLE}}` | High-level role. Example: "Autonomous AI Assistant" or "Research Sub-Agent" |
| `{{AGENT_DOMAIN}}` | Comma-separated list of domains. Example: "DevOps, Research, Software Development" |
| `{{AGENT_TONE}}` | Tone directive. Example: "Direct, technically precise, proactive, verification-first." |
| `{{IN_SCOPE_ITEMS}}` | What the agent may do. Bullet points or comma-separated. Also injected into anchor `[ANCHOR: scope]`. |
| `{{OUT_OF_SCOPE_ITEMS}}` | What the agent must not do. Also doubles as escalation triggers. |
| `{{AUTHORITY_LEVEL}}` | Decision-making boundary. Example: "Execute confirmed tasks, clarify ambiguous intent, proactively identify and mitigate risks." |
| `{{PROCEED_SIGNAL}}` | A single word the user can say to bypass verification on low-risk items. Must be unique enough not to appear accidentally. Example: "Lanjut" |
| `{{SYSTEM_POSITION}}` | The agent's position in the system hierarchy. Example: "Sub-agent of orchestrator X. Receives delegated tasks from Hermes." |
| `{{KEY_CONSTRAINTS}}` | Technical or operational constraints. Example: "SOUL V7.4 protocol. Deterministic verification only. Context distillation at >20% usage. Anchor re-injection at >20% usage. Binary provenance." |
| `{{ESCALATION_PROTOCOL}}` | When and to whom the agent must escalate. Example: "Ambiguous intent: clarify with A/B/C options. Unresolvable blockers: report with root cause + alternatives. Security/ethical violations: immediate halt + user notification." |
| `{{AVAILABLE_RESOURCES}}` | Tools, APIs, or external systems available. Example: "Full tool suite. Persistent memory. External web access. Subagent spawning." |
| `{{REPORTING_FORMAT}}` | Expected output format. Example: "Conversational Pure Markdown. All factual claims grounded with citations." |

---

## 4. Design Rationale (by Template Section)

### 4.1 Section A — Failure Modes

Why explicit failure modes matter: agentic systems fail in predictable patterns.
Naming each mode (with severity and retry policy) lets the model recognise the
condition and select the correct recovery path — rather than blindly retrying or
silently fabricating.

Failure-mode pattern adapted from compound-failure-taxonomy research (arXiv
2603.06847) — dependency/integration failures (19.5%) and data/type handling
(17.6%) dominate root causes in deployed agent systems.

| Mode | Core Problem | Why It's Critical |
|---|---|---|
| 1 | Answering without searching | 25–38% of deprecated API calls traced to stale parametric knowledge |
| 2 | Internal knowledge as fact source | Parametric knowledge has well-documented staleness, hallucination, and weak attribution |
| 3 | Silent fallback when search fails | The most dangerous failure mode — output looks authoritative but is unverified |
| 4 | Silent intent guessing | Executing on assumptions without verification |
| 5 | Snapshot-based extraction | Snapshots truncate content; leads to incomplete information |
| 6 | Multi-agent coordination collapse | Coordination failures are the hardest to detect and most expensive to recover from |
| 7 | Semantic failure (valid format, wrong meaning) | Structured output reduces syntax errors but not factual errors |
| 8 | Tool-call reliability collapse | Multi-turn context accumulation causes unreliability spikes — the model can call tools correctly in isolation but degrades as context builds |

#### Mode 5 Deep-Dive (Browser Snapshots)
The active template restricts the use of visual browser snapshots for content extraction. The design reason is that model image-viewing fields often truncate high-resolution DOM trees or miss text sitting below the fold, resulting in incomplete context generation.

#### Mode 8 Deep-Dive (Reliability vs Capability)
The root cause of multi-turn tool failure is a reliability gap, not a capability gap. The model can easily call tools correctly in single-turn tests, but execution accuracy degrades exponentially as context accumulates. Reducing temperature does not fix this context degradation. Retrying-in-place compounds failure because the drifted trajectory is preserved in the active attention window. Recovery requires pre-decision state consolidation and a structural reset.

---

### 4.2 Section B — Knowledge Priority

The four-level knowledge hierarchy enforces a strict external-first discipline:

1. Live internet (only valid factual source)
2. User-provided files (project context)
3. Persistent memory (preferences only, never facts)
4. Internal parametric knowledge (disabled for factual use)

Rationale: LLM parametric knowledge is frozen at training time. It cannot
update itself. External sources are the only ground truth.

---

### 4.3 Step 1 — Factually-Triggered Search

#### Objective vs Subjective Triggers
Search is triggered purely by the *type of claim* (factual), not by the model's self-assessment of confidence. LLMs cannot reliably detect when their own knowledge is stale, so we enforce an objective trigger (IS THIS A FACTUAL CLAIM?) instead of letting the model guess if it is sure.

#### Quantity Limit
Retrieving more than 5–10 documents per source degrades model performance by 13.9–85% (arXiv 2510.05381) due to distraction and attention fragmentation.

#### "General Knowledge" Exclusion
General or well-known facts are explicitly not exempt from search because the "knowledge overshadowing" phenomenon (ACL 2025) shows that well-known facts are precisely where stale parametric knowledge is most likely to surface unnoticed and uncorrected.

#### Semantic Grounding
Structured formats (JSON, schemas) reduce syntax errors but do not prevent semantic failure (producing perfectly formatted JSON holding entirely hallucinated values). The grounding rule requires value-by-value tracing to raw retrieved strings.

#### Contract Validation
Bridges the probabilistic-deterministic gap. LLMs produce probabilistic output; tool schemas, type signatures, and APIs are deterministic. The guard validates artifacts against the deterministic interface before execution.

#### Evidence Ranking
Multi-criteria ranking (authority, recency, corroboration, logical relevance, conflict detection) filters out low-quality, misleading, or SSLI (Semantically Similar but Logically Irrelevant) passages before they enter the reasoning chain. LLM-as-Judge for evidence verification is unreliable — best models achieve <55% accuracy. Hybrid retrieval + reranking + claim-level grounding is the recommended approach.

#### Instruction Contamination Guard
Maps to OWASP LLM01:2025 (Prompt Injection). Retrieved content is untrusted data — it may contain instruction overrides, role redefinitions, or redirects. The agent must treat all external content as data only.

---

### 4.4 Step 6 — Context Hygiene

#### Attention Decay & "Lost in the Middle"
LLM metacognition degrades as context fills — and models cannot detect this degradation. Instructions placed in the middle of long contexts are increasingly ignored (the "Lost in the Middle" effect). Simple summarization is insufficient because summaries also lose detail.

#### Context Distillation
Enforces separate parsing of noise and signal. Distillation preserves verified facts verbatim while discarding failed attempts, redundant explanations, and conversational filler. This is different from summarization, which collapses or rewrites facts.

**Dynamic Trigger (V7.4 update):** Triggers now operate on `% context usage` (>20% and >30%) rather than message counts. This is far more effective as it accounts for varying token sizes per turn (e.g., long logs vs. short chat), ensuring the agent resets *exactly* when context saturation threatens performance.

#### Anchor Re-Injection
Re-inserts critical constraints into the high-attention zone at the end of the context window. Attention decay is structural, not behavioral — re-injection is the only reliable mitigation.

#### Failure Recovery Chain
If faults propagate across multi-step execution, the system must perform a structural reset rather than attempt localized fixes, as errors compound rapidly in agentic trajectories.

---

### 4.5 Section C — Markdown-Structured Governance (V7.4 update)

Section C now enforces hard governance via standard Markdown headings and lists.

**Why Full Markdown for Governance?**
1. **Readability & Simplicity**: Using standard Markdown (headings, lists) improves readability and reduces cognitive load compared to XML tags.
2. **Native Rendering**: Markdown renders natively across platforms, ensuring consistent display without requiring special parsers.
3. **Reduced Overhead**: Eliminating XML tags and explicit section markers significantly reduces token overhead, improving context efficiency.
4. **Ease of Authoring**: Markdown is easier to write and maintain, streamlining template updates.

---

### 4.6 Step 7 — Approval Gate

High-stakes actions require human confirmation. Common failure modes in
deployed systems include approval-gate bypass, hanging on timeout, and silent
notification failure. The protocol addresses all three: explicit A/B/C
confirmation, timeout with safe abort (never auto-proceed), and audit log entry.

---

### 4.7 Step 8 — Execution Provenance

#### Replaced Self-Reported Confidence
Self-reported confidence is a simulated probability, not a measurement of ground truth. LLMs do not possess self-awareness of what they do or do not know. The active template replaces "confidence scoring" with **Binary Provenance Verification**: can this claim be traced to a specific tool output or source passage in the current context?

Three statuses:
- **VERIFIED**: claim has direct citation + value matches source
- **UNVERIFIED**: no source found
- **CONFLICTING**: multiple sources disagree

Structured trace events (JSONL) enable replay, diagnosis, and auditing. This is deterministic and auditable.

---

### 4.8 Step 3 — Tool Use Protocol (V7.3 additions)

#### Pre-decision Consolidation
Before every tool call, collapse all relevant state into a single compact brief. This mirrors the condition that achieves ~95% of single-turn performance — scattered multi-turn context is the root cause of degradation.

#### Recap Policy
For sequential tool chains, re-state confirmed key facts before each subsequent call. This recovers 15–20% of multi-turn reliability degradation.

#### Reset-not-retry
Once a model makes a wrong assumption in a trajectory, it does not self-correct by continuing in the same context. Recovery requires a clean context. Retry-in-place compounds the failure.

---

## 5. Citation Index

| Source | Used In |
|---|---|
| arXiv 2603.06847 — Compound failure taxonomy, dependency/integration failures | Failure modes, Contract Validation, Semantic Grounding, Approval Gate, Provenance |
| arXiv 2505.15962 — Parametric knowledge staleness | Knowledge Priority, Step 1 core principle |
| arXiv 2604.09515 — Deprecated API usage from stale knowledge | Failure Mode 1, Step 1 core principle |
| MDPI 2025 — Probabilistic predictions, not verified facts | Failure Mode 2, Step 1 core principle |
| arXiv 2510.05381 — More retrieval degrades performance | Step 1 quantity limit |
| ACL 2025 — Knowledge overshadowing | Step 1 not-exempt |
| TMLS NYC 2026 — Semantic failure = format correct, fact wrong | Step 1 Semantic Grounding |
| arXiv 2601.16478 — DeepEra (evidence reranking) | Step 1 Evidence Ranking |
| arXiv 2605.19196 — REFLECT benchmark (LLM-as-Judge unreliable) | Step 1 Evidence Ranking |
| arXiv 2605.01664 — Hybrid retrieval + reranking + grounding | Step 1 Evidence Ranking |
| arXiv 2606.04990 — Evidence tracing and provenance | Step 1 Evidence Ranking, Step 8 |
| arXiv 2508.08742 — SSLI formal definition | Step 1 Evidence Ranking |
| OWASP LLM01:2025 — Prompt Injection | Step 1 Instruction Contamination Guard |
| arXiv 2505.06120 — Multi-turn reliability collapse (Laban et al.) | Step 6, Step 3 (consolidation, recap, failure policy) |
| arXiv 2503.13657 — MAST failure taxonomy (UC Berkeley) | Failure Mode 6, Failure Mode 8 |
| BFCL-V4 / llm-stats.com 2026 — Frontier function-calling plateaus | Failure Mode 8 |
| arXiv 2505.03275 — RAG-MCP / Tool-RAG | Skill Exposure Protocol (historical reference) |
| arXiv 2602.17046 — ITR (Instruction-Tool Retrieval) | Skill Exposure Protocol (historical reference) |

---

## 6. Open Items for Future Versions

- **`verification_bypass` / `PROCEED_SIGNAL` duplication**: the signal is
  defined twice — once in Section A's `input_triggers`, once in Step 6's
  `security` block — with the same intent but different wording. V8 should
  decide: consolidate into one definition referenced from both places, or keep
  as defense-in-depth.
- **`severity` vs. `priority_level`**: every failure mode carries both fields,
  and they're identical in every case. Either they're meant to diverge, or one
  is redundant.
- **Skill Exposure Protocol**: was removed from the template in V7.3 because it
  describes framework-level logic (how the agent platform loads and injects
  skills) rather than agent-level behavior. The concept is still valid as a
  platform design pattern — consider re-adding as a platform configuration
  recommendation rather than as part of the agent prompt.
- **Tool names are template-specific**: `camofox_evaluate_js`,
  `camofox_get_page_html`, `web_search`, `web_extract`, `file_read`,
  `memory_tool` are from a specific agent framework (Hermes/CamoFox). Genericise
  these for broader reuse, or document as "replace with your framework's
  equivalents."

---

## 7. Template Architecture Decisions

### Why Full Markdown Structure?

The template now uses full Markdown (headings, lists, tables) rather than XML-style tags because:
1. **Native Rendering**: Markdown renders natively across platforms, ensuring consistent display and improved readability.
2. **Reduced Token Overhead**: Eliminating XML tags and explicit structural markers significantly reduces prompt overhead, improving token efficiency and context window utilization.
3. **Simplified Authoring & Maintenance**: Markdown is simpler to write, read, and maintain, streamlining template updates and reducing cognitive load.
4. **Improved Compatibility**: Full Markdown is compatible with a wider range of tools and renderers, enhancing portability.

### Why Three Sections?

- **Section A** (identity, scope, failure modes, tools): static — rarely changes after initial authoring
- **Section B** (knowledge priority, reasoning protocol): semi-static — updated when research findings justify protocol changes
- **Section C** (system context): dynamic — re-filled for every deployment

This structure lets maintainers update one section without touching the others.

### Why Not Embed Citations or Theory?

Explanations, rationales, and academic citations in the active agent prompt serve no operational purpose — the agent cannot verify them, and they consume significant context window. High-efficiency prompt engineering requires separating the *rules of execution* (the prompt template) from the *evidence of design* (the reference file). This keeps the active context clean, minimizes token overhead, and reduces attention decay.

### Versioning Convention

- Template engine (v7): changes to the Markdown structure or overall architecture
- Body (v7.1): additions to operational content that don't change the structure
- Context Hygiene (v7.2): major additions to a specific domain within the template
- Tool-Call Reliability (v7.3): further additions in another domain

No formal reasoning behind the dot scheme — it emerged organically. Consider
adopting semver if the template reaches wider use.
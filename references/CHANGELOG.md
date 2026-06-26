# Changelog — soul-management

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0/0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [v7.4.0] — 2026-06-26

### Added
- **Dynamic Context Hygiene:** Implemented fine-grained distillation and anchor re-injection triggers based on >20% and >30% context usage percentage, improving upon coarse message-based triggers.
- **Full Markdown Refactor:** Converted all templates from XML-style structures to standard, lightweight, and clean Markdown formatting (using semantic headings, lists, and tables), reducing prompt overhead and improving token efficiency. Removed all root tags (`<soul_template>`, `<soul_config>`) and XML structures.

### Changed
- **Language Standardization:** Fully converted all template text to English for improved portability.
- **Format Migration:** `Format` field changed from `semantic-xml` to `full-markdown` across all reference files.

## [v7.3.0-ORCH] — 2026-06-25

### Added
- **Orchestrator Reference File** — Created `references/SOUL_TEMPLATE_ORCHESTRATOR_V73_REFERENCE.md` holding all 7+ literature paper citations (AOrchestra, VMAO, HERA, Adimulam, etc.), changelogs, structural axioms, and human-maintainer design rationales.
- **Worker Reference File** — Created `references/SOUL_TEMPLATE_V7_REFERENCE.md` holding all baseline research, grounding, and context hygiene design rationales for Worker (V7) profiles.

### Changed
- **Pure Prompt Engineering Optimization** — Stripped all theoretical justifications, academic citations, "why" paragraphs, philosophical axioms, and HTML comments from active templates (`SOUL_TEMPLATE_V7.md` and `SOUL_TEMPLATE_ORCHESTRATOR_V73.md`), resulting in 15–25% token overhead reduction and increased model attention on execution instructions.
- **Model-Agnostic & General-Purpose Transition** — Removed all vendor-specific or model-specific references (Gemini, Gemma, Google models, etc.), replacing them with universal terminology (`frontier models`) for broader portability.
- **References Directory Consolidation** — Deleted 16 separate, disjointed research and note-taking drafts inside the `references/` folder and compiled them into the two canonical reference documents.
- **Skill Manual Synchronization** — Updated `SKILL.md` template inventory, key reference links, and workflows to align with the consolidated repo layout.

---

## [v7.2.1] — 2026-06-23

### Fixed
- **Placeholder format compliance** — Added mandatory format check in both Worker (V7) and Orchestrator (V73) deployment workflows. Agent must now follow the exact format specified in each placeholder's GUIDANCE/Example.
- **Post-deployment validation** — Added mandatory placeholder leak check and HTML comment leak check.
- **Simplified comment handling** — Removed "Preserve section divider comments" instructions. All HTML comments are now removed on deployment without exception.

### Changed
- Worker deployment workflow: 8 → 9 steps (added compliance check + validation)
- Orchestrator deployment workflow: 8 → 9 steps (added compliance check + validation)
- Pitfalls point 2 updated to match new comment handling rule.

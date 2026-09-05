# CCE_SKILL

Evidence-gated OpenCode workflow for direct CCE-C operator development on **Ascend 310P / DaVinci AI Core**.

Current package:

- `ascend310p_ccec_davinci_opencode_agent_pack_v3.zip`
- internal version `3.1.0`
- 1 operator orchestrator + 14 operator stages
- 1 independent `ascend310p-ccec-kb-ingestion` Skill for bootstrapping the knowledge base from known-good operator directories and/or interface-document PDFs

The ingestion Skill is never called by the operator orchestrator. After the independent bootstrap run, routine knowledge updates are handled exclusively by `ascend310p-ccec-kb-curation`.

Version 3.1 validates candidate cards before promotion, applies strict compatibility-first KB retrieval, enforces evidence-bearing stage reports, and closes terminal failed runs through release audit plus failure-only KB curation.

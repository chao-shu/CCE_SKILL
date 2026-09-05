# CCE_SKILL

Evidence-gated OpenCode workflow for direct CCE-C operator development on **Ascend 310P / DaVinci AI Core**.

面向 Ascend 310P 的 CCE-C 算子开发、验证与优化工作流。

**开始使用：[中文使用说明](README.zh-CN.md)**，包含安装、独立知识摄取、主工作流提示词、结果查看、去重、回滚和升级步骤。

Current package:

- `ascend310p_ccec_davinci_opencode_agent_pack_v3.zip`
- internal version `3.2.0`
- 1 operator orchestrator + 14 operator stages
- 1 independent `ascend310p-ccec-kb-ingestion` Skill for bootstrapping the knowledge base from known-good operator directories and/or interface-document PDFs

The ingestion Skill is never called by the operator orchestrator. After the independent bootstrap run, routine knowledge updates are handled exclusively by `ascend310p-ccec-kb-curation`.

Version 3.2 validates stage run IDs and real input/output/evidence file hashes, blocks duplicate facts across different card IDs, requires a complete environment fingerprint for production KB queries, and checkpoints matching code/design/verification artifacts for optimization rollback.

本次修复：报告文件与哈希校验、知识重复检测及合并决策、生产检索版本约束、优化回滚证据一致性。包内 `python tools/self_check.py` 包含离线回归测试；通过这些测试不代表真实 310P 编译或板端验证通过。

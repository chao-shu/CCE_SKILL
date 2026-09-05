# Ascend 310P CCE-C 算子工作流：中文使用说明

版本：3.2.0。压缩包沿用 `ascend310p_ccec_davinci_opencode_agent_pack_v3.zip` 文件名，具体版本看本说明和 README。

这套工作流帮助支持 Skill 和 subagent 的 Agent，根据 Python 参考实现开发、修复、验证和优化 Ascend 310P 的 CCE-C 算子。它包含 1 个总控 Skill、14 个算子阶段 Skill，以及 1 个独立知识摄取 Skill。

它提供流程、知识结构和检查工具；不会附带 CANN 编译器、NPU 驱动、板端环境，也不自带一套已经验证完毕的算子 API 知识库。

## 1. 使用前需要什么

- 目标硬件为 Ascend 310P，内核编程方式为直接 CCE-C / `.cce`。默认不会改用 Ascend C、TIK 或 TBE。
- Agent 能读取本地文件、执行命令、按名称或路径加载 Skill，并创建 subagent。仅贴一个 Skill 路径，不会让不支持 subagent 的 Agent 自动具备这项能力。
- 一个包含算子构建、注册、调用或测试约定的工程，以及 Python 参考实现或待修复的 `.cce` 文件。
- 若需要完成验证，应能访问匹配的 CANN/CCE 工具链、310P 设备和工程测试/性能测量命令。缺少环境时必须如实报告 BLOCKED 或 NOT_RUN。
- 辅助脚本使用 Python 3.10 或更新版本的标准库。PDF 文本提取/OCR 能力由你的 Agent 环境提供。

## 2. 如何放进工作目录

解压后，将包内的 `.opencode/skills/`、`tools/`、`contracts/`、`knowledge/` 等内容合并到算子工程根目录。注意复制以点开头的目录。工程原本有 `AGENTS.md`、同名 tools 或知识库时，先比较并合并，保留原有工程规则和数据，不要直接覆盖。

目前工具的默认相对路径以工程工作目录为基准。推荐在工程根目录运行 Agent 和命令。把整个包作为一个嵌套子文件夹放入工程、只提供主 Skill 路径，并不等于已经配置好工具路径和输出路径。

| 路径 | 用途 |
|---|---|
| `.opencode/skills/ascend310p-ccec-operator-orchestrator/SKILL.md` | 算子开发主入口 |
| `.opencode/skills/ascend310p-ccec-kb-ingestion/SKILL.md` | 独立知识摄取入口 |
| `knowledge/` | Markdown 知识卡，作为知识源文件 |
| `tools/` | 环境探测、索引、校验、去重和回滚辅助工具 |
| `contracts/` | 各阶段输入输出与证据约定 |
| `.operator-agent/runs/<run_id>/` | 本次执行的状态、报告、日志和优化记录 |
| `.operator-agent/kb.sqlite` | 可重建的检索索引 |

安装后在工程根目录执行：

```bash
python tools/self_check.py
```

通过只表示包结构与离线校验行为正常，不代表算子已编译或在 310P 上运行。

## 3. 第一次使用：单独摄取知识

包内的默认卡片主要是目标约定和概念线索，不足以证明具体 intrinsic 在你的环境中可用。你可以提供：

- 已经正确运行的算子目录，尽量包括源码、构建命令、CANN/编译器版本、精度结果、板端记录；
- 接口文档 PDF，尽量提供对应的产品范围和版本；
- 或同时提供两种输入，让 Agent 交叉核对。

可直接给 Agent 以下提示词，把路径替换成实际位置：

```text
在我的算子工程根目录工作。
以 subagent 的方式使用 skill `.opencode/skills/ascend310p-ccec-kb-ingestion/SKILL.md`。
这是一次独立知识初始化，不要调用算子总控。
先执行 python tools/init_run.py --operator kb-bootstrap，使用命令返回的目录作为 RUN_DIR。
正确算子目录：/实际路径/known-good-operators。
接口文档：/实际路径/ccec-manual.pdf。（没有该输入时删除这一行）
目标是 Ascend 310P / DaVinci / CCE-C。
核对来源、版本与运行证据；先形成候选、检查与已有知识的重复和冲突，再按契约晋升。
输出摄取报告、每张候选的去重处理决定和未解决问题。
```

“正确算子”是输入线索，仍要核对证据。只有源码或 PDF 解析成功，不等于已经得到板端验证。未验证内容保留为候选或隔离，不能伪造验证结果。

摄取由你独立调用一次完成初始化。后续算子执行产生的经验只通过 `kb-curation` 更新。总控不会自动重新调用摄取；你主动提供新资料并要求再摄取时才再次执行。

## 4. 日常开发：调用主工作流

```text
以 subagent 的方式使用 skill `.opencode/skills/ascend310p-ccec-operator-orchestrator/SKILL.md`。
工程工作目录：/实际路径/my-operator-project。
Python 参考实现：python_ref/my_op.py。
目标：Ascend 310P / DaVinci / CCE-C。
按完整流程完成设计、代码、构建、精度、性能、回归与发布审计。
先确认工程布局；将源码和测试用例放入工程对应的 MyOp 算子目录，具体路径记录在 repo_context.json。
遵守当前工程构建和注册方式。缺少证据或硬件能力时如实报告，不要猜 API 或把 NOT_RUN 写成 PASS。
```

总控依次安排环境/工程调查、语义分析、知识检索与测试规划、硬件设计、代码生成、构建、精度、性能分析、优化、回归、发布审计和知识维护。

修复已有算子时，在提示中同时提供现有 `.cce` 路径和 Python oracle。只想做设计时可单独调用 `ascend310p-ccec-davinci-design`，并提供语义、环境、工程上下文和知识证据；不要把一次设计输出当成完整发布。

算子源码和可执行用例的位置遵循实际工程，不会因为工具创建了 RUN_DIR 就自动落入某个固定算子文件夹。`repo_context.json` 应先记录目标路径；执行报告集中存放在 RUN_DIR。

## 5. 怎么查看结果

先看 `RUN_DIR/release_report.json`，再按其中路径查看细节。

| 状态 | 含义 |
|---|---|
| PASS | 对应阶段确实完成，并有支撑证据 |
| FAIL | 已执行但未通过 |
| BLOCKED | 缺少环境、输入或必要条件，无法完成 |
| NOT_RUN | 没有执行，不能解释成通过 |

`BOARD=NOT_RUN` 就不能称为“310P 板端验证通过”；没有性能测量也不能称为“已证明提速”。工作流中途失败仍会生成收尾审计，并让 curation 记录有依据的失败知识。

`stage_reports/<stage>.json` 是各阶段的校验凭据，`state.json` 由总控维护。新版报告包含输入/输出 SHA-256，证据必须指向实际文件及其哈希。校验能发现文件缺失、内容变化和串用其他 run 的报告；Agent 仍须检查日志内容是否真正支持结论。

## 6. 知识库如何避免越来越冗余

每次摄取或 curation 先与已有知识比较，并为每张候选记录以下决定：

- SKIP：已有知识覆盖、没有新增价值，不新增卡片。
- MERGE：相同适用范围有新证据或有效修正，更新原卡片并保留 ID。
- ADD：确有新事实，或版本/签名/适用条件需要独立保存。
- QUARANTINE：存在冲突或证据不足，保留问题但不作为代码生成依据。

`kb_dedupe.py` 只读扫描；`kb_lint.py` 会拒绝不同 ID 下完全相同的事实和适用范围。近义表述仍需要 Agent 判断，不会按相似度自动删除。具体契约见 `contracts/KB_DEDUPLICATION.md`。

```bash
python tools/kb_lint.py
python tools/kb_index.py
python tools/kb_dedupe.py --knowledge knowledge
```

下面是生产查询格式示例，两个版本占位符必须替换成 environment.json 中的实际值：

```bash
python tools/kb_query.py "copy" --soc Ascend310P --architecture DaVinci --cann "实际CANN版本" --compiler "实际CCE编译器版本" --backend cce-c --usable-for-codegen true --status VERIFIED
```

未知版本不能用于生产查询。仅查概念线索时，可以使用：

```bash
python tools/kb_query.py "vector" --soc Ascend310P --include-unknown-dimensions
```

概念搜索结果不能直接证明 API 可用于代码生成。SQLite 是索引，不是源文件；改动 Markdown 后需要重新 lint 并建立索引。

## 7. 优化失败时怎么回退

每轮优化在 `RUN_DIR/optimization/<iteration>/` 保存已接受版本和候选版本，包含相关代码、设计和验证报告。候选必须重新完成构建、精度、性能对比，再决定 KEEP 或 REVERT。

REVERT 会恢复明确列出的文件及对应报告，并重新校验；只回退代码、保留失败候选报告是不允许的。候选的新文件在删除前会被候选快照保留。回滚工具不负责未列入清单的文件或外部板卡状态。详细命令见 `contracts/OPTIMIZATION_EVIDENCE.md`，通常由总控安排，无需你逐轮手动操作。

## 8. 从 3.1.0 升级

1. 备份已有 knowledge 和需要保留的运行记录；停止正在执行的旧流程。
2. 更新 Skill、tools、contracts 和说明文档，合并工程规则；保留已有知识，不用包内初始知识覆盖它。
3. 在工程根目录运行 `python tools/self_check.py`、`python tools/kb_lint.py`。若检测到重复知识，按去重契约合并并保留来源，不要直接批量删除。
4. 重新执行 `python tools/kb_index.py`。
5. 新任务建立新 run。旧报告缺少 output_hashes 或文件证据时不能直接通过新版门禁；从真实保留证据重新生成，证据缺失则重跑受影响验证。

运行记录默认不提交 Git；需要长期支撑知识卡的日志和证据应另行归档并保持路径可追溯。升级工作流与发布算子是两个不同动作，这次包的自检不替代你的真实编译、精度或板端测试。

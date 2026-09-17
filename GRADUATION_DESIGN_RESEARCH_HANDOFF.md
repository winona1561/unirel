# CTI 关系抽取毕业设计研究交接文档

更新日期：2026-09-17  
当前节点：D 已停止、E confirmatory 已阻塞；F screening 已因可见关系原型缺失而 fail-closed，备选 A 未通过；CyberEntRel/CTINexus 尚未通过外部数据准入；无新增人工标注时转入研究问题收缩讨论；outer 仍未打开  
项目目录：`E:\代码\UniRel-main\UniRel-main`

## 1. 新对话接手指令

先阅读 `AGENTS.md`、`Riichi.md`、本文档，以及 `CTI_PROJECT_HISTORY.md` 最后的最新记录。先核验本文列出的权威文件和哈希，再从第 9 节继续。不要重新运行已关闭的 v1 路线，不要读取锁定评测，不要把模型辅助审核材料当作人工金标。

## 2. 当前研究方向与最终目的

项目研究网络威胁情报（CTI）中的 **18 类有向候选实体对多标签关系识别**。长期论文主题调整为：

> 面向网络威胁情报的证据感知、跨域泛化与开放关系识别。

后续分别检验三个可证伪问题：

| 方向 | 研究问题 | 当前状态 |
| --- | --- | --- |
| D：角色条件化关系证据对齐 | 不同关系及主客体方向是否依赖不同证据？ | inner screening 完成；D05 未晋级、无 survivor，D 在此门槛停止 |
| E：关系条件化实体捷径抑制 | 模型能否减少实体共现捷径并跨真实来源泛化？ | 403/403 development 记录无可信来源元数据；confirmatory 阻塞，仅允许 diagnostic-only |
| F：多标签开放关系识别 | 已知关系与未知关系共存时能否同时保留与检测？ | 三个 controlled relation-holdout schemes 与 CPU 合同已冻结；仅 2 个选定关系共存对，禁止自然共存主张 |

最终成果应提供可复现的任务定义、冻结数据与评测协议、强匹配基线、独立消融、报告级不确定性分析、失败条件和证据边界。任何创新点必须能独立成立；组合收益只能在各方向独立验证后研究。

## 3. M1 已完成事项

### 3.1 工程与冻结资产

- 活动父模型：`tools/train_cti_multilabel_atgl.py::CTIMultilabelATGLModel`，由 `tools/train_cti_stix_ore_base.py::Task78StixOreModel` 包装。
- 候选构造入口：`tools/build_cti_multilabel_pairs.py`。
- 任务：18 类有向候选实体对多标签分类。
- 冻结训练集：9,986 个候选对、508 个文本记录、1,262 个正对、1,265 个关系三元组。
- 原始开发集：822 个候选对、159 个文本记录。
- 历史 `dev_select`：76 个候选对、38 个文本记录，仅作历史诊断和 v1 选择，不是 v2 无偏外层评测。
- 5 折训练侧 OOF 清单按 `report_id` 分组，无组间泄漏；但全部训练记录都轮流成为 held-out，因此不能直接充当新的 untouched v2 outer evaluation。
- 冻结基线、训练集、开发集、标签及 OOF 绑定检查通过。

### 3.2 训练文本血缘修复

此前来源工具使用失效默认路径，表现为 508 条全部 `unmatched_raw_report`。现已使用 `data/external/task103a/sources/AZERG-Dataset/train/azerg_T4_train.json` 重新冻结训练侧血缘：

- 扫描 1,510 条原始 AZERG 训练行；
- 508/508 个冻结训练文本记录全部匹配到至少一条原始行；
- 原始文件 SHA256：`0da9fc7f5a8857fc25f93ac72eed850179094e6a33d4216871c62c5cc6bfeaa9`；
- 该文件只有 `instruction/input/output`，没有可采信的发布者、来源族、报告 URL 或时间字段；
- 发布者来源解析仍为 0/508，来源族为 0；
- 禁止根据文本内容猜测来源。

因此已经恢复“原始 AZERG 行 → 文本记录 → 候选对”的血缘，但尚未恢复“发布者报告 → 文本记录”的映射。`report_id` 目前只能解释为文本记录指纹，不能称为真实发布报告 ID。

### 3.3 D/E/F 数据可行性

- 训练集只有 3 个天然多标签候选对，F 的“已知与未知关系共存”主张缺少充足自然样本。
- 1,262 个训练正对中，1,256 个存在反向候选；1,250 个的反向候选没有标注关系，6 个双向均为正。该统计只说明数据结构，不能把缺失反向标签宣称为现实负事实。
- 所有 9,986 条训练记录都有候选证据；1,262 个正对都有已有 evidence sentence ID；892 个正对至少有一个候选句同时包含两端实体。
- 这些 evidence ID 不是人工证据金标，不能报告 evidence F1，也不能据此声称因果解释。
- 训练侧有 1,118 个唯一规范化实体；历史 `dev_select` 的 66 个实体中有 41 个训练未见实体，76/76 个有向实体组合均未在训练出现。这可支持有限的未见实体诊断，但不能替代真实跨来源评测。

## 4. v1 路线封存结论

- Route A：安全删句只覆盖 12/76，启动前提失败，未正式训练。
- Route B：三种子平均 micro-F1 增量 `+0.008784`，低于预注册 `+0.01`，结论为 `complete_gate_failed`；锁定 final 未授权。
- Route C：预注册已冻结但尚未实现、未产生新结果。M1 决定在查看任何新 C 结果前退役该路线；保留预注册作为历史机制基线，不实施、不换名并入 v2。

Route C 历史预注册：`cti_improvement/no_new_label_runs/relation_text_joint_encoding/preregister_relation_text_joint_encoding.json`  
SHA256：`957ffab70af3ec0cdba5e4c80daa60f6ec35b9f34e5f2b2fca1eaa2234744304`

## 5. M1 权威产物

- v2 准入审计：`cti_improvement/research_protocol/readiness_audit.json`
- 可复现审计入口：`tools/audit_cti_research_protocol_readiness.py`
- 训练来源冻结：`output/training_source_provenance_freeze/SOURCE_PROVENANCE_MANIFEST.json`
- 文本记录血缘：`output/training_source_provenance_freeze/report_source_provenance.jsonl`
- OOF 分组冻结：`output/report_grouped_oof_predictions/folds/FOLD_FREEZE_MANIFEST.json`
- 冻结基线：`reports/baseline_freeze.json`
- Riichi 长期计划：`Riichi.md`
- 研究历史：`CTI_PROJECT_HISTORY.md`
- v2 总冻结清单：`cti_improvement/research_protocol/V2_PROTOCOL_FREEZE_MANIFEST.json`
- v2 outer/inner 策略：`data/v2_report_grouped_evaluation_policy.json`
- v2 划分冻结：`output/v2_report_grouped_evaluation_freeze/V2_REPORT_GROUPED_EVALUATION_FREEZE.json`
- D/E/F 创新卡：`cti_improvement/research_protocol/innovation_cards.json`
- D 最小实验表：`cti_improvement/research_protocol/d_minimal_experiment_matrix.csv`
- D CPU 合同总清单：`cti_improvement/research_protocol/D_CPU_CONTRACT_FREEZE_MANIFEST.json`
- D inner 运行预注册：`cti_improvement/research_protocol/preregister_d_inner_fold_runs.json`
- D CPU 验证报告：`cti_improvement/research_protocol/d_cpu_contract_verification.json`
- D 语义编码策略：`data/d_relation_semantic_encoding_policy.json`
- D 真实语义冻结：`output/d_relation_semantic_freeze/RELATION_SEMANTIC_FREEZE_MANIFEST.json`
- D 六臂参数审计：`cti_improvement/research_protocol/d_screening_arm_parameter_audit.json`
- D 当前 screening 准入总清单：`cti_improvement/research_protocol/D_SCREENING_ADMISSION_FREEZE_MANIFEST.json`
- D screening 执行合同：`cti_improvement/research_protocol/d_screening_execution_contract.json`
- D screening 输出 schema：`data/d_screening_output_schema.json`
- D screening CPU-safe 入口：`tools/run_d_screening.py`
- D screening 执行合同总清单：`cti_improvement/research_protocol/D_SCREENING_EXECUTION_FREEZE_MANIFEST.json`
- D CPU training backend：`tools/d_screening_training_backend.py`
- D CPU backend 验证入口：`tools/verify_d_screening_training_backend_cpu.py`
- D CPU backend 验证报告：`cti_improvement/research_protocol/d_training_backend_cpu_verification.json`
- D CPU backend 当前权威清单：`cti_improvement/research_protocol/D_TRAINING_BACKEND_FREEZE_MANIFEST.json`
- D 完整 CPU epoch/inner-heldout 编排：`tools/d_screening_epoch_orchestrator.py`
- D 完整编排 CPU 验证入口：`tools/verify_d_screening_epoch_orchestration_cpu.py`
- D 完整编排 CPU 验证报告：`cti_improvement/research_protocol/d_epoch_orchestration_cpu_verification.json`
- D 完整编排当前权威清单：`cti_improvement/research_protocol/D_EPOCH_ORCHESTRATION_FREEZE_MANIFEST.json`
- D GPU 用户授权清单：`cti_improvement/research_protocol/D_GPU_SCREENING_AUTHORIZATION.json`
- D GPU 单 run 执行器：`tools/execute_d_screening_gpu.py`
- D GPU 服务器 16-run 入口：`scripts/run_d_screening_gpu.sh`
- D GPU screening 启动权威清单：`cti_improvement/research_protocol/D_GPU_SCREENING_LAUNCH_FREEZE_MANIFEST.json`
- D screening 结果审计入口：`tools/audit_d_screening_results.py`
- D screening 结果审计：`cti_improvement/research_protocol/d_screening_result_audit.json`
- D screening 分析报告：`output/d_screening_analysis/analysis-report.md`
- D screening 精确汇总：`output/d_screening_analysis/exact-summary.csv`
- D screening 结果冻结总清单：`cti_improvement/research_protocol/D_SCREENING_RESULT_FREEZE_MANIFEST.json`
- E 来源元数据准入审计入口：`tools/audit_e_source_metadata_readiness.py`
- E 来源元数据准入审计：`cti_improvement/research_protocol/e_source_metadata_readiness_audit.json`
- E 来源元数据准入冻结总清单：`cti_improvement/research_protocol/E_SOURCE_METADATA_READINESS_FREEZE_MANIFEST.json`
- F relation-holdout 支持度审计入口：`tools/audit_f_relation_holdout_support.py`
- F relation-holdout 支持度审计：`cti_improvement/research_protocol/f_relation_holdout_support_audit.json`
- F exploratory 协议与 Research Question Card：`cti_improvement/research_protocol/f_exploratory_relation_holdout_protocol.json`
- F episode schema：`data/f_relation_holdout_episode_schema.json`
- F 纯 CPU 合同：`tools/f_relation_holdout_contract.py`
- F CPU 验证报告：`cti_improvement/research_protocol/f_relation_holdout_cpu_contract_verification.json`
- F 协议历史冻结：`cti_improvement/research_protocol/F_RELATION_HOLDOUT_PROTOCOL_FREEZE_MANIFEST.json`
- F formal development backend：`tools/f_relation_holdout_backend.py`
- F00–F05 schema-conditioned 模块：`model/schema_conditioned_open_relation.py`
- F 六臂参数审计：`cti_improvement/research_protocol/f_relation_holdout_arm_parameter_audit.json`
- F 合成 epoch-0 CPU 验证：`cti_improvement/research_protocol/f_relation_holdout_backend_cpu_verification.json`
- F backend 历史冻结：`cti_improvement/research_protocol/F_RELATION_HOLDOUT_BACKEND_FREEZE_MANIFEST.json`
- F scheme-specific parent：`model/f_scheme_specific_parent.py`
- F CPU loss/optimizer backend：`tools/f_relation_holdout_training_backend.py`
- F 合成单批过拟合验证：`cti_improvement/research_protocol/f_relation_holdout_training_cpu_verification.json`
- F training-contract 历史冻结：`cti_improvement/research_protocol/F_RELATION_HOLDOUT_TRAINING_CONTRACT_FREEZE_MANIFEST.json`
- F episode schedule 授权：`cti_improvement/research_protocol/F_EPISODE_SCHEDULE_AUTHORIZATION.json`
- F episode schedule materializer：`tools/materialize_f_relation_holdout_episode_schedule.py`
- F 聚合 episode schedule：`output/f_relation_holdout_episode_schedule/F_EPISODE_SCHEDULE_MANIFEST.json`
- F episode schedule 审计：`cti_improvement/research_protocol/f_relation_holdout_episode_schedule_audit.json`
- F CPU dry-run 验证：`cti_improvement/research_protocol/f_relation_holdout_episode_dry_run_verification.json`
- F episode schedule 历史冻结：`cti_improvement/research_protocol/F_EPISODE_SCHEDULE_FREEZE_MANIFEST.json`
- F base encoder 来源冻结：`cti_improvement/research_protocol/F_BASE_PRETRAINED_ENCODER_SOURCE_FREEZE.json`
- F 单批 CPU 烟测授权：`cti_improvement/research_protocol/F_FORMAL_BATCH_CPU_SMOKE_AUTHORIZATION.json`
- F 正式 scheme parent：`model/f_formal_scheme_parent.py`
- F 正式 token/feature backend：`tools/f_formal_batch_cpu_backend.py`
- F 单批 CPU 烟测入口：`tools/verify_f_formal_batch_cpu.py`
- F 单批 CPU 烟测报告：`cti_improvement/research_protocol/f_formal_batch_cpu_smoke_verification.json`
- F 单批 CPU 烟测历史冻结：`cti_improvement/research_protocol/F_FORMAL_BATCH_CPU_SMOKE_FREEZE_MANIFEST.json`
- F 正式编排合同授权：`cti_improvement/research_protocol/F_FORMAL_TRAINING_ORCHESTRATION_AUTHORIZATION.json`
- F 正式运行预注册：`cti_improvement/research_protocol/preregister_f_formal_runs.json`
- F CPU-only 两阶段编排合同：`tools/f_formal_training_orchestrator.py`
- F 合成编排验证：`cti_improvement/research_protocol/f_formal_training_orchestration_cpu_verification.json`
- F runner/CPU execution contract 授权：`cti_improvement/research_protocol/F_FORMAL_EXECUTION_RUNNER_AUTHORIZATION.json`
- F screening execution contract：`cti_improvement/research_protocol/f_formal_screening_execution_contract.json`
- F fail-closed 正式入口：`tools/run_f_formal_screening.py`
- F CPU execution contract 验证：`cti_improvement/research_protocol/f_formal_screening_execution_contract_cpu_verification.json`
- F backend 合成验证授权：`cti_improvement/research_protocol/F_FORMAL_CPU_TRAINING_BACKEND_AUTHORIZATION.json`
- F 独立正式 CPU backend：`tools/f_formal_cpu_training_backend.py`
- F backend 合成验证：`cti_improvement/research_protocol/f_formal_cpu_training_backend_verification.json`
- F 当前权威清单：`cti_improvement/research_protocol/F_FORMAL_CPU_TRAINING_BACKEND_FREEZE_MANIFEST.json`

准入审计结论：

```text
M1_audit_complete=true
baseline_and_candidate_contract_reproducible=true
raw_text_record_mapping_complete=true
source_metadata_complete=false
v2_main_result_training_allowed=false
test_data_read=false
locked_evaluation_accessed=false
```

## 6. 数据与评价边界

- 只允许既有 AZERG 关系标签进入严格同数据主线。
- 195 条模型辅助审核、11 条关系边界复核、4 条待人工裁决记录和 R-1～R-4 规则建议继续保持 `diagnosis_only=true`、`human_gold=false`、`training_allowed=false`。
- 不自动引入外部关系标签、LLM 伪标签或人工证据标注。
- 不读取 Task105 或其他锁定评测来设计 v2。
- 历史 dev/test 结果不能重新包装成新的独立验证。
- source/vendor/time 必须来自明确元数据或证据支持的映射，不能由实体名或正文猜测。

## 7. 当前决策

M1 工程与数据审计以及 M2 v2 协议冻结已经完成。v2 outer 是从 original train 中事前划出的 105 个文本记录组、1,834 个候选对；development 是 403 组、8,152 对，并已冻结 5 个 inner folds。outer 覆盖 17 个非单例关系，唯一单例关系 `has` 保留在 development。这里的 untouched 是 v2 前瞻性含义，不能声称这些历史训练记录从未被旧实验使用。

D 的 seed13、inner folds 02/03 共 16 个 screening runs 已在服务器完成并通过结果审计。D04 的等折平均 micro-F1 为 `0.9196653648`，仅作为注册的匹配控制和描述性领先项；D05-name 为 `0.9147557248`，相对 D00 的平均差为 `-0.0021181498`，两个 folds 均未优于 D00 或 D04。D05-definition 与 D05-permuted 的 3,064 个配对预测在阈值 0.5 下决策差异为 0，虽最大概率绝对差为 `0.0135483742`，但没有形成决策级语义敏感性证据。因此 D05 不晋级完整 inner，正式 survivor 为空，D 不再追加 GPU/full-inner 训练，也不将 D04 改名包装为 D 创新。该判断只适用于本次预注册筛选；仅 1 seed、2 folds，不做显著性推断，也不能声称关系语义永远无效。

E 的 development-only 来源元数据准入审计也已完成。审计仅使用 frozen inner fold 02 train+heldout，覆盖 403 个文本记录指纹和 8,152 个候选对；403/403 均无可采信的 publisher/vendor/source family、report URL、publication time 或 upstream report ID，确认来源族为 0。`source_candidate_pair_id` 只是候选生成血缘，不能当作发布者来源。E confirmatory 因而冻结为 `blocked_missing_publisher_source_lineage`，不授权 E GPU 或跨来源实验；只允许 development-only 未见实体、实体名匿名化敏感性和有向实体对新颖性诊断，且不得使用“跨来源泛化改善”表述。F 仍只允许 relation-holdout 探索。

F 已冻结 formal development episode schedule，并完成当前唯一授权的 full-stack CPU epoch-0 smoke。仓库本地 `bert-base-cased` 五文件聚合 SHA256=`db1de69f33d462d1422903c4eb5644e536243b70c4824e3efcdd98011f30c084`，只允许 exact-hash、local-files-only 加载；本地资产无法证明远端 revision，因此不作远端 revision 声明。feature 统计只取 fold00 `communicates_with` scheme 经整报告隔离后的 5,886 个 train rows，heldout 参与数为 0；model-facing batch 不含标签、隐藏关系或 ID。唯一 formal batch 为 2 rows，CPU epoch-0 total loss=`1.3733675479888916`，known passthrough delta=`0.0`，F03 unknown scores=`[0.5,0.5]`。当前仍没有 optimizer/backward、参数更新、checkpoint/run、正式训练或 GPU。

完整 development 只有 3 个天然多标签对，三个选定 scheme 仅涉及其中 2 个唯一 `targets + variant_of` 对。因此 known+unknown 非互斥仅在合成合同中验证接口，在真实数据上只能描述 2 个共存样本，不能进入支持门槛、显著性推断或论文的自然共存解决主张。受控 relation holdout 也不能表述为识别任意现实未知关系。

## 8. M2 v2 协议冻结结果与下一阶段

M2 已完成以下冻结，期间未启动 GPU 训练：

1. outer/development 报告组互斥，覆盖完整 508 组；outer 不物化行文件，只冻结组成员。
2. development 的 5 个 inner folds 报告互斥，heldout 合计覆盖完整 403 组。
3. 五种子固定为 `13, 42, 1729, 2026, 31415`；阈值固定 0.5；报告级 paired bootstrap 为 10,000 次、seed `20260913`。
4. outer 只允许在方法、匹配基线、checkpoint、五种子、阈值、指标、bootstrap 和否证门槛全部哈希冻结后一次性打开；不得用 outer 早停、选超参数或返调。
5. D/E/F 创新卡和 D 最小实验表均已由总清单哈希绑定；E/F 限制保持不变。

总冻结清单 SHA256：`8240e2b221c04c4ba02598b1b107f66607cee7ecad272cca0c6ec0fd5d41651b`。

D CPU 合同、真实关系语义和六臂参数审计现已完成。早期 D CPU 总清单 SHA256 为 `a6f0aa5e1133c328588b5fc95bdf755484e7394b8f91c358d2e1f07925460710`，只作为当时实现的历史 checkpoint；screening 准入清单 SHA256 为 `2235a663edf071c520e596030204bd50d90556c93ed4267d9035316eae4393f9`。name/definition/permuted tensors 分别为 `6ac984b1…`、`e9f0f78d…`、`30976299…`，错配固定点为 0。这些早期清单均由下述完整编排清单接管当前权威角色。

早期 runner-only 执行清单 SHA256 `d719fc0d8dee9ab15981e6d6231effd2ff7e103021f09517c4fc7805cae670e3` 仅保留为历史 checkpoint；其后 CPU backend 与完整 epoch 编排门槛均已完成。当前 `epoch-contract` 核验 16 项允许绑定且不解析记录、不创建 `output/d_screening_runs`；无授权 execute 仍在绑定读取前失败。

D CPU training backend、完整 CPU 编排与 GPU 启动清单现在作为历史 checkpoint 保留。服务器 16/16 runs、96/96 文件哈希条目及 16/16 completion manifests 均已核验；本地逐字节重算了 80 个非 checkpoint 文件，16 个未下载 checkpoint 的服务器哈希均与各 completion manifest 匹配。所有 run 均绑定授权 SHA256 `c7ff26e9bfba677cb9977e0c7af8305261a7f212477980ff5b3cdda49b24829a`，且 outer/锁定评测访问标志全部为 false。结果冻结总清单 `D_SCREENING_RESULT_FREEZE_MANIFEST.json` SHA256=`078b0b0fcfb7d37fad1859e3c3657a53192a5f4b044a80b67112fb2ac8783327`，绑定 14 项审计、分析、哈希与测试产物。当前结论为 D05 未晋级、正式 survivor `[]`、D 在 inner screening 停止；outer opening 仍未授权。

E 来源元数据准入权威清单 `E_SOURCE_METADATA_READINESS_FREEZE_MANIFEST.json` SHA256=`e37d8de2efecfeb8b8e1cefde45de76148fda615ad19a3fb0fb4c3dd7f120ea5`，8 项绑定通过。E 审计完成时要求在 diagnostic-only E 与 exploratory F 中另行选择；随后用户已选择并只授权冻结 F 的协议与 CPU 合同，这仍不自动获得训练或 outer 权限。

F training-contract、episode schedule、单批 CPU smoke 与 runner-only 历史清单继续作为阶段记录。当前权威清单 `F_FORMAL_CPU_TRAINING_BACKEND_FREEZE_MANIFEST.json` SHA256=`63d1c078de746ee9a7d2aa80fc1cac72180f8dc8d3de19ce3b77104d1faa87fc`，直接绑定现行 runner、独立 CPU backend、旧 fail-closed executor、授权、合同、合成 verifier/报告/测试共 10 项，全部回读匹配。上一 runner-only 清单 SHA256=`7e318975a3b766845d88f65ae9724624884d7c76ec5a5385212a47424e29a7cc` 不变，但它绑定的旧 runner 一项已被新增 backend/freeze 哈希门槛的现行 runner 替代；当前清单显式记录这唯一 drift。screening 仍固定 folds 02/03、seed13、三 schemes、F00–F05、6 个共享 parent/36 个 arm 与 24/252 个 parent/arm 输出路径。backend 的两阶段预算、prototype、指标、原子文件/恢复合同已在纯合成 CPU 数据上验证，联合 pytest 23 项通过；正式 adapter 代码未在 development 数据上运行或验收。未来 `execute` 必须另有 CPU-only 授权同时绑定 contract、runner、backend、本清单哈希；该授权当前不存在，正式输出根不存在。没有读取 formal rows、再次 formal forward、加载 BERT/formal model、formal optimizer/training/checkpoint/run、GPU 或 outer。下一步由用户单独决定是否授权 development-only CPU screening，建议先一个矩阵内单元；GPU、full-inner、outer 均未授权。

2026-09-15 首单元执行更新（覆盖上一段的旧当前状态）：用户仅授权 fold02、communicates_with、seed13、F00 的 development-only CPU 首单元。历史 F_FORMAL_CPU_TRAINING_BACKEND_FREEZE_MANIFEST.json SHA256=63d1c078de746ee9a7d2aa80fc1cac72180f8dc8d3de19ce3b77104d1faa87fc 保持原字节，但其旧 runner/backend/test 绑定已由本轮必要的单单元范围防护替代，不再称 10 项当前全匹配。独立授权 F_FORMAL_SCREENING_EXECUTION_AUTHORIZATION.json 精确绑定 contract、现行 runner/backend 与历史清单，并强制单 parent/arm、禁止 full-inner、CUDA_VISIBLE_DEVICES=-1、禁止 GPU/outer；runner/backend 都在正式 import 前拒绝邻近单元。新当前清单 F_FORMAL_SINGLE_CPU_EXECUTION_FREEZE_MANIFEST.json 绑定授权、contract、历史清单、runner/backend、服务器标准库审计入口共 6 项；本地 ruff 和专用合成/边界 pytest 22 项通过。服务器 preflight 将只核验已冻结 fold02 字节哈希、BERT 来源 5 文件与空输出根；postrun 将验收单 parent 四件套和 F00 arm 七件套。代码应由 PowerShell 分目录上传，训练仅由服务器 shell 启动；当前代理无交互 SSH 返回 Permission denied (publickey,password)，所以尚未上传、未执行正式 development 单元、未生成正式输出。其他单元、full-inner、GPU、outer/锁定评测仍禁止。

首单元冻结续记（覆盖上段的 6 项版本）：发现旧 F spec/read 会对 00–04 全 inner folds 做字节哈希，但本轮未调用。新 tools/f_formal_fold02_cpu_partition.py 保留历史协议文件原字节，对其他 folds 仅核对冻结元数据，对 fold02 train/heldout 做唯一正式字节校验和旧行级标签校验。runner/backend 均将其 SHA256 纳入授权并只在单单元授权后惰性导入。当前权威 F_FORMAL_SINGLE_CPU_EXECUTION_FREEZE_MANIFEST.json SHA256=89d403c0ba5991f00adf54e0431e5d56f134bfdbe1894139a5b5b941f7c3354，7 项绑定回读匹配；本地 ruff、专用合成/边界 pytest 24 项通过。服务器无交互 SSH 仍因公钥/密码认证被拒，尚未上传或启动任何正式开发训练。下一步先由用户在 PowerShell 交互登录、按目录上传代码/授权/冻结，在服务器 CPU-only preflight 通过后，仅执行 fold02/communicates_with/seed13/F00。

服务器 preflight 修正续记（覆盖上段单元清单哈希）：PowerShell 已上传 F 执行代码、授权/冻结、模型依赖与 episode 聚合清单；服务器 CPU-only preflight 的历史编排绑定缺失/漂移已通过补传 verifier/报告/测试解除。最新失败在 BERT 文件集合判断：服务器冻结要求的 5 个同大小普通文件齐全，额外存在 .cache 目录；旧审计误把目录算作文件。只修正审计入口为统计普通文件，不删除缓存、不变更五文件 SHA256 要求。新权威 F_FORMAL_SINGLE_CPU_EXECUTION_FREEZE_MANIFEST.json SHA256=abc870d76b583b309a375cd44477c591b5da3015ac31b7c2c8d433a5eac295a4，7 项绑定匹配、ruff/专用合成与范围 pytest 25 项通过。当前下一步由用户 PowerShell 补传 tools/audit_f_formal_single_cpu_execution.py 与本清单，随后在 PyCharm 服务器终端重跑 CUDA_VISIBLE_DEVICES=-1 的 preflight；BERT 内容 SHA256 与 fold02 输入哈希仍待服务器验证。正式 F00 CPU parent/arm、任何 GPU、其他单元和 outer 均未执行。

服务器 launch 断点续记：用户已在 PowerShell 补传修正审计入口和冻结清单；服务器 CUDA_VISIBLE_DEVICES=-1 preflight 报告通过，授权 SHA256=a88b25319f088eefa1a97d40d6a5b04dacfbfd30b8d27b7435abb61c4c58fa3、当前冻结 SHA256=abc870d76b583b309a375cd44477c591b5da3015ac31b7c2c8d433a5eac295a4、BERT aggregate SHA256=db1de69f33d462d1422903c4eb5644e536243b70c4824e3efcdd98011f30c084、fold02 train/heldout SHA256=2c00c2ae723a1d9ee66d306d7dcb4f257091088e61bb28168e9915613b61799a/deb3fe70e06b82ac64dc6ad957d180449870d7dce7d1f46d21a3412ce929940a；输出根为空，outer/GPU 标志 false。服务器 CPU synthetic runtime 检查通过，固定 Python 环境 PyTorch 2.1.2+cu121、transformers 4.46.3；cgroup 内存上限 96 GiB。用户明确正式训练由 PyCharm Run Configuration 执行、本地 PowerShell 只上传。已安装 computer-use 的 node_repl/sky 内核初始化三次异常退出，未控制 PyCharm，不能代用户点击 Run；未改用 SSH/终端启动。唯一下一步为用户在 PyCharm 以服务器 interpreter /root/autodl-tmp/lgx/conda_envs/unirel38/bin/python、script /root/autodl-tmp/lgx/Unirel-main/tools/run_f_formal_screening.py、工作目录 /root/autodl-tmp/lgx/Unirel-main、CUDA_VISIBLE_DEVICES=-1 和 exact fold02/communicates_with/seed13/F00 参数点击 Run 一次。正式 parent/arm 尚未运行，GPU、其他单元、full-inner、outer 均禁止。

## 9. 新对话的下一步执行命令

将下面整段作为新对话的第一条消息：

```text
阅读 AGENTS.md、Riichi.md、GRADUATION_DESIGN_RESEARCH_HANDOFF.md 和 CTI_PROJECT_HISTORY.md 的最新记录，核验 cti_improvement/research_protocol/F_FORMAL_CPU_TRAINING_BACKEND_FREEZE_MANIFEST.json SHA256 为 63d1c078de746ee9a7d2aa80fc1cac72180f8dc8d3de19ce3b77104d1faa87fc 及其 10 项绑定。上一 runner-only 清单 SHA256 仍为 7e318975a3b766845d88f65ae9724624884d7c76ec5a5385212a47424e29a7cc，但旧 runner 一项绑定因更强的 backend/freeze 哈希门槛被现行 runner 替代，勿按旧清单要求当前 8 项全部匹配。F 的正式 CPU backend、两阶段固定 epoch/梯度累积、parent-train prototype、固定指标和原子输出合同已用纯合成 CPU 数据验证，23 项联合测试通过；没有正式 development 执行或 GPU。下一步仅由用户决定是否单独授权 development-only CPU screening；建议先限定一个注册矩阵内单元，仍不得启动 GPU、full-inner 或打开 outer/锁定评测，不得新增标签或使用模型辅助材料。
```

不要把 `report_id` 称为真实发布报告 ID；它仍只是文本记录指纹。不要因已有 outer 划分而解除 E 的来源元数据阻塞或 F 的天然多标签支持不足。

## 10. 本地复核命令

D screening 结果冻结复核：

```powershell
cd "E:\代码\UniRel-main\UniRel-main"
.venv\Scripts\python.exe -m pytest -q tests\test_d_screening_result_audit.py
Get-FileHash -Algorithm SHA256 cti_improvement\research_protocol\D_SCREENING_RESULT_FREEZE_MANIFEST.json
```

预期 4 项测试通过；结果冻结总清单 SHA256 为 `078B0B0FCFB7D37FAD1859E3C3657A53192A5F4B044A80B67112FB2AC8783327`。复核只能读取已回传的 inner screening 产物，不得连接或解析 outer。

E 来源元数据准入冻结复核：

```powershell
cd "E:\代码\UniRel-main\UniRel-main"
.venv\Scripts\python.exe tools\audit_e_source_metadata_readiness.py --mode verify
.venv\Scripts\python.exe -m pytest -q tests\test_e_source_metadata_readiness.py
Get-FileHash -Algorithm SHA256 cti_improvement\research_protocol\E_SOURCE_METADATA_READINESS_FREEZE_MANIFEST.json
```

预期审计输出包含 `source_families=0 outer_accessed=false training_executed=false`，7 项测试通过；总清单 SHA256 为 `E37D8DE2EFECFEB8B8E1CEFDE45DE76148FDA615AD19A3FB0FB4C3DD7F120EA5`。

F exploratory protocol 与 CPU 合同复核：

```powershell
cd "E:\代码\UniRel-main\UniRel-main"
.venv\Scripts\python.exe tools\audit_f_relation_holdout_support.py --mode verify
.venv\Scripts\python.exe tools\verify_f_relation_holdout_contract_cpu.py --mode verify
.venv\Scripts\python.exe -m pytest -q tests\test_f_relation_holdout_contract.py
Get-FileHash -Algorithm SHA256 cti_improvement\research_protocol\F_RELATION_HOLDOUT_PROTOCOL_FREEZE_MANIFEST.json
```

预期支持审计输出三个 schemes、`development_multilabel_pairs=3`、`selected_coexistence_pairs=2`；CPU 输出必须为 `optimizer_constructed=false gpu_used=false outer_accessed=false`，11 项专用测试通过。总清单 SHA256 为 `0E09D637ECDC85CA5B57E0CAC10B61293F9D8FEF2EE465F2324629B873E37EC7`。

F formal backend、参数审计与合成 epoch-0 冻结复核：

```powershell
cd "E:\代码\UniRel-main\UniRel-main"
.venv\Scripts\python.exe tools\audit_f_relation_holdout_parameters.py --verify
.venv\Scripts\python.exe tools\verify_f_relation_holdout_backend_cpu.py --verify
.venv\Scripts\python.exe -m pytest -q tests\test_f_relation_holdout_backend.py
Get-FileHash -Algorithm SHA256 cti_improvement\research_protocol\F_RELATION_HOLDOUT_BACKEND_FREEZE_MANIFEST.json
```

预期两个 JSON 均逐字节复现，16 项专用测试通过；当前总清单 SHA256 为 `4A2E9D8EF5D987CC3485B4D4A6B5AA252F6615B8380195DDF00F0A300AF08D7D`。这些命令只构造 CPU F 模块并使用合成张量；不得改成正式 episode 迭代、optimizer、训练或 CUDA 调用。

F scheme parent、CPU optimizer 与合成单批过拟合冻结复核：

```powershell
cd "E:\代码\UniRel-main\UniRel-main"
.venv\Scripts\python.exe tools\verify_f_relation_holdout_training_cpu.py --verify
.venv\Scripts\python.exe -m pytest -q tests\test_f_relation_holdout_training_backend.py
Get-FileHash -Algorithm SHA256 cti_improvement\research_protocol\F_RELATION_HOLDOUT_TRAINING_CONTRACT_FREEZE_MANIFEST.json
```

预期验证 JSON 逐字节复现，12 项专用测试通过；当前总清单 SHA256 为 `0C5DAFD11A649309A958AF00D069D8A714922DDAECA1D7BE2EBC49896679A890`。该验证只在合成 CPU tensors 上构造 optimizer/backward；不得接入正式 inner rows、base pretrained encoder、checkpoint/run 写入、CUDA 或 outer。

F formal episode schedule 与 CPU dry-run 冻结复核：

```powershell
cd "E:\代码\UniRel-main\UniRel-main"
.venv\Scripts\python.exe tools\materialize_f_relation_holdout_episode_schedule.py --mode verify
.venv\Scripts\python.exe -m pytest -q tests\test_f_relation_holdout_episode_schedule.py
Get-FileHash -Algorithm SHA256 cti_improvement\research_protocol\F_EPISODE_SCHEDULE_FREEZE_MANIFEST.json
```

预期输出 `episode_count=160`、`parent_train=15`、`unknown_train=130`、`inner_heldout=15`，10 项专用测试通过；当前总清单 SHA256 为 `96E3644B072164689E57040E732170DACAEA5EF94C03EEC00F5FBCD5E9FF23B6`。复核会读取冻结 development inner 并重建 CPU targets，但不得构造模型/optimizer、执行训练、写 checkpoint/run、调用 CUDA 或访问 outer。

M2 冻结边界测试与总清单哈希：

```powershell
cd "E:\代码\UniRel-main\UniRel-main"
.venv\Scripts\python.exe -m unittest tests.test_v2_report_grouped_evaluation -v
Get-FileHash -Algorithm SHA256 cti_improvement\research_protocol\V2_PROTOCOL_FREEZE_MANIFEST.json
```

预期测试为 3 项通过；总清单 SHA256 为 `8240E2B221C04C4BA02598B1B107F66607CEE7ECAD272CCA0C6EC0FD5D41651B`。

M1 来源复核入口（仅在明确需要重建 M1 产物时运行）：

```powershell
cd "E:\代码\UniRel-main\UniRel-main"
.venv\Scripts\python.exe tools\freeze_training_source_provenance.py --raw-train-file data/external/task103a/sources/AZERG-Dataset/train/azerg_T4_train.json --output output/training_source_provenance_freeze
.venv\Scripts\python.exe tools\audit_cti_research_protocol_readiness.py
```

预期第二条命令输出包含：

```text
m1_complete=true reproducible=true v2_training_allowed=false
```

## 11. 用户的代码上传与服务器训练约定

以下约定是后续接手者的默认工作方式；如用户在当次对话中给出新地址、目录或环境，以当次明确指令为准。不得把登录密码、令牌或其他凭据写入仓库。

### 11.1 固定环境信息

- 本地 Windows 项目目录：`E:\代码\UniRel-main\UniRel-main`
- 当前 SSH 入口：`root@connect.westd.seetacloud.com`
- 当前 SSH 端口：`45423`
- 服务器项目目录：`/root/autodl-tmp/lgx/Unirel-main`
- 服务器 conda 环境：`/root/autodl-tmp/lgx/conda_envs/unirel38`
- 最近一次人工核验的服务器环境：PyTorch `2.1.2+cu121`、CUDA `12.1`、单卡 NVIDIA GeForce RTX 4090。该信息只表示最近一次核验结果；更换实例后必须重新检查。

### 11.2 本地 PowerShell 只负责上传

用户习惯先由本地代码代理列出本轮确实修改且服务器运行所需的文件，再在 PowerShell 中逐目录上传。不要默认上传整个仓库，不要上传本地缓存、锁定评测、无关数据、已有输出或大 checkpoint，也不要覆盖服务器上的无关文件。

PowerShell 中 `scp` 的端口参数必须使用大写 `-P`；`ssh` 的端口参数使用小写 `-p`。同一条 `scp` 命令只组合服务器目标目录相同的文件。例如：

```powershell
Set-Location "E:\代码\UniRel-main\UniRel-main"

scp -P 45423 `
  tools\<entrypoint>.py `
  tools\<backend>.py `
  root@connect.westd.seetacloud.com:/root/autodl-tmp/lgx/Unirel-main/tools/

scp -P 45423 `
  model\<model_file>.py `
  root@connect.westd.seetacloud.com:/root/autodl-tmp/lgx/Unirel-main/model/

scp -P 45423 `
  scripts\<authorized_launcher>.sh `
  root@connect.westd.seetacloud.com:/root/autodl-tmp/lgx/Unirel-main/scripts/
```

尖括号占位符必须替换为当轮列出的真实文件名。若服务器尚无目标子目录，用户先登录服务器创建该目录，再回到 PowerShell 上传。PowerShell 不用于拼接一条“上传后立即远程训练”的命令，也不用于启动正式训练。

上传后应在服务器项目目录中核验关键文件的 SHA256，尤其是入口、backend、模型代码、启动脚本和授权/冻结清单；只有与本地冻结值一致的文件才能进入已授权运行。

### 11.3 登录后从服务器 shell 启动与监控

用户会单独登录服务器，然后在服务器 shell 中进入项目目录并启动训练：

```powershell
ssh -p 45423 root@connect.westd.seetacloud.com
```

```bash
cd /root/autodl-tmp/lgx/Unirel-main
```

不要依赖系统 `python`；该入口曾返回 `python: command not found`。CPU 合同或环境核验使用固定 conda prefix：

```bash
conda run --no-capture-output \
  -p /root/autodl-tmp/lgx/conda_envs/unirel38 \
  python -c 'import torch; print(torch.__version__, torch.version.cuda, torch.cuda.is_available(), torch.cuda.device_count(), torch.cuda.get_device_name(0)); assert torch.cuda.is_available()'
```

普通 Python 入口沿用相同形式：

```bash
conda run --no-capture-output \
  -p /root/autodl-tmp/lgx/conda_envs/unirel38 \
  python tools/<entrypoint>.py <authorized-arguments>
```

只有在相应阶段已经完成冻结并得到用户单独、明确的 GPU 授权后，才可启动 GPU 作业。后台运行采用服务器端 `nohup`，日志放在 `/root/autodl-tmp/lgx/`，例如：

```bash
nohup env CUBLAS_WORKSPACE_CONFIG=:4096:8 \
  PYTHON_BIN=/root/autodl-tmp/lgx/conda_envs/unirel38/bin/python \
  bash scripts/<authorized_launcher>.sh \
  > /root/autodl-tmp/lgx/<run_name>.log 2>&1 &
```

确定性 GPU 运行必须在第一次 Python/CUDA 调用前设置 `CUBLAS_WORKSPACE_CONFIG=:4096:8`；否则 PyTorch 的 deterministic algorithms 会在 CuBLAS 运算处失败。实际启动参数仍以当阶段冻结的正式入口和命令模板为准，不能把本节示例视为任何尚未授权实验的训练许可。

常用服务器端监控命令：

```bash
tail -n 100 -f /root/autodl-tmp/lgx/<run_name>.log
ps -ef | grep '[r]un_<authorized_name>' || true
```

### 11.4 结果回传习惯

正式运行结束后，先在服务器核验 run 数、completion manifests、文件数、不完整暂存目录和日志状态。默认回传轻量审计结果：指标、预测、配置、manifest、日志与 SHA256 清单；除非后续核验明确需要，不把多 GB checkpoint 打进下载包。

如果下载包排除 checkpoint，应在服务器生成覆盖全部 run 文件的 SHA256 清单，并确保每个未下载 checkpoint 的服务器哈希同时被对应 completion manifest 绑定。压缩包及哈希清单各自再计算 SHA256，用户下载完成后在本地复核。服务器原始输出在冻结与本地验收完成前保留，不因生成轻量包而删除。

### 11.5 始终有效的研究边界

- 上传和登录不等于训练授权；每个新门槛仍需按本交接文档所述单独冻结、单独授权。
- 未授权时不得启动 GPU、正式训练或隐式构造正式 optimizer/backward。
- 不得读取、复制、上传、下载或推导 outer 与任何锁定评测内容。
- 不得新增标签、使用模型辅助材料，或改变已封存结论。
- 所有服务器运行必须 fail-closed：代码、输入哈希、授权清单或输出绑定不一致时立即停止，不得临时绕过。

## 12. 2026-09-15 首个 F development-only CPU 单元完成断点

用户仅授权 fold02、communicates_with、seed13、F00 的一个 development-only CPU screening 单元。服务器首次启动在历史 F 协议文件缺失检查处停止，尚未训练；用户通过本地 PowerShell 补传 E_SOURCE_METADATA_READINESS_FREEZE_MANIFEST.json 和其余 8 个冻结绑定后，服务器 FOLD02_BINDINGS_OK、CPU-only preflight 重新通过。随后仅后台启动一次该单元，PID=281223，完成后日志状态 complete_development_only_CPU_formal_screening_unit。postrun 审计状态 passed_single_development_only_CPU_F00_output_audit：唯一 parent 4 件、唯一 F00 arm 7 件、completion 内部哈希与配置授权通过，GPU/outer/锁定评测标志 false。授权 SHA256=a88b25319f0888eefa1a97d40d6a5b04dacfbfd30b8d27b7435abb61c4c58fa3；当前单单元执行冻结 SHA256=abc870d76b583b309a375cd44477c591b5da3015ac31b7c2c8d433a5eac295a4。

服务器原位唯一 parent 目录为 output/f_formal_screening_runs/parents/f-screening-parent-fold02-communicates-with-seed13，唯一 arm 目录为 output/f_formal_screening_runs/arms/f-screening-f00-fold02-communicates-with-seed13。用户服务器 sha256sum 回传三个 provenance 哈希：PARENT_COMPLETION_MANIFEST.json=14c404aa31f54af0d71bc2112b6b21af2f9112d4fd1ee7d696b1adf0dde29d38；RUN_COMPLETION_MANIFEST.json=a3b1f51c8107d621d7cdda70548f09fef196c61255aa33b8a6b9d96646992dc4；inner_heldout_metrics.json=325a8f3af7058fd80e257f0e249a8ff7504b0636a352b978fba4226357ab95a0。未下载服务器产物到本地独立回算，服务器原始输出不得覆盖或删除。

该唯一 F00 指标：known micro/macro-F1@0.5 均 0.0；unknown AUPRC=0.0013454186885910103、AUROC=0.019243530192435302、FPR@95TPR=0.9900464499004645、prevalence=0.001986754966887417。数值明显偏低，但无其他 arm/fold/seed 的可比输出，不能作 F 假设支持、模型优劣、阈值调整或 full-inner 决策。Riichi 与预注册边界不因首单元完成而自动放宽。其余 35 arm、其他 parent、full-inner、GPU、outer/锁定评测均未授权；不得新增标签或使用模型辅助材料。

该 parent 5 条训练日志已由用户回传：epoch 1–5 mean_loss 分别 0.2413836459451091、0.04208108860566382、0.035255392686380456、0.03236164962624811、0.03057865698350544；每个 epoch 104 optimizer steps，总计 520。训练 loss 下降不能解释 heldout known-F1=0。冻结 F00 unknown_score=1-max(known_probabilities)，低 AUROC 的可能排序原因仍是待检验假设。

已由用户从唯一 F00 已提交 prediction 文件只读汇总：1510 行，207 个 known 阳性目标条目，固定 0.5 下 0 个 known 阳性预测；unknown=0 有 1507 行、F00 score 均值 0.9178468889959142，unknown=1 仅 3 行、均值 0.6895481745402018。这解释本单元 known-F1=0，并与低 unknown AUROC 的排序方向一致；不证明代码错误，也不足以作跨单元显著性、winner 或阈值调整结论。

该唯一 F00 predictions 文件 SHA256 已由用户服务器回传：a890fbbce284a4997ea5ce365d8330d9f99c9bb551048011a1bd3e5ed8aaf381。此前 parent/F00 completion manifests、metrics 外层哈希及 postrun 内部文件绑定亦已记录，首单元执行与输出门槛完成。哈希未在本地下载后独立复算。

下一步执行命令：无。当前服务器执行暂停，不启动新单元、GPU 或 full-inner；保留并不得覆盖唯一 parent/F00 原始产物。下一步需要用户另行明确选择根因诊断范围或授权新的 development-only 单元；任何 outer/锁定评测访问仍禁止。

## 13. 2026-09-15 F01 共享 parent CPU 单元准备断点

用户要求按 Riichi 与冻结 F screening 顺序依次执行。本轮已把 F00 diagnosis-only 聚合记入 CTI_PROJECT_HISTORY.md，不修改冻结阈值、loss 或旧结果。F00 原 runner/backend/执行授权/单单元冻结的 SHA256 仍分别为 bb5917bb7f07dfedd62a0a84e6e12267b789699c6c50bcdd3b3f46f2503f3048、70967ba01ef8c23ffdc55346bfdbc705dff69685ece50753c29995a0d18c949d、a88b25319f0888eefa1a97d40d6a5b04dacfbfd30b8d27b7435abb61c4c58fa3、abc870d76b583b309a375cd44477c591b5da3015ac31b7c2c8d433a5eac295a4；原服务器 parent/F00 输出不得覆盖。

新 F01-only、development-only CPU 入口 tools/run_f_formal_reused_parent_cpu.py SHA256=78b321cca6f7e42c2446cce6defebd541526bc21e8b5951f9997442e8dc4362c；评估 backend tools/f_formal_reused_parent_cpu_backend.py SHA256=55d13390550510a9ad06bc7b18ab189a887c914b57476bf72366694c5f49dd86。它们先核验新授权、旧 F00 parent/F00 completion 外层哈希、completion 内所有文件哈希、固定 fold02 输入/BERT 来源以及仅有旧 parent/F00 的输出状态，再惰性加载模型和 fold02 development 行。F01 只复用原 checkpoint，使用预注册 entropy unknown score，不创建 parent 或 unknown optimizer、不 backward、不覆盖原产物；新 arm 仍按七件套 staging→rename 原子提交。

新独立授权 cti_improvement/research_protocol/F_FORMAL_REUSED_PARENT_CPU_EXECUTION_AUTHORIZATION.json SHA256=81b521172eb2f65c77d2f6f78f2dcd752f7955b9a8f1fae2cb5472247ed446f6，唯一单元 fold02/communicates_with/seed13/F01，绑定原 parent completion SHA256=14c404aa31f54af0d71bc2112b6b21af2f9112d4fd1ee7d696b1adf0dde29d38、原 F00 arm completion SHA256=a3b1f51c8107d621d7cdda70548f09fef196c61255aa33b8a6b9d96646992dc4。新权威清单 cti_improvement/research_protocol/F_FORMAL_F01_CPU_EXECUTION_FREEZE_MANIFEST.json SHA256=c0aaeb70fe6c68a2dd21ba989bbb9664fb93795c20ec38475e148b6fcf764d4e，10 项本地绑定全部通过。合成边界测试 tests/test_f_formal_reused_parent_cpu.py SHA256=25a5d1ac0ed491072dac722af2a2a7f01072eb85dc4890c1a41b6617ec478c2b，4 项通过；Python 3.8 grammar/ruff 通过。正式 F01 preflight、forward、run 和 postrun 尚未执行；本地 Python 导入 torch 时在 NumPy BLAS 初始化处异常中止，未用本地环境跑 formal forward。服务器既定无交互 SSH 再次返回 Permission denied (publickey,password)，代理未上传或远程运行。用户按约定用本地 PowerShell 分目录上传新文件，再在 PyCharm 服务器终端运行。

本地 PowerShell 仅上传以下新增文件，目标目录分别对应，不上传整个仓库或旧 parent/F00 产物：

```powershell
Set-Location "E:\代码\UniRel-main\UniRel-main"
scp -P 45423 tools\run_f_formal_reused_parent_cpu.py tools\f_formal_reused_parent_cpu_backend.py root@connect.westd.seetacloud.com:/root/autodl-tmp/lgx/Unirel-main/tools/
scp -P 45423 tests\test_f_formal_reused_parent_cpu.py root@connect.westd.seetacloud.com:/root/autodl-tmp/lgx/Unirel-main/tests/
scp -P 45423 cti_improvement\research_protocol\F_FORMAL_REUSED_PARENT_CPU_EXECUTION_AUTHORIZATION.json cti_improvement\research_protocol\F_FORMAL_F01_CPU_EXECUTION_FREEZE_MANIFEST.json root@connect.westd.seetacloud.com:/root/autodl-tmp/lgx/Unirel-main/cti_improvement/research_protocol/
```

下一步执行命令：用户上传后先在服务器项目目录只读复核上述五个 SHA256；固定 CUDA_VISIBLE_DEVICES=-1，用服务器 conda interpreter 运行 `tools/run_f_formal_reused_parent_cpu.py --mode preflight`。只有返回 passed_F01_reused_parent_CPU_preflight，且 parent/F00 原目录和哈希仍绑定，才能单次运行同一入口 `--mode execute`，完成后运行 `--mode postrun` 并回传轻量指标/completion 哈希。若任一检查失败，停止，不修改冻结输入、授权或原产物。其余 F arm/parent、GPU、full-inner、outer/锁定评测均不得从 F01 单元授权推导出执行许可。

## 14. 2026-09-16 F01 完成与 F02 provenance 待闭合断点

用户服务器回传 F01 execute 完成、postrun passed_F01_reused_parent_CPU_postrun，唯一 fold02/communicates_with/seed13/F01 复用原 parent；parent_training_executed=false、unknown_optimizer_constructed=false、GPU/outer/锁定评测标志 false。F01 指标：known micro/macro-F1@0.5 均 0.0；unknown AUPRC=0.060713489409141584、AUROC=0.9798717097987171、FPR@95TPR=0.028533510285335104、prevalence=0.001986754966887417。postrun 报告授权 SHA256=81b521172eb2f65c77d2f6f78f2dcd752f7955b9a8f1fae2cb5472247ed446f6、原 parent completion manifest SHA256=14c404aa31f54af0d71bc2112b6b21af2f9112d4fd1ee7d696b1adf0dde29d38。F01 相对 F00 的单单元 unknown AUPRC 差值为 +0.05936807072055057，仅描述已冻结对照；unknown 正例仅 3 个，known-F1 仍为 0，不能选方法或声称已知关系得到有效保留。F01 arm RUN_COMPLETION_MANIFEST、metrics、predictions 的三个服务器外层 SHA256 已向用户请求，只读 provenance 待回传。

按注册顺序仅准备下个 fold02/communicates_with/seed13/F02 距离控制，本地新增 tools/run_f_formal_prototype_distance_cpu.py SHA256=c5b222a15d3bbc045bc049aafdc5003e9ea17897007d2bae284b0af87c6f0c16、tools/f_formal_prototype_distance_cpu_backend.py SHA256=aa16b944549544f1060408fac28d7bf9b5724a2a0fec0e5695391662e7b2359a、tests/test_f_formal_prototype_distance_cpu.py SHA256=7822f5e7f1192ff6f131e9efd5802121695401270a21063fd02e9612044340d3。F02 使用旧 parent_checkpoint 内冻结的 17×832 prototypes 和预注册 cosine-distance 分数；preflight 必须见到唯一原 parent、F00 arm、F01 arm，逐项核验 F00/F01 completion 及内部文件哈希，并拒绝已有 F02/stale partial。三个纯合成边界 pytest、Python 3.8 grammar、ruff 均通过；未读取正式 development 行、未加载模型/forward、未构造 optimizer 或写正式 F02 输出。本地不生成可执行 F02 授权/冻结，直到 F01 外层 completion SHA256 回传并绑定；当前下一步服务器执行命令：仅 F01 三件 provenance 的只读 sha256sum，无 F02 --mode execute 命令。保留原 parent/F00/F01 输出，GPU/full-inner/outer/锁定评测始终禁止。

## 15. 2026-09-16 F02 原型距离 CPU 单元可上传断点

F01 execute/postrun 已完成并通过，用户服务器只读回传 F01 arm/RUN_COMPLETION_MANIFEST.json SHA256=a8b9a2c30ee132d72d3d93be57f2dadbacb1dc7d9a2ed596b7e7b7ab1369ed23、inner_heldout_metrics.json SHA256=26e2b1ade7439e2a32fb4ac1f1cf0f2aae7eb55043594bf455fd6af032ec5e0d、inner_heldout_predictions.jsonl SHA256=26e91ca33e466699e678e22326382255d8bbb9df21320a99d4cdae77d9d6cd6f。postrun 已核验 completion 内部 payload 哈希；外层 SHA256 未在本地下载独立复算，原 parent/F00/F01 产物必须保持原位。F01 单元只有 3 个 unknown 正例、known-F1=0，不能按单单元 AUPRC 选择模型或调整阈值。

唯一 F02 development-only CPU 单元已本地冻结：fold02/communicates_with/seed13/F02 prototype-distance，严格共享既有 F00 parent，要求 F00/F01 两 arm 的完成清单原位匹配。新文件与 SHA256：tools/run_f_formal_prototype_distance_cpu.py=c5b222a15d3bbc045bc049aafdc5003e9ea17897007d2bae284b0af87c6f0c16；tools/f_formal_prototype_distance_cpu_backend.py=aa16b944549544f1060408fac28d7bf9b5724a2a0fec0e5695391662e7b2359a；tests/test_f_formal_prototype_distance_cpu.py=7822f5e7f1192ff6f131e9efd5802121695401270a21063fd02e9612044340d3；cti_improvement/research_protocol/F_FORMAL_PROTOTYPE_DISTANCE_CPU_EXECUTION_AUTHORIZATION.json=a750a2e986e7312936db4a469ce15c7188a0ac4fcba32a9ef2042f1b9c763270；cti_improvement/research_protocol/F_FORMAL_F02_CPU_EXECUTION_FREEZE_MANIFEST.json=665d7b5eaf51b9419b9bb02598ebaf758fbd9df1ba208b621041ba6ad32ae124。12 项本地清单绑定、授权元数据闭包、3 项纯合成边界 pytest、Python 3.8 grammar、ruff 均通过。正式 F02 服务器 preflight/forward/run/postrun 尚未执行；本地未正式推理/训练，旧冻结文件原字节保留。CPU 环境使用服务器 /root/autodl-tmp/lgx/conda_envs/unirel38/bin/python 和 CUDA_VISIBLE_DEVICES=-1，不启动 GPU、full-inner、outer/锁定评测。

本地 PowerShell 仅按目录上传五个新增 F02 文件，不上传整个仓库、旧产物、缓存或评测：

```powershell
Set-Location "E:\代码\UniRel-main\UniRel-main"
scp -P 45423 tools\run_f_formal_prototype_distance_cpu.py tools\f_formal_prototype_distance_cpu_backend.py root@connect.westd.seetacloud.com:/root/autodl-tmp/lgx/Unirel-main/tools/
scp -P 45423 tests\test_f_formal_prototype_distance_cpu.py root@connect.westd.seetacloud.com:/root/autodl-tmp/lgx/Unirel-main/tests/
scp -P 45423 cti_improvement\research_protocol\F_FORMAL_PROTOTYPE_DISTANCE_CPU_EXECUTION_AUTHORIZATION.json cti_improvement\research_protocol\F_FORMAL_F02_CPU_EXECUTION_FREEZE_MANIFEST.json root@connect.westd.seetacloud.com:/root/autodl-tmp/lgx/Unirel-main/cti_improvement/research_protocol/
```

下一步服务器执行命令：用户上传后从服务器项目目录先运行下列只读 SHA256 检查，然后在 PyCharm 服务器终端跑 preflight；只有状态 passed_F02_reused_parent_CPU_preflight，且原 parent/F00/F01 completion 哈希与输出范围通过，才单次 execute 并随后 postrun。任何失败立即停下，不改授权或旧数据/产物。

```bash
cd /root/autodl-tmp/lgx/Unirel-main
sha256sum -c <<'HASHES'
c5b222a15d3bbc045bc049aafdc5003e9ea17897007d2bae284b0af87c6f0c16  tools/run_f_formal_prototype_distance_cpu.py
aa16b944549544f1060408fac28d7bf9b5724a2a0fec0e5695391662e7b2359a  tools/f_formal_prototype_distance_cpu_backend.py
7822f5e7f1192ff6f131e9efd5802121695401270a21063fd02e9612044340d3  tests/test_f_formal_prototype_distance_cpu.py
a750a2e986e7312936db4a469ce15c7188a0ac4fcba32a9ef2042f1b9c763270  cti_improvement/research_protocol/F_FORMAL_PROTOTYPE_DISTANCE_CPU_EXECUTION_AUTHORIZATION.json
665d7b5eaf51b9419b9bb02598ebaf758fbd9df1ba208b621041ba6ad32ae124  cti_improvement/research_protocol/F_FORMAL_F02_CPU_EXECUTION_FREEZE_MANIFEST.json
HASHES
CUDA_VISIBLE_DEVICES=-1 /root/autodl-tmp/lgx/conda_envs/unirel38/bin/python tools/run_f_formal_prototype_distance_cpu.py --mode preflight
CUDA_VISIBLE_DEVICES=-1 /root/autodl-tmp/lgx/conda_envs/unirel38/bin/python tools/run_f_formal_prototype_distance_cpu.py --mode execute
CUDA_VISIBLE_DEVICES=-1 /root/autodl-tmp/lgx/conda_envs/unirel38/bin/python tools/run_f_formal_prototype_distance_cpu.py --mode postrun
```

回传 preflight/execute/postrun 轻量结果后，先只读闭合 F02 completion/metrics/predictions 外层 SHA256，再按注册顺序考虑 F03。没有从 F02 单单元授权推导出其他 arm/parent/GPU/full-inner/outer 许可。

## 16. 2026-09-16 F02 完成、F03 授权待准备断点

服务器唯一 F02 fold02/communicates_with/seed13 prototype-distance development-only CPU execute 已完成，postrun=passed_F02_reused_parent_CPU_postrun；execute 内部先调用 preflight。复用原 F00 parent，无 parent 重训、无 unknown optimizer，GPU/outer/锁定评测标志 false。F02 known micro/macro-F1@0.5=0；unknown AUPRC=0.006699469893355654、AUROC=0.8004866180048662、FPR@95TPR=0.31917717319177175、prevalence=0.001986754966887417。postrun 绑定 F02 授权 SHA256=a750a2e986e7312936db4a469ce15c7188a0ac4fcba32a9ef2042f1b9c763270、原 parent completion SHA256=14c404aa31f54af0d71bc2112b6b21af2f9112d4fd1ee7d696b1adf0dde29d38。服务器 F02 外层 completion/metrics/predictions SHA256 尚待只读回传；原 parent/F00/F01/F02 输出不得覆盖或删除，本地未下载复算。

下一步服务器执行命令：

```bash
cd /root/autodl-tmp/lgx/Unirel-main
sha256sum output/f_formal_screening_runs/arms/f-screening-f02-fold02-communicates-with-seed13/RUN_COMPLETION_MANIFEST.json output/f_formal_screening_runs/arms/f-screening-f02-fold02-communicates-with-seed13/inner_heldout_metrics.json output/f_formal_screening_runs/arms/f-screening-f02-fold02-communicates-with-seed13/inner_heldout_predictions.jsonl
```

回传三个 SHA256 后，按冻结顺序准备 F03 learned unknown-head control 的独立 exact-unit CPU 授权、共享 parent 复用与 10-epoch unknown-training backend 合成边界；未授权前没有 F03 execute 命令。禁止其他 arm/parent、GPU、full-inner、outer/锁定评测。三个 unknown 正例及 known-F1=0 不支持当前方法选择或阈值修改。

## 17. 2026-09-17 F03 已完成、F04 独立授权待准备断点

F02 外层 completion/metrics/predictions SHA256 已由服务器回传并记入 CTI_PROJECT_HISTORY.md。唯一 fold02/communicates_with/seed13/F03 learned unknown-head development-only CPU 单元已按独立授权执行，服务器 tiny synthetic CPU 10-epoch regression 返回 F03_TINY_SYNTHETIC_CPU_TRAINING_PASSED；服务器 conda Python 缺少 pytest，因此以标准库 runpy/unittest.mock 调用已冻结的同一合成测试函数。F03 preflight=passed_F03_reused_parent_CPU_preflight，execute=complete_development_only_CPU_F03_learned_unknown，postrun=passed_F03_reused_parent_CPU_postrun；共享原 parent 未重训，unknown optimizer 构造并固定训练 10 epochs，GPU/outer/锁定评测标志 false。PyTorch TypedStorage 弃用提示未造成失败。

权威本地 F03 六文件及 SHA256：tools/run_f_formal_learned_unknown_cpu.py=56069f27c9341060b7e6ce87fcb476c1dfe740fcac91f2b915837eea526eb1cd；tools/f_formal_learned_unknown_cpu_backend.py=bd3d7d85c1d2b17ffe8a017c366f20a65b6574899bb9cfe5421e8ef743712dbb；tests/test_f_formal_learned_unknown_cpu.py=8b1be4ebd945179796b3019e8451c2a3e7664838159aab0382b7f07c717ab8dc；tests/test_f_formal_learned_unknown_training_cpu.py=7eae85eadb0322ce4921eb1b801703dd22c83014f32182b4a1f926ff899567fb；cti_improvement/research_protocol/F_FORMAL_LEARNED_UNKNOWN_CPU_EXECUTION_AUTHORIZATION.json=bfcd2175938c373fdace36f5fc6b1f2704b25b2c5a35bcb973a68ec4aab90fcc；cti_improvement/research_protocol/F_FORMAL_F03_CPU_EXECUTION_FREEZE_MANIFEST.json=638906a110ae23ba954eaa86870052b1135d36ec4dd89253be06f7f7da8477fa，18 项本地冻结绑定通过。F03 postrun 绑定原 parent completion SHA256=14c404aa31f54af0d71bc2112b6b21af2f9112d4fd1ee7d696b1adf0dde29d38，并核验七件套 completion 内部 payload 哈希与 10 行 unknown-train 日志。

F03 固定 inner-heldout 指标：known micro/macro-F1@0.5=0；unknown AUPRC=0.07444005270092227、AUROC=0.9840743198407432、FPR@95TPR=0.019907100199071003、prevalence=0.001986754966887417。仅 3 个 unknown 正例，不得根据单单元差异选 arm 或调阈值。用户服务器只读回传 F03 RUN_COMPLETION_MANIFEST.json SHA256=67fc9f594c56db58617a6d1913c606ac347cdcce01cea3f2b0015783fc4285f4、inner_heldout_metrics.json SHA256=d15b904de1e5a5d0ea5178c1919ae339346b2f8f52af6b3788e6d25a5729b6ee、inner_heldout_predictions.jsonl SHA256=6e6fb89461ebb685431543c29eee0f2c7ce72c8f596a36640858c3413e98248c。外层 SHA 本地未下载独立复算；服务器原 parent/F00/F01/F02/F03 输出须保持原位且不可覆盖。

下一步执行命令：无服务器 F04 命令。先按冻结矩阵准备唯一 fold02/communicates_with/seed13/F04 schema-residual learned unknown-head CPU 单元的独立代码、合成边界测试、精确授权与冻结；需要绑定 F03 完成清单外层 SHA 及既有 parent/F00–F02 原位 provenance。完成本地核验后再给出六文件上传、服务器 synthetic/preflight/单次 execute/postrun 的顺序。禁止 GPU、full-inner、outer/锁定评测、其他 arm/parent 或修改已冻结 v1 结论；不新增标签或将模型辅助材料当人工金标。

## 18. 2026-09-17 F04 schema-residual CPU 单元可上传断点

唯一预注册 fold02/communicates_with/seed13/F04 development-only CPU 单元已本地准备并独立授权，复用原 F00 parent，要求 F00–F03 completion 原位匹配；F04 仅训练 schema-residual unknown head 的固定 10 epochs，不重训 parent，不访问 GPU/full-inner/outer/锁定评测。F03 completion/metrics/predictions 外层 SHA256 已由用户服务器回传并绑定，分别为 67fc9f594c56db58617a6d1913c606ac347cdcce01cea3f2b0015783fc4285f4、d15b904de1e5a5d0ea5178c1919ae339346b2f8f52af6b3788e6d25a5729b6ee、6e6fb89461ebb685431543c29eee0f2c7ce72c8f596a36640858c3413e98248c；外层值未本地下载独立复算。

六个新增文件及 SHA256：tools/run_f_formal_schema_residual_cpu.py=660643dfae27568e3b5a0389a276f34a720b4aa693bd372e8b7b982ce6802e15；tools/f_formal_schema_residual_cpu_backend.py=501c164d53eeab716acb0ce2590acd9df440a918cc6c94ecce352522d7987223；tests/test_f_formal_schema_residual_cpu.py=d91c20385644b7ffefd9e7214d997e52e8e8c2eed10e0a47658137d6d91f176e；tests/test_f_formal_schema_residual_training_cpu.py=59e5b394c409cc8329540bb570e7f5bc737863b1e2cac2578125084c0e7bb0e2；cti_improvement/research_protocol/F_FORMAL_SCHEMA_RESIDUAL_CPU_EXECUTION_AUTHORIZATION.json=14a830b7e08d015139d664d8f5bf3aa1e7817f4819d2604313cba140dd94df28；cti_improvement/research_protocol/F_FORMAL_F04_CPU_EXECUTION_FREEZE_MANIFEST.json=15a460808a52779887642bdd3c6e6e2fb09fa91a4e4ff4a5d02fac22f82e1051。24 项本地冻结绑定与 exact 授权元数据核验通过；3 项标准库合成边界 pytest、Python 3.8 grammar/ruff 通过。tiny torch 10-epoch 合成回归仅写好，服务器尚未运行；本机 torch/NumPy BLAS 异常不适合形式训练。

下一步本地 PowerShell 仅上传六个新增 F04 文件：

```powershell
Set-Location "E:\代码\UniRel-main\UniRel-main"
scp -P 45423 tools\run_f_formal_schema_residual_cpu.py tools\f_formal_schema_residual_cpu_backend.py root@connect.westd.seetacloud.com:/root/autodl-tmp/lgx/Unirel-main/tools/
scp -P 45423 tests\test_f_formal_schema_residual_cpu.py tests\test_f_formal_schema_residual_training_cpu.py root@connect.westd.seetacloud.com:/root/autodl-tmp/lgx/Unirel-main/tests/
scp -P 45423 cti_improvement\research_protocol\F_FORMAL_SCHEMA_RESIDUAL_CPU_EXECUTION_AUTHORIZATION.json cti_improvement\research_protocol\F_FORMAL_F04_CPU_EXECUTION_FREEZE_MANIFEST.json root@connect.westd.seetacloud.com:/root/autodl-tmp/lgx/Unirel-main/cti_improvement/research_protocol/
```

服务器先 `cd /root/autodl-tmp/lgx/Unirel-main`，对上述六文件逐项 sha256sum -c；全匹配后，在 CUDA_VISIBLE_DEVICES=-1 与 `/root/autodl-tmp/lgx/conda_envs/unirel38/bin/python` 下，用标准库 runpy/unittest.mock 运行 `tests/test_f_formal_schema_residual_training_cpu.py` 的 `test_f04_tiny_unknown_head_trains_without_parent_change`，因为服务器该 Python 没有 pytest。合成 test 通过后运行 `tools/run_f_formal_schema_residual_cpu.py --mode preflight`；只有状态 passed_F04_reused_parent_CPU_preflight，才允许单次 --mode execute，随后 --mode postrun。先回传 SHA/tiny/preflight 输出；失败立即停下，保留原 parent/F00–F03 产物。没有从此授权推出其他 arm/parent、GPU、full-inner、outer/锁定评测许可。

## 19. 2026-09-17 F04 完成、F05 provenance 待闭合断点

服务器唯一 fold02/communicates_with/seed13/F04 schema-residual development-only CPU 单元的 tiny synthetic CPU training regression 返回 F04_TINY_SYNTHETIC_CPU_TRAINING_PASSED；preflight=passed_F04_reused_parent_CPU_preflight，execute=complete_development_only_CPU_F04_schema_residual，postrun=passed_F04_reused_parent_CPU_postrun。复用原 parent，未重训 parent；unknown optimizer 构造并固定 10 epochs，GPU/outer/锁定评测标志 false。postrun 绑定 F04 授权 SHA256=14a830b7e08d015139d664d8f5bf3aa1e7817f4819d2604313cba140dd94df28、原 parent completion SHA256=14c404aa31f54af0d71bc2112b6b21af2f9112d4fd1ee7d696b1adf0dde29d38，F04 内部 payload 哈希通过。

F04 指标：known micro/macro-F1@0.5=0；unknown AUPRC=0.08928989139515456、AUROC=0.9869497898694979、FPR@95TPR=0.014598540145985401、prevalence=0.001986754966887417。仅 3 个 unknown 正例，不得按单单元差异选模型或调阈值。F04 completion/metrics/predictions 三项服务器外层 SHA256 未回传，本地未下载独立复算；保留原 parent/F00–F04 所有原始输出。

下一步服务器执行命令：

```bash
sha256sum \
  output/f_formal_screening_runs/arms/f-screening-f04-fold02-communicates-with-seed13/RUN_COMPLETION_MANIFEST.json \
  output/f_formal_screening_runs/arms/f-screening-f04-fold02-communicates-with-seed13/inner_heldout_metrics.json \
  output/f_formal_screening_runs/arms/f-screening-f04-fold02-communicates-with-seed13/inner_heldout_predictions.jsonl
```

回传三项 SHA 后，才能据此绑定并准备唯一 fold02/communicates_with/seed13/F05 mode-consistency control 的独立 CPU 授权；目前没有 F05 execute 命令。禁止其他 arm/parent、GPU、full-inner、outer/锁定评测及覆盖旧产物。

## 20. 2026-09-17 F05 mode-consistency CPU 单元可上传断点

唯一预注册 fold02/communicates_with/seed13/F05 development-only CPU 单元已本地准备并独立授权，复用原 F00 parent，要求 F00–F04 completion 原位匹配；只训练 F05 unknown head 固定 10 epochs，BCE 加 0.1×margin 0.1 consistency，且至少一个训练 episode 必须实际激活一致性项。F04 服务器 completion/metrics/predictions 外层 SHA256 分别为 2eeba8edf7887e409f95734d04cd9cadb528b66c0501a3b3caca2400209b0e0f、78823308268b3dcd72e41e3fe44a3d139d0c5f1ebe95d1752c95151c877cc3ec、8135f0b6f7a49c56b4d0a591318599fdc74d4ecf9fa56bc617c51d18a5b5740a，已绑定；外层值未本地下载独立复算。

六个新增文件及 SHA256：tools/run_f_formal_mode_consistency_cpu.py=8e7de29c592a77f9aad701794414f7e69ec4b5b4aaad17bad1524c7ce2259ae4；tools/f_formal_mode_consistency_cpu_backend.py=b04b16a6729ef64dd97f5fe29495c04382d99fa972f6ebfba60f116ff76192ba；tests/test_f_formal_mode_consistency_cpu.py=655c36816ced1a8b25a92c7f30a00914e96e2c7c3e267539425046a78e4ce806；tests/test_f_formal_mode_consistency_training_cpu.py=0f595a79d47a682fdc5de4fb84d4efaa747ee05a6445954badf9c5074d633d1f；cti_improvement/research_protocol/F_FORMAL_MODE_CONSISTENCY_CPU_EXECUTION_AUTHORIZATION.json=de0105f3d04920a55fd9df1e7198a3eade8438987c805829d8b620900374925c；cti_improvement/research_protocol/F_FORMAL_F05_CPU_EXECUTION_FREEZE_MANIFEST.json=988a8138a840ccb387808f7a00f05fbb5c2e9e45938008cd91122dafd383f8c4。30 项本地冻结绑定、exact 授权元数据、3 项合成边界 pytest、Python 3.8 grammar/ruff 通过。tiny torch 10-epoch 回归已写好但服务器尚未运行；本机 torch/NumPy BLAS 异常未运行正式训练。

下一步本地 PowerShell 仅上传六个新增 F05 文件：

```powershell
Set-Location "E:\代码\UniRel-main\UniRel-main"
scp -P 45423 tools\run_f_formal_mode_consistency_cpu.py tools\f_formal_mode_consistency_cpu_backend.py root@connect.westd.seetacloud.com:/root/autodl-tmp/lgx/Unirel-main/tools/
scp -P 45423 tests\test_f_formal_mode_consistency_cpu.py tests\test_f_formal_mode_consistency_training_cpu.py root@connect.westd.seetacloud.com:/root/autodl-tmp/lgx/Unirel-main/tests/
scp -P 45423 cti_improvement\research_protocol\F_FORMAL_MODE_CONSISTENCY_CPU_EXECUTION_AUTHORIZATION.json cti_improvement\research_protocol\F_FORMAL_F05_CPU_EXECUTION_FREEZE_MANIFEST.json root@connect.westd.seetacloud.com:/root/autodl-tmp/lgx/Unirel-main/cti_improvement/research_protocol/
```

服务器先 `cd /root/autodl-tmp/lgx/Unirel-main` 并对六文件逐项 sha256sum -c；全匹配后用固定 CUDA_VISIBLE_DEVICES=-1 与 `/root/autodl-tmp/lgx/conda_envs/unirel38/bin/python` 通过标准库 runpy/unittest.mock 调用 `tests/test_f_formal_mode_consistency_training_cpu.py` 的 `test_f05_tiny_unknown_head_trains_without_parent_change`，验证 10 epochs、非零 consistency、parent/known logits 不变。服务器该 Python 无 pytest。合成测试通过后运行 `tools/run_f_formal_mode_consistency_cpu.py --mode preflight`；只有返回 passed_F05_reused_parent_CPU_preflight，才单次 execute，随后 postrun。先回传六哈希/合成测试/preflight 输出；失败立即停下、保留原 parent/F00–F04 产物。没有从 F05 授权推出其他 arm/parent、GPU、full-inner、outer/锁定评测许可。

## 21. 2026-09-17 F05 完成、外层 provenance 待闭合断点

唯一 fold02/communicates_with/seed13/F05 mode-consistency development-only CPU 单元服务器 tiny synthetic training regression 返回 F05_TINY_SYNTHETIC_CPU_TRAINING_PASSED；preflight=passed_F05_reused_parent_CPU_preflight，execute=complete_development_only_CPU_F05_mode_consistency，postrun=passed_F05_reused_parent_CPU_postrun。共享原 parent 未重训；unknown optimizer 构造并固定 10 epochs；postrun 验证非零 mode-consistency loss，GPU/outer/锁定评测标志 false。postrun 绑定 F05 授权 SHA256=de0105f3d04920a55fd9df1e7198a3eade8438987c805829d8b620900374925c、原 parent completion SHA256=14c404aa31f54af0d71bc2112b6b21af2f9112d4fd1ee7d696b1adf0dde29d38，F05 completion 内部 payload 哈希通过。

F05 固定 inner-heldout 指标：known micro/macro-F1@0.5=0；unknown AUPRC=0.0644078144078144、AUROC=0.9814200398142005、FPR@95TPR=0.021897810218978103、prevalence=0.001986754966887417。仅 3 个 unknown 正例，本单元 F00–F05 known-F1 全为 0；不得选臂、调阈值或主张 F 假设得到支持。F05 completion/metrics/predictions 三项服务器外层 SHA256 尚待回传，本地未下载独立复算；原 parent/F00–F05 原始产物不得覆盖或删除。

下一步服务器执行命令：

```bash
sha256sum \
  output/f_formal_screening_runs/arms/f-screening-f05-fold02-communicates-with-seed13/RUN_COMPLETION_MANIFEST.json \
  output/f_formal_screening_runs/arms/f-screening-f05-fold02-communicates-with-seed13/inner_heldout_metrics.json \
  output/f_formal_screening_runs/arms/f-screening-f05-fold02-communicates-with-seed13/inner_heldout_predictions.jsonl
```

回传三项外层 SHA 后，再依据 Riichi 预注册矩阵决定下一 development-only exact CPU 单元及其独立授权；当前没有下个 parent/arm execute 命令。禁止 GPU、full-inner、outer/锁定评测及覆盖旧产物。

## 22. 2026-09-17 首组六臂 provenance 闭合、下一 parent 待授权

用户服务器只读回传 fold02/communicates_with/seed13/F05 arm completion SHA256=44cf7ee2dc83a54ac20f9d570f5edb92c95546b825ebbedeaf2d2ee2260c00a4、metrics SHA256=51d7a3aa119be3ff4d737ea23feba93423dc54af8d9d18082528da57af6aa3dd、predictions SHA256=129157bc6dbe524e1039611c3e39eeaf5bfbc7a882b583cfd4d5fb428efdc42f。F05 postrun 已核验内部 payload 哈希；三个外层值未在本地下载后独立复算。首个 parent 与 F00–F05 六臂 provenance 门槛闭合，服务器原始输出必须保留。

预注册 screening 仍是 folds02/03、seed13、三 scheme、各六臂，共 6 parent/36 arm；F05 支持门槛要求等权 fold/scheme 汇总对最强控制 unknown AUPRC 至少 +0.03、known micro-F1 delta 不低于 -0.01、至少 2/3 scheme 为正且无泄漏。当前唯一 fold02/communicates_with 单元 F05 AUPRC 低于 F04，known-F1 全零；不足以提前宣布完整 screening 失败或支持 F，也不得事后改变门槛。按冻结 runner 的矩阵顺序，下一单元是 fold02/targets/seed13/F00，需训练并冻结新的 17 类 parent（parent_id=f-screening-parent-fold02-targets-seed13），不是复用 communicates_with parent。

下一步执行命令：无服务器下个 parent/F00 命令。先本地建立该唯一单元的独立 development-only CPU runner/backend/授权/冻结，必须对原 1 parent/6 arm 原位哈希和唯一新输出范围做 fail-closed 验收；新单元 formal training 与旧 communicates_with F00-only 授权不可互换。服务器只能在新授权和合成边界验证后执行 CPU-only preflight；GPU、full-inner、outer/锁定评测、旧产物覆盖仍禁止。

## 23. 2026-09-17 fold02 targets parent/F00 CPU 单元可上传断点

首个 fold02/communicates_with/seed13 parent 与 F00–F05 六臂 provenance 已闭合。预注册 screening 尚需其余 fold×scheme；F05 支持门槛须按等权 fold/scheme 汇总判断，当前单 scheme F05 低于 F04、known-F1 全零，不能提前宣布完整筛选失败或支持 F。下一矩阵单元是 fold02/targets/seed13/F00，训练并冻结新的 scheme-specific parent，再 eval-only F00；禁止复用旧 communicates_with parent 或旧 F00-only 执行授权。

七个新增 targets parent/F00 文件及 SHA256：tools/run_f_formal_targets_parent_cpu.py=56f1085bd02d01948997c6d24e4c0f7a60011472540741580bbb061b31ad5026；tools/f_formal_targets_parent_cpu_backend.py=423b593122514e9b1e8e214f74b9ce17722d234e287dbd7a76956c03635db07c；tools/f_formal_fold02_targets_cpu_partition.py=c5db0be7b857dbb57fc959cb3adec8c34a625c8e94d70469b0eaa4d5a568ccc8；tests/test_f_formal_targets_parent_cpu.py=d574407fcfb23f4b8c37494e7d2c60e84a72ccdd03029ab28c81f0870aff81fa；tests/test_f_formal_targets_parent_training_cpu.py=c581f23a69f8212866da0501da692b9a794547b007cca6982ce0f61d4526011c；cti_improvement/research_protocol/F_FORMAL_TARGETS_PARENT_CPU_EXECUTION_AUTHORIZATION.json=e439c814f421525c7cbf3a545c7b082482b41e7e876c9aabf51f988033d7c2d6；cti_improvement/research_protocol/F_FORMAL_TARGETS_PARENT_CPU_EXECUTION_FREEZE_MANIFEST.json=24e9a7919007ca75c71556c24d5be7d415bc91a336e18e4ebcabb7aefa3fa203。37 项本地冻结绑定、exact 授权元数据、3 项合成边界 pytest、Python 3.8 grammar/ruff 通过；tiny torch 5-epoch parent 回归尚待服务器运行。

本地 PowerShell 仅按目录上传七个新增文件：

```powershell
Set-Location "E:\代码\UniRel-main\UniRel-main"
scp -P 45423 tools\run_f_formal_targets_parent_cpu.py tools\f_formal_targets_parent_cpu_backend.py tools\f_formal_fold02_targets_cpu_partition.py root@connect.westd.seetacloud.com:/root/autodl-tmp/lgx/Unirel-main/tools/
scp -P 45423 tests\test_f_formal_targets_parent_cpu.py tests\test_f_formal_targets_parent_training_cpu.py root@connect.westd.seetacloud.com:/root/autodl-tmp/lgx/Unirel-main/tests/
scp -P 45423 cti_improvement\research_protocol\F_FORMAL_TARGETS_PARENT_CPU_EXECUTION_AUTHORIZATION.json cti_improvement\research_protocol\F_FORMAL_TARGETS_PARENT_CPU_EXECUTION_FREEZE_MANIFEST.json root@connect.westd.seetacloud.com:/root/autodl-tmp/lgx/Unirel-main/cti_improvement/research_protocol/
```

服务器先进入 `/root/autodl-tmp/lgx/Unirel-main`，对上述七文件逐项 sha256sum -c；全部匹配后固定 CUDA_VISIBLE_DEVICES=-1 和 `/root/autodl-tmp/lgx/conda_envs/unirel38/bin/python`，用标准库 runpy/unittest.mock 调用 `tests/test_f_formal_targets_parent_training_cpu.py` 的 `test_targets_tiny_parent_trains_five_epochs`，再运行 `tools/run_f_formal_targets_parent_cpu.py --mode preflight`。仅合成 5 epoch 训练通过且状态 passed_targets_parent_F00_CPU_preflight 时，才能单次 --mode execute，随后 --mode postrun。先回传七哈希/tiny/preflight 输出；失败立即停下，禁止重试覆盖旧 parent/F00–F05、GPU、full-inner、outer/锁定评测。


## 24. 2026-09-17 targets parent/F00 原型缺失，F screening fail-closed

上一节待执行命令已失效。服务器 tiny synthetic 5-epoch CPU 测试与 exact preflight 通过；单次 `--mode execute` 在五轮 parent 训练后的原型构建处失败：`visible relation prototype has no parent-train positive`。本地对唯一授权的 fold02 train 冻结文件只读统计：6642 原行；targets 严格按报告隔离后保留 3945 行；可见 `impersonates` 正例为 0。原始 4 个 `impersonates` 行均在同一个含 targets 的报告中，必须全部隔离。tiny 测试的每类一个合成正例未覆盖该条件。

预注册 `missing_visible_relation_prototype_policy="fail closed"`，不得用 heldout/被隔离行/其他 parent/伪造原型补齐，也不得改已冻结代码与授权来续跑。代码显示 `_commit("parent", ...)` 在抛错点之后，推断本次没有新 parent/arm 提交；服务器文件系统尚待只读确认。立即暂停现 F 矩阵后续单元、`postrun` 和重试；保留原 communicates_with 一 parent/六 arm 产物。不能将不完整 screening 解释成 F05 支持成功。下一步只读检查服务器 targets parent/F00 输出路径与 partial 状态并补记核验结果；GPU、full-inner、outer/锁定评测仍禁止。


服务器只读核验已闭合：targets parent 路径与 targets F00 arm 路径均不存在，相应 `.incomplete-*` 也均为空。本次异常未留下需处置的正式或 staging 产物；不得清理原 communicates_with parent/六臂。F screening 当前状态为预注册缺失可见关系原型而 fail-closed，后续 F 单元、postrun、重试、full-inner、GPU、outer/锁定评测均停止。


## 25. 2026-09-17 备选 A 跨句可行性审计未通过

用户授权的 development-only、只读备选 A 审计已完成。事前结构定义：正 candidate 的 subject/object 已有 mention sentence-ID 集合不相交；这只是结构 proxy，不是人工 evidence gold。精确输入为 fold02 inner train+heldout，共 8152 candidates、403 文本记录指纹、996 正对。280/996（28.1124%）正对属于结构性跨句，覆盖 83 个文本记录指纹；总量、占比和文本覆盖通过门槛，但只有 `uses`、`indicates`、`targets` 三个关系满足每关系至少 10 对且至少 5 个文本记录，低于事前要求的 5 个关系。因此冻结状态为 `backup_a_feasibility_gate_failed`，不准备备选 A 模型协议、不训练、不打开 outer/锁定评测。

权威产物为 `cti_improvement/research_protocol/BACKUP_A_CROSS_SENTENCE_FEASIBILITY_FREEZE_MANIFEST.json`，SHA256=c83dd0534c1a2b93978b781e4ec13e5b1cee7f710dc2a54a225028d1d4d3288c，7/7 绑定通过；事前协议、审计 JSON 和逐关系 CSV 分别为 `backup_a_cross_sentence_feasibility_protocol.json`、`backup_a_cross_sentence_feasibility_audit.json`、`output/backup_a_cross_sentence_feasibility/relation_support.csv`。3 项合成边界测试、py_compile、ruff 通过。下一步只能讨论获批外部数据，或单独授权备选 B 固定人工预算标注可行性；当前无服务器训练命令。

安全事件必须随交接保留：准备阶段过宽本地搜索意外返回过 Task105 路径行，未用于本审计统计、门槛或结论。实际审计输入仅为两个 exact-bound development inner 文件，但不能宣称本轮会话全程零锁定访问；冻结产物已显式记录该事件。

## 26. 2026-09-17 外部数据准入审计与无人工标注研究断点

CyberEntRel 与 CTINexus 已完成许可证、报告来源字段、标注单位和 18 类映射的只读准入前审计。CyberEntRel 的 Figshare 发布条目为 CC BY 4.0，但部分由 100 篇 public+private 报告生成的 JSON 未公开，本环境也未能读取数据 ZIP，逐报告来源和实际字段未核验；其 11 类联合实体—关系三元组任务不与当前固定候选实体对 18 类多标签任务同构。CTINexus 仓库 MIT 明确覆盖源码，没有独立数据及第三方报告全文许可；本地非评测 demo 的 148 个 JSON 只有 text/explicit_triplets/entities/implicit_triplets，未见逐报告 source/link/url/publisher/date，且 1807 个显式三元组使用 847 种自由文本谓词。两项均不得并入现有 AZERG 冻结数据，当前不申请新的外部数据协议。详细证据见 `CTI_PROJECT_HISTORY.md` 最新审计记录。

在不新增人工标注的约束下，CTI 关系识别大方向仍可继续，但必须收缩研究主张：仅使用现有获准 AZERG 原标签开展 T0 同数据方法、行为诊断和鲁棒性研究；不得把来源迁移、现实未知关系识别、人工证据一致性或标签完备性作为已验证结论。D 无 survivor、E 缺来源元数据、F 与备选 A 数据门槛失败，继续增加训练次数不能修复独立观察单位和监督缺口。下一步是与导师确认是否接受“一个方法问题 + 严格误差/鲁棒性研究”的论文范围；若毕业要求三个独立且正向支持的贡献，则需要调整论文问题或另行批准外部数据/人工评测。本断点没有服务器训练命令，不授权 outer/Task105/锁定评测访问。

# CTI 关系抽取毕业设计研究交接文档

更新日期：2026-09-11  
项目仓库：`E:\代码\UniRel-main\UniRel-main`

## 1. 交接目的

本文档用于帮助新的 Agent 快速理解当前毕业设计的研究问题、已有实验结论、现阶段研究方向和后续优化计划。新 Agent 的首要任务是梳理研究主线、检查现有实验是否足以支撑毕业论文，并在不破坏既有数据隔离与实验协议的前提下规划后续优化。

详细研究过程、历史决策和实验记录以仓库根目录的 `CTI_PROJECT_HISTORY.md` 为准。执行任何代码或实验前，必须先阅读 `AGENTS.md`。

## 2. 毕业设计当前研究方向

本项目研究网络威胁情报（Cyber Threat Intelligence，CTI）中的实体对关系抽取。输入是包含威胁实体的 CTI 文本和候选实体对，模型需要判断有向实体对之间存在哪些关系，例如：

- `uses`
- `targets`
- `communicates_with`
- `exploits`
- `drops`
- `attributed_to`
- `variant_of`

当前任务采用多标签候选对分类形式：同一个有向实体对可以对应零个、一个或多个关系。研究重点不是继续扩大候选实体集合，而是在固定候选对和原始 AZERG 标签下，提高关系判断的准确性、方向区分能力和跨种子稳定性。

当前处于“不新增人工标注”的研究阶段。监督训练只能使用既有 AZERG train/dev 关系标签，不引入新的人工关系金标、外部关系标签或大模型生成标签。

当前具体优化方向是：

> 在现有 CTI 多标签研究父模型中加入关系文本与候选对 token 的联合编码，使模型能够在生成关系 logits 之前，显式建模当前文本与每个关系名称或定义之间的交互。

该方向在当前计划中称为 **Route C：relation-text joint encoding**。

## 3. 研究这个方向的最终目的

### 3.1 模型目标

现有研究父模型主要使用候选对文本的 `[CLS]` 表示和结构特征进行多标签分类，关系名称和关系定义没有进入活动模型的前向计算。Route C 希望验证：关系语义能否通过 token 级联合交互，为不同关系提供更有区分度的文本表示。

该研究需要回答三个问题：

1. 加入关系文本联合编码后，模型是否优于只使用候选对表示的现有方法？
2. 性能提升是否来自真实关系定义的语义内容，而不是新增参数或训练随机性？
3. 提升能否在 seed 13、42、2026 上稳定复现，而不是只出现在单个开发运行中？

### 3.2 毕业论文目标

最终成果应形成一条可以审计和复现的毕业设计研究链：

1. 明确 CTI 有向实体对多标签关系抽取的任务定义。
2. 建立冻结的数据、标签、开发选择和最终评价协议。
3. 给出基线模型及其主要局限。
4. 提出一个与既有失败方法有明确区别的改进机制。
5. 通过名称、真实定义和错配定义消融验证机制与语义内容。
6. 使用三个随机种子报告均值、离散程度和一致性。
7. 仅在预注册门槛通过后执行一次锁定最终评价。
8. 如实记录负结果、适用范围和数据限制，避免把开发集结果表述为独立外部验证。

理想结果是新方法通过三种子工程门槛，并在一次锁定最终评价中证明其相对严格最终模型或匹配基线的实际价值。如果新方法未通过门槛，毕业设计仍应形成有效结论：说明哪些机制在当前数据、模型和候选对设置下没有产生稳定收益，并分析失败原因。

## 4. 当前数据与实验边界

### 4.1 冻结数据

- 训练集：9,986 个候选实体对，来自 508 个文本记录。
- 原始开发集：822 个候选实体对，来自 159 个文本记录。
- 当前模型选择视图 `dev_select`：76 个有向候选对，来自 38 个报告。
- 关系类别：18 类。
- 研究种子：`13`、`42`、`2026`。
- 固定研究解码阈值：`0.5`。

数据与选择清单的哈希绑定保存在：

- `reports/baseline_freeze.json`
- `cti_improvement/manifests/select_disjoint_development.json`
- `cti_improvement/no_new_label_runs/continuation_20260910/selection_contract.json`

### 4.2 不能用于当前训练的材料

以下内容只能用于研究诊断，不能作为训练标签、损失权重或困难样本选择依据：

- 195 条模型辅助审核结果。
- 11 条重点关系边界复核。
- 4 条待人工裁决关系记录。
- R-1～R-4 未经人工批准的关系边界与实体归一规则。
- CTINexus 外部关系标签或 replay 数据。
- MITRE、AnnoCTR 等外部材料中的关系标签。
- 大模型生成或 Agent 推断的关系结论。

相关记录必须继续保持：

```text
diagnosis_only=true
human_gold=false
training_allowed=false
automatic_relabeling_allowed=false
```

### 4.3 锁定评价约束

`Task99` 是当前严格最终模型，不能被开发实验自动替换。锁定 final 在新方法通过 seed13 pilot 和三种子门槛前不得读取或运行。

禁止：

- 根据锁定评价结果调整阈值。
- 根据锁定评价结果选择模型或关系定义。
- 将历史已经读取的内部测试结果用于 Route C 配置选择。
- 在实验失败后追加未预注册配置追逐更好的分数。

## 5. 已完成的主要研究工作

### 5.1 数据与标注审核

- 已完成训练数据恢复、哈希绑定和基础准入检查。
- 已检查训练集与开发集的文本及候选对重叠问题。
- 已固定去重后的 `dev_select`，用于当前唯一的模型选择。
- 已完成模型辅助来源复核、关系定义边界检查和待人工裁决材料整理。
- 人工裁决目前无法完成，因此该路线已经暂停。
- 模型辅助结果没有进入训练数据。

### 5.2 无新增标注 Route A

Route A 研究自动上下文裁剪和训练内一致性。

诊断结果：只有 `12/76` 个 `dev_select` 候选能够进行满足保护条件的安全句子移除，覆盖率过低；上下文干预没有产生稳定收益。因此 Route A 的启动前提不成立，没有进入正式训练。

### 5.3 无新增标注 Route B

Route B 研究有向实体 span pooling 和 attention intersection 局部上下文。

已完成：

- 9,986 个训练候选和 76 个开发候选的 token 对齐检查。
- 三个种子的 epoch-0 等价性检查。
- seed13 pilot。
- seed13、42、2026 三种子训练。
- 预测、指标、输入哈希和准入状态审核。

入选方法是 `attention_intersection`。三种子结果如下：

| Seed | Method micro-F1 | Control micro-F1 | 差值 |
| --- | ---: | ---: | ---: |
| 13 | 0.880000 | 0.853333 | +0.026667 |
| 42 | 0.864865 | 0.876712 | -0.011847 |
| 2026 | 0.864865 | 0.853333 | +0.011532 |

三种子平均结果：

- micro-F1 增量：`+0.008784`
- recall 增量：`+0.008772`
- 固定 macro-F1 增量：`+0.000711`
- micro-F1 增量总体标准差：`0.015843`
- 正增量种子数：`2/3`

预注册要求平均 micro-F1 增量至少达到 `+0.01`。Route B 没有通过门槛，当前状态为：

```text
C5_dev_select_training=complete_gate_failed
C6_locked_final=not_authorized
```

不得继续修改 B2 的阈值、层数、训练预算或追加配置。

相关结果位于：

- `cti_improvement/no_new_label_runs/continuation_20260910/pair_run_selection.json`
- `cti_improvement/no_new_label_runs/continuation_20260910/run_manifest.json`

## 6. 当前 Route C 的研究假设

### 6.1 为什么选择 Route C

选择 Route C 的依据是：

1. Route A 因安全干预覆盖率低而不适用。
2. Route B 已完成三种子评估，但没有达到预注册收益门槛。
3. 当前研究父模型不使用关系名称或定义作为前向输入。
4. 关系语义是否能通过早期联合交互改善 CTI 多标签分类仍未在当前研究父模型上得到验证。

### 6.2 与旧关系定义残差实验的区别

历史失败的 relation-definition residual 分支：

- 冻结父模型。
- 缓存 pooled pair representation。
- 独立编码并冻结关系定义向量。
- 在分类结果后使用小型残差 MLP。

Route C 的冻结设计：

- 使用每个候选对的 token hidden states。
- 为 18 个关系分别构造关系状态。
- 将 384 个文本状态与 18 个关系状态拼接。
- 在生成关系 logits 前执行一层可训练的双向 self-attention。
- 文本能够关注关系状态，关系状态也能够关注当前文本。
- 从联合编码后的关系状态生成逐关系残差。

因此，Route C 检验的是逐样本 token 级关系语义交互，不是重复旧的“冻结向量＋后接残差”方法。

### 6.3 已完成的可行性检查

- 冻结的 18 个标签全部存在于 `data/relation_descriptions/cti_relations.json`。
- 关系名称长度为 1～5 个 WordPiece。
- `关系名称: 现有定义` 长度为 21～35 个 WordPiece。
- 固定关系文本上限为 40，不需要截断定义。
- 原始文本继续保留 384-token 上限。
- 18 个关系文本各池化成一个关系状态，联合层长度为 `384+18=402`。
- backbone position 上限为 512，因此结构可行。

## 7. 已冻结的 Route C 预注册方案

权威预注册文件：

`cti_improvement/no_new_label_runs/relation_text_joint_encoding/preregister_relation_text_joint_encoding.json`

预注册文件 SHA256：

`957ffab70af3ec0cdba5e4c80daa60f6ec35b9f34e5f2b2fca1eaa2234744304`

当前状态：

```text
frozen_before_implementation
```

### 7.1 冻结架构

1. 从研究父模型取得最多 384 个候选对文本 token states。
2. 根据固定关系文本 token ID 构造 18 个 relation states。
3. 将文本状态和关系状态拼接为 402 个 states。
4. 通过恰好一层 Transformer encoder：
   - hidden size 与父模型一致；
   - 12 个 attention heads；
   - FFN size 为 3072；
   - dropout 继承父模型。
5. 对每个联合编码后的关系状态使用共享线性 scorer。
6. scorer 输出作为对应父 relation logit 的残差。
7. 最终 scorer 零初始化，epoch-0 概率最大绝对差必须不超过 `2e-6`。

保持不变的内容：

- 原始 AZERG 多标签目标。
- 父模型 loss 及各损失权重。
- 优化器分组和学习率。
- `candidate_pair_features`。
- `candidate_evidence`。
- `schema_valid_mask`。
- 候选实体对集合。
- `dev_select` 顺序。
- 固定全局解码阈值 0.5。

### 7.2 三个固定变体

只允许三个等预算变体：

| 变体 | 输入 | 作用 |
| --- | --- | --- |
| `C_name` | 正确关系名称 | 结构及短语义控制 |
| `C_definition` | 正确关系名称＋冻结定义 | 拟议方法 |
| `C_permuted_definition` | 正确关系名称＋固定错配定义 | 语义内容控制 |

错配定义采用预注册文件中保存的循环置换映射，固定点为 0。不得重新随机生成映射。

### 7.3 seed13 pilot 门槛

seed13 必须同时满足：

- epoch-0 最大概率差 `<=2e-6`。
- `C_definition - C_name` micro-F1 `>=+0.01`。
- `C_definition - C_permuted_definition` micro-F1 `>=+0.01`。
- 相对两个控制的 recall 差值均 `>=-0.01`。
- 相对两个控制的固定 macro-F1 差值均 `>=-0.01`。

任一条件失败，应记录负结果并终止 Route C。不得修改定义、置换关系、阈值、层数或训练预算后重试。

### 7.4 三种子门槛

只有 seed13 pilot 通过后才能运行 seed42 和 seed2026。三种子必须同时满足：

- `C_definition` 相对 `C_name` 的平均 micro-F1 增量 `>=+0.01`。
- `C_definition` 相对错配定义的平均 micro-F1 增量 `>=+0.01`。
- 相对两个控制都至少有 2/3 个正增量种子。
- 相对两个控制的平均 recall 增量均 `>=-0.01`。
- 相对两个控制的平均固定 macro-F1 增量均 `>=-0.01`。

全部通过后才允许执行一次锁定 final。

## 8. 新 Agent 应优先梳理的问题

新 Agent 不应立即扩大实验范围。首先需要从毕业论文整体角度检查以下问题：

1. 当前研究问题能否清晰表述为“关系语义联合编码是否改善 CTI 有向实体对多标签关系抽取”。
2. Route C 与历史 definition residual、原始 UniRel 模型和其他关系语义模块是否有足够清晰的机制差异。
3. 三个固定变体是否能分别隔离结构增量、正确语义内容和错误语义内容。
4. 当前 38 报告、76 候选的开发视图能支持哪些结论，哪些结论必须保留到锁定 final。
5. 如果 Route C 失败，毕业论文是否已有足够的负结果、诊断和消融形成完整研究故事。
6. 是否需要在不接触锁定数据的前提下增加训练内诊断，例如：
   - 关系注意力熵；
   - 三个变体的预测分歧；
   - 已支持关系的分关系指标；
   - 新增计算量和峰值显存。

如果 Agent 认为需要改变 Route C 的研究假设、配置或门槛，必须在查看 Route C 训练结果前明确关闭当前预注册并建立新的预注册版本。不能先训练再修改研究问题。

## 9. 接下来预期执行步骤

### 阶段一：研究规划复核

1. 阅读 `AGENTS.md`。
2. 阅读本文档和 `CTI_PROJECT_HISTORY.md` 最新记录。
3. 审核 Route C 预注册文件和历史负结果。
4. 输出一份简洁的研究逻辑检查：研究问题、假设、自变量、对照、指标、失败条件和论文可主张范围。
5. 如无原则性问题，确认继续执行已经冻结的 Route C。

### 阶段二：本地实现

按照预注册实现以下功能文件：

- `model/relation_text_joint_encoder.py`
- `tools/train_no_new_label_relation_text_joint.py`
- `tools/summarize_no_new_label_relation_text_joint.py`
- `scripts/run_no_new_label_relation_text_joint.sh`
- `tests/test_relation_text_joint_encoder.py`

不得同时实现未预注册的第四种配置。

### 阶段三：本地验证

正式训练前完成：

- Python 编译检查。
- 标签顺序和关系文本绑定检查。
- 固定错配映射检查。
- tensor shape、padding 和 mask 检查。
- 前向及反向有限数值检查。
- CPU 小样本训练、保存和回载 smoke test。
- seed13 epoch-0 等价性检查。
- 输入哈希和锁定数据隔离检查。

### 阶段四：服务器 seed13 pilot

本地验证通过后，将必要代码上传服务器，在固定 conda 环境中运行：

- `C_name`
- `C_definition`
- `C_permuted_definition`

只使用 `dev_select` 进行 checkpoint 和配置判断。运行结束后，将不含模型权重的指标、预测和清单同步回本地审核。

### 阶段五：门槛裁决

- pilot 失败：记录负结果，停止 Route C，不继续调参。
- pilot 通过：运行 seed42 和 seed2026。
- 三种子失败：停止，不执行锁定 final。
- 三种子通过：冻结模型与配置，申请一次锁定 final。

### 阶段六：毕业论文材料整理

无论 Route C 是否通过，都需要整理：

- 任务和数据协议。
- 基线模型结构。
- Route A、B、C 的选择逻辑。
- Route B 三种子负结果。
- Route C 的结构、对照和消融。
- 多种子均值与离散程度。
- 数据重叠、开发规模和人工标注缺失等限制。
- 可以支持的结论与不能支持的结论。

## 10. 输出和记录要求

- 文件使用功能性名称，不以 Task 编号作为主要文件名。
- 所有实验历史继续追加到 `CTI_PROJECT_HISTORY.md`。
- 不创建多个重复的实验总结文档。
- 每个实验产物必须记录输入哈希、seed、配置、指标和数据访问标志。
- 所有训练结果必须明确包含：

```text
labels_modified=false
new_annotations_used=false
model_assisted_annotations_used=false
external_relation_training_data_used=false
test_data_read=false
```

- 不将模型辅助审核描述为人工金标。
- 不将 `dev_select` 结果描述为独立外部验证。
- 不使用统计显著性措辞解释只有三个种子的工程门槛结果。
- 发现实现与预注册冲突时，停止训练并先记录冲突。

## 11. 建议新 Agent 的首次回复

新 Agent 接手后应先用 1～2 句话确认：

> 当前毕业设计研究的是 CTI 有向实体对多标签关系抽取，在不新增人工标注的约束下，下一条冻结路线是验证关系名称/定义与候选文本的 token 级联合编码是否带来可复现增益。我将先从毕业论文整体研究逻辑审查 Route C 的假设、对照和可主张范围，再决定是否按现有预注册进入实现。

随后给出：

1. 对当前研究主线的理解。
2. Route C 是否足以形成论文贡献的判断。
3. 后续阶段及每阶段验收条件。
4. 主要风险和失败后的论文收束方案。


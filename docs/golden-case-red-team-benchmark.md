# Golden Case 红队验证基准与 AutoResearch 交互设计

## 1. 定位

《数据驱动的电力系统暂态稳定评估方法综述》作为隐藏的 Golden Case，只供红队评估器读取，不直接提供给 AutoResearch 生产端。它用于验证系统能否独立重建一套覆盖充分、结构合理、证据完整、叙事连贯的综述，而不是要求生成文本复写原文。

Golden Case 被抽象为三类基准：

1. **Gold Coverage Map**：核心文献、关键知识单元、证据片段、检索边界与时间截点。
2. **Gold Technology Map（技术分类与谱系）**：技术路线、方法族、子方法、前置依赖、适用条件及其层级与交叉关系。
3. **Gold Narrative Map**：从研究背景到分类综述、横向比较、局限归纳和研究展望的论证链，以及关键论断与证据之间的关系。

> 为防止答案泄漏，每个检查点必须先冻结 AutoResearch 的首次输出并计算基线得分，再向生产端返回诊断。修复后的结果另行计分，不覆盖首次得分。

## 2. 总体交互图

```mermaid
flowchart TB
    G["隐藏 Golden Case<br/>仅红队评估器可见"] --> M["离线构建 Golden Maps<br/>Coverage · Technology · Narrative"]

    subgraph IA["A. 文献与证据覆盖：Step 3–4 后触发"]
        direction LR
        S34["Step 3 文献导入<br/>Step 4 文献卡片"] -->|"冻结<br/>SourceTable + SourceCard"| A["检查点 A"]
        A -. "实时比对" .-> CV["Coverage Map<br/>核心文献召回<br/>知识单元覆盖<br/>证据密度<br/>卡片与元数据准确性<br/>边界与时间截点合规"]
        CV -. "返回<br/>Missing / Weak Evidence / Card Error" .-> A
    end

    M -. "提供隐藏基准" .-> CV
    A -->|"未通过：扩检或重制卡"| S34
    A -->|"通过"| S56

    subgraph IB["B. 知识分类完备性：Step 5–6 后触发"]
        direction LR
        S56["Step 5 方向归类<br/>Step 6 大纲确认"] -->|"冻结<br/>TopicCluster + Outline"| B["检查点 B"]
        B -. "实时比对" .-> TV["Technology Map<br/>方法族与子路线覆盖<br/>技术层级完整度<br/>分类重叠率<br/>孤立证据率<br/>章节证据均衡度<br/>依赖与顺序一致性"]
        TV -. "返回<br/>Missing Category / Overlap / Imbalance" .-> B
    end

    M -. "提供隐藏基准" .-> TV
    B -->|"未通过：重聚类或重构大纲"| S56
    B -->|"通过"| S78

    subgraph IC["C. 综合写作完备性：Step 7–8 后触发"]
        direction LR
        S78["Step 7 章节写作<br/>Step 8 引用检查"] -->|"冻结<br/>Draft + Claim–Citation Map"| C["检查点 C"]
        C -. "实时比对" .-> NV["Narrative Map<br/>叙事功能覆盖<br/>跨文献综合率<br/>比较维度覆盖<br/>Claim–Evidence 支撑率<br/>Research Gap 闭合度<br/>引用语义支持率"]
        NV -. "返回<br/>Unsupported / Unsynthesized / Gap Break" .-> C
    end

    M -. "提供隐藏基准" .-> NV
    C -->|"未通过：重写或修复证据绑定"| S78
    C -->|"通过"| E["Step 9 导出"]

    classDef stage fill:#F8FAFC,stroke:#94A3B8,stroke-width:1.5px,color:#0F172A;
    classDef gateA fill:#DBEAFE,stroke:#2563EB,stroke-width:2.5px,color:#1E3A8A;
    classDef gateB fill:#FFEDD5,stroke:#EA580C,stroke-width:2.5px,color:#7C2D12;
    classDef gateC fill:#F3E8FF,stroke:#9333EA,stroke-width:2.5px,color:#581C87;
    classDef gold fill:#ECFDF5,stroke:#059669,stroke-width:2px,color:#064E3B;

    class S34,S56,S78,E stage;
    class A,CV gateA;
    class B,TV gateB;
    class C,NV gateC;
    class G,M gold;

    style IA fill:#F8FBFF,stroke:#BFDBFE,stroke-width:1px
    style IB fill:#FFF9F5,stroke:#FED7AA,stroke-width:1px
    style IC fill:#FCF8FF,stroke:#E9D5FF,stroke-width:1px
```

### 三类 Golden Map 的元素、评分与 Step 交互

#### Coverage Map：Step 4 主导

Coverage Map 同时读取 Step 3 和 Step 4 的产物，但红队正式触发点位于 Step 4 完成之后。Step 3 决定“有哪些文献进入系统”，Step 4 将文献转化为可比较的 SourceCard、知识单元与证据绑定。由于 Coverage 不能只按论文数量判断，而要按结构化知识是否完整判断，因此整体由 **Step 4 文献卡片**主导；Step 3 是缺口出现后的主要回流位置。

| 元素 / 打分项 | 明确定义与建议评分 | 与 Step 3 文献导入的交互 | 与 Step 4 文献卡片的交互 | 局部主导 |
|---|---|---|---|---|
| 核心文献覆盖 | Golden Case 中的核心文献按重要性赋权；以 DOI、标准化标题或唯一文献 ID 匹配。得分为：已召回核心文献权重 ÷ 全部核心文献权重 | 负责检索、导入、去重和补充核心来源；出现 Missing 时回到 Step 3 | 将已导入文献标记为 core / supporting，并确认其是否形成有效 SourceCard | Step 3；但在 Step 4 后统一验收 |
| 关键知识单元覆盖 | 将研究问题、方法、数据、指标、结论、局限等定义为 Knowledge Unit。得分为：已被证据支持的加权知识单元 ÷ Golden 知识单元总权重 | 为缺失知识单元补充候选来源 | 从全文中抽取、归一化并绑定 Knowledge Unit，是该指标的直接产出步骤 | **Step 4** |
| 主题证据密度 | 每个关键知识单元拥有的独立有效来源数量及来源多样性；不能用同一论文的多个片段重复计数。可按“达到最低证据阈值的知识单元比例”评分 | 扩展证据不足的主题、补充不同作者或来源类型 | 将 SourceChunk 绑定到 Knowledge Unit，识别重复证据和 Weak Evidence | **Step 4** |
| SourceCard 准确性与完整性 | 检查研究问题、方法、数据、实验、结论和局限等必填字段。得分为：正确且有原文依据的字段数 ÷ 被审计字段总数 | 提供可读取的全文、元数据和附件；处理解析失败文献 | 生成结构化卡片并保留字段—原文片段映射；错误时返回 Card Error | **Step 4** |
| 边界与时间截点合规 | 判断文献是否满足任务定义的主题范围、年份、文献类型、语言和质量条件。得分为：合规纳入文献数 ÷ 全部纳入文献数；重大越界可设置硬失败 | 在导入和筛选时执行初步过滤 | 根据完整元数据和卡片内容进行二次审计，防止标题相关但正文越界 | Step 3 + Step 4；最终由 Step 4 验收 |

#### Technology Map：Step 5 主导

Technology Map 在 Step 6 形成候选大纲后触发，但其核心判断对象是 Step 5 产生的技术分类。Step 5 决定技术路线、方法族与子方法的边界，Step 6 负责把这些关系落实为章节层级和叙事顺序。因此整体由 **Step 5 方向归类**主导，章节均衡和顺序问题再定向回流 Step 6。

| 元素 / 打分项 | 明确定义与建议评分 | 与 Step 5 方向归类的交互 | 与 Step 6 大纲确认的交互 | 局部主导 |
|---|---|---|---|---|
| 方法族与子路线覆盖 | 比较 AutoResearch 分类与 Golden Technology Map 的核心方法族、子路线和适用场景。允许命名不同，按概念对齐计算加权覆盖率 | 生成 TopicCluster，决定每个方法族是否被识别；缺失返回 Missing Category | 确认每个重要方法族在大纲中具有明确章节位置 | **Step 5** |
| 技术层级完整度 | 检查“研究方向—方法族—子方法—典型技术”的父子关系是否完整。得分为：正确恢复的关键层级关系 ÷ Golden 关键层级关系 | 构建技术节点及父子、并列和交叉关系 | 将技术层级转换为章、节和小节，避免层级被压平 | **Step 5** |
| 分类重叠率 | 衡量同一知识单元或文献是否无明确分析目的地重复归入多个同级类别。重叠率越低越好，但允许有解释的交叉分类 | 识别同级类别边界，处理语义重复；过高返回 Overlap | 检查重复章节是否承担不同比较任务，否则合并或改写 | **Step 5** |
| 孤立证据率 | 具有较高权重但未被任何技术类别吸收的 SourceCard 或 Knowledge Unit 占比；越低越好 | 为重要证据指定技术归属；出现 Orphan Evidence 时重新聚类或增加类别 | 检查孤立证据是否需要形成新小节或作为跨章节比较证据 | **Step 5** |
| 章节证据均衡度 | 比较各技术方向的重要性权重与其文献数、证据数和预计篇幅是否匹配，避免重要方向过薄或次要方向膨胀 | 提供各 TopicCluster 的重要性和证据预算 | 根据证据预算调整章节层级、合并或拆分章节；失衡返回 Imbalance | **Step 6** |
| 技术依赖与顺序一致性 | 检查基础概念、方法介绍、横向比较、局限和展望之间的前置关系。得分为：满足的关键先后约束 ÷ 全部关键先后约束 | 标记技术之间的依赖、替代和演进关系 | 将这些关系落实为大纲顺序；冲突返回 Sequence Conflict | **Step 6** |

#### Narrative Map：Step 7 主导

Narrative Map 在 Step 8 完成引用检查后触发。Step 7 决定文章是否形成背景—分类—比较—局限—Research Gap—展望的完整论证，Step 8 负责判断引用能否真实支持这些论断。因此整体由 **Step 7 章节写作**主导；证据缺失和引用错配再回流 Step 8。

| 元素 / 打分项 | 明确定义与建议评分 | 与 Step 7 章节写作的交互 | 与 Step 8 引用检查的交互 | 局部主导 |
|---|---|---|---|---|
| 叙事功能覆盖 | 检查研究背景、技术分类、方法比较、适用条件、局限、Research Gap 和展望等必要叙事功能是否出现且承担实际论证作用。得分为：已完成且有依据的功能数 ÷ 必要功能总数 | 生成并组织各类叙事段落，避免只有背景和方法罗列 | 确认承担关键功能的段落具有充分来源 | **Step 7** |
| 跨文献综合率 | 在分析性段落中，真正联合两篇及以上文献进行归纳、比较或解释的段落比例；简单连续引用不计为综合 | 将多篇文献围绕共同问题或比较轴写入同一论证 | 检查综合句中的每个引用分别支持哪一部分判断 | **Step 7** |
| 比较维度覆盖 | Golden Narrative Map 预定义性能、数据需求、可解释性、适用场景、局限等关键比较轴。得分为已被系统比较且有证据的加权比较轴比例 | 围绕稳定比较轴组织方法对照，避免每节采用不同评价标准 | 检查各比较维度是否有对应证据，不允许无来源评价 | **Step 7** |
| Claim–Evidence 支撑率 | 关键 Claim 至少具有一个语义相关且强度足够的 SourceChunk。得分为：被充分支持的关键 Claim ÷ 全部关键 Claim | 生成可审计的 Claim，并控制论断强度 | 建立 Claim–Citation Map；缺少证据时返回 Unsupported | **Step 8** |
| Research Gap 推导闭合度 | 每个 Gap 必须能回溯到前文的现状、比较结果、已知局限及其研究后果。得分为：具有完整推导链的 Gap ÷ 全部 Gap | 从前文综合结果推导 Gap，而不是在结尾突然提出新观点 | 检查推导链各环节是否有证据；断裂返回 Gap Break | **Step 7** |
| 引用语义支持率 | 引用文献必须真实支持所在句子的对象、方向、范围和结论强度。得分为：语义支持成立的引用关系 ÷ 被审计引用关系 | 保持句子边界和 Claim 表述清晰，避免一个引用承担多个无关判断 | 对 Citation–Claim 关系做语义核验；错配返回 Citation Mismatch | **Step 8** |

### 检查点交互协议

图中的“实时比对”是**阶段完成后的事件触发式交互**，不是在 Agent 写作过程中持续窥视或逐 token 干预。每个检查点执行同一套协议：

1. **冻结快照**：对应步骤完成后，保存不可覆盖的首次版本。
2. **记录首次得分**：红队使用隐藏 Golden Map 评估首次独立输出。
3. **返回诊断**：只返回指标得分、问题标签和问题位置，不提供 Golden Case 原文或标准答案。
4. **执行路由**：未通过则按诊断类型回到指定步骤；通过才开放下一阶段。
5. **重新评估**：修复版本重新进入同一检查点，记录修复后得分与修复增益。
6. **保留轨迹**：首次得分、修复后得分、迭代次数和每次诊断均进入 Red-Team Trace。

```text
阶段产物冻结
    → Golden Map 隐藏式实时比对
    → 首次得分 + 诊断标签
    → 未通过：定向回流 / 通过：进入下一阶段
    → 修复版本重新评估并记录增益
```

## 3. A、B、C 与生产流程的交互

| 检查点 | 触发时点 | 冻结快照 | 实时比对对象 | 诊断标签 | 定向回流 | 通过后的动作 |
|---|---|---|---|---|---|---|
| A. 文献与证据覆盖 | Step 3–4 完成后 | SourceTable + SourceCard；检索日志、去重结果和证据片段作为审计附件 | Coverage Map | `Missing`、`Weak Evidence`、`Card Error` | Missing 或 Weak Evidence 回 Step 3 扩检；Card Error 回 Step 4 重新抽取 | 开放 Step 5–6 |
| B. 知识分类完备性 | Step 5–6 完成后 | TopicCluster + Outline；文献—主题映射和章节证据预算作为审计附件 | Technology Map | `Missing Category`、`Overlap`、`Orphan Evidence`、`Imbalance`、`Sequence Conflict` | 分类缺失、重叠或孤立证据回 Step 5；章节失衡或顺序冲突回 Step 6 | 开放 Step 7–8 |
| C. 综合写作完备性 | Step 7–8 完成后 | Draft + Claim–Citation Map；SourceChunk、引用位置和比较句作为审计附件 | Narrative Map | `Unsupported`、`Unsynthesized`、`Citation Mismatch`、`Gap Break` | Unsynthesized 或 Gap Break 回 Step 7；Unsupported 或 Citation Mismatch 回 Step 8，必要时同步修改 Step 7 的论断 | 开放 Step 9 导出 |

## 4. 三个检查点的具体交互逻辑

### A. 文献与证据覆盖

A 在 Step 4 产出全部文献卡片后触发。生产端冻结 SourceTable 和 SourceCard，红队再将其中的文献、概念和证据片段映射到 Coverage Map。

红队主要判断：

- 是否召回了 Golden Case 中具有高权重的核心文献；
- 是否覆盖关键知识单元，而不是只找到标题相似的论文；
- 每个重要主题是否拥有足够且相互独立的证据；
- SourceCard 是否正确提取研究问题、方法、数据、结论和局限；
- 检索范围、文献类型和时间截点是否满足任务约束。

A 的反馈必须具有可路由性：

- `Missing`：缺少核心主题或文献族，回到 Step 3 扩检；
- `Weak Evidence`：主题已经出现但证据不足，回到 Step 3 补充来源；
- `Card Error`：来源存在但卡片抽取错误，回到 Step 4 重新制卡。

A 通过后，说明系统已经形成足以支持综述构建的证据底座，但尚不代表分类和写作已经完整。

### B. 知识分类完备性

B 在 Step 6 形成候选大纲后触发。生产端冻结 TopicCluster 和 Outline，红队将其与 Technology Map 做概念级对齐。

这里不要求章节标题或分类标准与 Golden Case 完全相同。只要另一套分类能够覆盖相同的核心技术空间，并清楚表达方法之间的层级、交叉、依赖和适用条件，就可以被判定为合理。

B 的诊断与回流关系为：

- `Missing Category`：关键方法族没有对应位置，回到 Step 5 补充或重组方向；
- `Overlap`：多个类别边界模糊且重复吸收同一证据，回到 Step 5 重新聚类；
- `Orphan Evidence`：重要文献或证据没有归属，回到 Step 5 重新映射；
- `Imbalance`：章节体量与证据预算明显失衡，回到 Step 6 调整层级；
- `Sequence Conflict`：前置概念、方法比较和结论顺序不合理，回到 Step 6 重构大纲。

B 通过后，说明文献集合已经被组织成覆盖充分、边界清晰、顺序合理的技术知识地图。

### C. 综合写作完备性

C 在 Step 8 完成引用检查后触发。生产端冻结 Draft 和 Claim–Citation Map，红队通过 Narrative Map 检查文章是否完成从“材料组织”到“学术论证”的转换。

红队不比较与 Golden Case 的文本相似度，而检查：

- 背景、分类、比较、局限、Research Gap 和展望等叙事功能是否齐全；
- 是否对多篇文献进行综合，而不是逐篇罗列；
- 是否围绕一致的比较维度说明方法的差异、适用条件和代价；
- 每个关键论断是否能够追溯到支持它的证据片段；
- Research Gap 和未来方向是否由前文比较与局限自然推出。

C 的反馈按照问题性质分流：

- `Unsynthesized`：只有文献罗列、缺少综合判断，回到 Step 7 重写；
- `Gap Break`：Research Gap 与前文证据链断裂，回到 Step 7 重构论证；
- `Unsupported`：关键论断缺乏充分证据，回到 Step 8 补充绑定，必要时收缩或改写论断；
- `Citation Mismatch`：引用不能支持所在句子，回到 Step 8 更换证据或修订引用位置。

C 通过后才允许 Step 9 导出，表示这份综述同时满足叙事完整性与证据可追溯性。

## 5. 评分记录与使用边界

每个检查点至少保留四类运行记录：

| 记录项 | 含义 |
|---|---|
| First-pass Score | Agent 未获得任何 Golden 诊断时的独立完成能力 |
| Repaired Score | Agent 接收诊断并完成修复后的最终质量 |
| Repair Gain | Repaired Score 与 First-pass Score 的差值，衡量自我修复能力 |
| Iteration Count | 达到通过条件所需的回流次数，衡量流程效率与稳定性 |

同领域与跨领域测试必须采用不同的对齐边界：

| 使用场景 | 可使用的 Golden Benchmark | 不应强制比较的内容 |
|---|---|---|
| 同领域回放测试 | Coverage、Technology、Narrative 三类 Map；可比较核心文献、知识单元、技术路线和论证链 | 不要求章节标题、分类方式和文字表达完全一致 |
| 跨领域 AutoResearch | Coverage 的通用证据规则、Technology 的分类质量指标、Narrative 的叙事与证据规则 | 不比较具体技术方向、具体文献和 Golden Case 的领域结论 |

因此，Golden Case 提供的是**基准相对完备性**，而不是对某一领域“绝对无遗漏”的证明。红队最终至少区分三类主问题：

- **Missing**：重要内容或证据没有覆盖；
- **Misorganized**：内容已经存在，但分类、层级或顺序不合理；
- **Unsupported**：已经形成论断，但证据或引用不足。

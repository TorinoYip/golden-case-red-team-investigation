# 求是引擎（Qiushi Engine）架构调研

> 定位：面向真实科研过程的长程自主科学发现系统。它不是单独训练的基础大模型，而是建立在大模型、多个研究 Agent、结构化记忆、数字工具和物理实验接口之上的科研 Agentic System。

## 1. 项目入口

| 资源 | 链接 | 说明 |
|---|---|---|
| 官方技术页 | [Oxelra：求是引擎](https://oxelra.com/product/) | 系统目标、双层架构、Agent 分工、Meta-Trace 与实验接口 |
| 官方研究页 | [真实光学平台研究](https://oxelra.com/demo/) | 三项递进式实验、数据与部分复现边界 |
| 技术论文 | [End-to-end autonomous scientific discovery on a real optical platform](https://arxiv.org/abs/2604.27092) | 求是引擎在真实自由空间光学平台上的系统验证 |
| 第三方评测 | [ResearchClawBench](https://github.com/InternScience/ResearchClawBench) | 端到端自主科研评测框架与公开任务 |
| 实时榜单 | [ResearchClawBench Leaderboard](https://internscience.github.io/ResearchClawBench-Home/) | 求是引擎的多学科数字科研任务结果 |

## 2. 它重点解决什么问题

求是引擎关注的不是“大模型能否回答一个科研问题”，而是“大模型能否把一个开放科研问题持续推进下去”。其系统设计直接针对三类限制：

| 限制 | 常见科研 Agent 的问题 | 求是引擎的处理方式 |
|---|---|---|
| Workflow-bound | 固定执行 `planner → coder → executor → reviewer`，中途证据变化时难以改变方向 | 将角色与研究阶段解耦，依据当前证据动态选择下一步 |
| Environment-bound | 主要运行在文本、代码或高度受控的数字环境中 | 同时接入数字执行环境和经过工程化封装的真实仪器接口 |
| Horizon-bound | 数十次调用后结束，长程运行容易出现上下文膨胀和方向漂移 | 通过 Meta-Trace、轨迹管理和逐步交接维持千步级研究状态 |

## 3. 官方架构图

![求是引擎双层多智能体架构](https://oxelra.com/figures/fig1a-architecture.png)

> 图源：[Oxelra 求是引擎官方技术页](https://oxelra.com/product/)。图中核心研究 Agent 负责维持研究主线，支持 Subagent 在隔离上下文中提供检索、探索、核验和记忆服务；下层共享数字执行环境与物理实验接口。

## 4. 双层多智能体架构

求是引擎将“做研究决策的主线”与“为研究提供服务的辅助任务”分开。辅助 Agent 不把检索日志、工具输出和完整推理过程直接倒入主上下文，而是通过结构化请求接受任务，再返回压缩后的结果。

### 4.1 四个核心研究 Agent

| 核心 Agent | 主要职责 | 典型输出 |
|---|---|---|
| Lead Investigator | 全局规划、假设形成、研究方向选择和轨迹控制 | 研究目标、候选方向、优先级、下一步决策 |
| Method Builder | 理论到方法的转换、算法设计和稿件构建 | 方法方案、模型、实验设计、论文结构 |
| Experimentalist | 代码执行、仿真、物理测量和数据分析 | 脚本、实验记录、数据、图表和分析结果 |
| Critical Reviewer | 对证据、主张、风险与局限进行对抗式检查 | 证据不足项、反例、边界条件、修改或补验要求 |

这四个角色不是固定流水线。任何角色都可以根据证据状态进入探索、执行或表达阶段。

### 4.2 五个支持 Subagent

| 支持 Subagent | 主要职责 | 为什么需要上下文隔离 |
|---|---|---|
| Knowledge Retriever | 本地与在线文献检索 | 原始检索结果数量大，只向主线返回相关且压缩后的知识 |
| Hypothesis Explorer | 并行探索替代假设 | 避免未经筛选的支线假设扰乱当前研究方向 |
| Evidence Verifier | 检查证据是否真正支持主张 | 为核心 Agent 提供相对独立的证据审计 |
| History Analyst | 回顾已有研究证据、失败和判断 | 按问题检索历史，而不是把全部历史重新塞入上下文 |
| Trajectory Manager | 维护、压缩和交接长程研究状态 | 防止跨数百或数千步运行时目标和证据链漂移 |

## 5. 角色与研究阶段解耦

系统使用三个非线性研究阶段，但不把阶段固定分配给某一个 Agent：

| 研究阶段 | 包含的活动 |
|---|---|
| Explore | 文献解读、假设生成、理论映射、可观测量与验证方案设计 |
| Execute | 代码实现、仿真、物理实验、数据获取和数据分析 |
| Express | 图表构造、稿件撰写、证据综合和对抗式审查 |

四个核心 Agent 与三个研究阶段形成 `4 × 3 = 12` 种角色—阶段组合。研究路径由证据决定：失败的测量可以使系统从 Execute 返回 Explore；草稿中的无依据主张可以使系统从 Express 返回 Execute 补充实验或分析。

```text
当前研究状态
      ↓
证据是否足以支持当前判断？
      ├─ 否：回到 Explore，调整假设、方法或观测量
      ├─ 部分支持：进入 Execute，补充代码、仿真或实验
      └─ 支持：进入 Express，综合证据并形成可审查结论
                    ↓
              Critical Reviewer
                    ↓
           通过 / 降低主张强度 / 再验证
```

## 6. Meta-Trace：结构化长程科研记忆

Meta-Trace 不是聊天记录摘要。每个 Agent Step 结束时，系统把当前研究状态压缩成一个能够交给下一位 Agent 的结构化单元，同时将原始脚本、数据、图表、实验参数和工具日志作为可审计 Artifact 单独保存。

| Meta-Trace 字段 | 含义 |
|---|---|
| What attempted | 本步尝试解决什么问题、执行了什么动作 |
| What found | 得到哪些结果、观察或新问题 |
| Evidence supports | 哪些证据支持当前判断，支持强度如何 |
| Limitations | 仍未解决的问题、适用边界和潜在反例 |
| Artefacts | 本步产生的脚本、数据、图表、报告和实验记录 |
| Next handoff | 下一位 Agent 的目标、输入、约束和建议动作 |

建议的最小表示：

```json
{
  "step_id": "step_039",
  "role": "Critical Reviewer",
  "phase": "Execute",
  "attempted": "检查当前测量能否支持更强的机制主张",
  "findings": ["主要效应成立", "更强主张证据不足"],
  "evidence_refs": ["measurement_018", "figure_012"],
  "limitations": ["缺少针对替代解释的对照实验"],
  "artifact_refs": ["analysis_v3.py", "review_note_039.md"],
  "next_handoff": "设计区分两种解释的补充实验"
}
```

## 7. 数字执行与物理接口

| 基础设施层 | 能力 | 设计原则 |
|---|---|---|
| Digital Execution | 文件访问、代码执行、数据存储、模型仿真、图表与报告生成 | 所有中间产物版本化，并与产生它们的 Agent Step 绑定 |
| Physical Interface | 激光控制、光场调制、功率读取、相机成像等 | Agent 只调用经过校准的结构化操作，不直接处理脆弱的底层硬件细节 |

这层设计的关键不是“让大模型直接控制所有设备”，而是先把设备封装为输入、输出、校准边界和失败状态明确的科研工具，再允许 Agent 调用。

## 8. 最核心的五种构建方式

1. **主线—服务分层**：核心 Agent 负责科研判断，支持 Agent 负责检索、探索、核验与记忆，降低主上下文噪声。
2. **角色—阶段解耦**：Agent 角色不等于流水线步骤，系统可依据证据在 Explore、Execute、Express 之间非线性回流。
3. **Meta-Trace 交接**：每一步都形成结构化研究状态，原始日志和可用结论分离保存。
4. **Evidence-first 闭环**：下一步任务由证据缺口、失败结果和 Reviewer 诊断触发，不由预设流程机械决定。
5. **工具接口工程化**：代码、数据、仿真和物理设备均通过边界清晰、可记录、可复现的接口提供给 Agent。

## 9. 与当前 AutoResearch 的直接对应

| 求是引擎机制 | 当前 AutoResearch 可借鉴的位置 | 建议映射 |
|---|---|---|
| 核心/支持双层架构 | Step 3–8 的检索、分类、写作和核验 | 生产 Agent 保留研究主线；检索、历史回顾和证据核验使用隔离 Subagent |
| Meta-Trace | V2 Research Trace Graph | 每一步保存 Attempt–Finding–Evidence–Limitation–Artifact–Handoff |
| Critical Reviewer | A/B/C 红队检查点 | Reviewer 输出可路由诊断，不直接提供 Golden Case 答案 |
| 非线性阶段回流 | A/B/C 未通过后的修复分支 | 根据诊断回到文献、分类、写作或引用步骤，而不是整条流程重跑 |
| Artifact 体系 | SourceCard、TopicCluster、Outline、Draft、Claim–Citation Map | 所有产物版本化并绑定生成动作、证据来源和检查点结果 |

## 10. 公开边界

根据官方技术页与研究页，目前公开的是系统设计、论文、实验结果，以及用于结果核验和部分复现的研究级数据与分析材料。求是引擎核心系统、完整 Prompt/调度实现和硬件控制软件尚未公开，因此目前没有可直接部署的官方 Qiushi Engine GitHub 仓库。

ResearchClawBench 是第三方开源评测框架，不是求是引擎源码。其公开记录显示，榜单中的 Qiushi 使用 GPT-5.5 作为基础模型；成绩应理解为“基础模型 + 求是引擎编排与工具系统”的整体表现。

## 11. 一句话结论

求是引擎最值得迁移的不是光学实验本身，而是：**用双层 Agent 隔离噪声，用角色—阶段解耦支持非线性研究，用 Meta-Trace 保持长程状态，再用独立证据核验驱动下一轮研究动作。**

---

资料整理日期：2026-07-15。本文为基于公开网页与论文的结构化研究笔记；架构图版权归原作者及 Oxelra 所有，仓库仅通过官方链接引用。

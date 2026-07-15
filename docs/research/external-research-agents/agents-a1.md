# Agents-A1 外部项目调研

> **性质说明：** 本页是对 InternScience / 上海人工智能实验室 Agents-A1 的外部调研，不是本项目 AutoResearch 的模型或系统说明。

## 1. 项目入口

| 资源 | 链接 | 说明 |
|---|---|---|
| 技术报告 | [Agents-A1: Training Scientific Agents through Knowledge–Action Graphs](https://arxiv.org/abs/2606.30616) | 方法、训练数据构建与评测结果 |
| 评测与代码仓库 | [InternScience/Agents-A1](https://github.com/InternScience/Agents-A1) | 官方公开仓库 |
| 模型与资源 | [InternScience/Agents-A1](https://huggingface.co/InternScience/Agents-A1) | Hugging Face 模型页面 |

## 2. 核心定位

Agents-A1 面向科学研究与复杂 Agent 任务，重点不是单次问答，而是让小参数模型学习“知识判断—工具动作—环境反馈—继续决策”的长程研究过程。其关键组织方式是 **Knowledge–Action Graph（KAG）**：把答案、知识依据、工具调用和中间反馈组织成可学习、可检查的研究轨迹。

## 3. Knowledge–Action Graph 的主要对象

| 对象 | 定义 | 对 AutoResearch 的启发 |
|---|---|---|
| Knowledge / Evidence | 支持当前判断的知识、文献或结构化证据 | 文献卡片和 Claim–Citation 关系不能脱离来源 |
| Action | 检索、读取、计算、调用工具或修改方案等可执行动作 | 每个 Step 应记录“为什么做、做了什么” |
| Observation | 工具或环境返回的结果，包括成功、失败与异常 | 失败结果也应成为可回放研究资产 |
| Judgment / Verifier | 对结果正确性、相关性和下一步价值的判断 | 红队诊断应绑定证据，并路由到具体修复步骤 |

## 4. 训练与轨迹组织方式

| 机制 | 作用 | 适合借鉴的层级 |
|---|---|---|
| 多领域数据与任务管线 | 覆盖搜索、科学、工程和指令遵循等不同任务 | 数据集与评测任务设计 |
| 成功与失败轨迹并存 | 模型既看到有效路径，也学习错误诊断和恢复动作 | 研究过程记录与修复 |
| 多领域教师 | 不同教师提供搜索、科学、工具和指令能力 | 后续模型训练阶段 |
| 统一模型蒸馏/优化 | 将多个教师的行为分布压缩到一个小参数模型 | 模型层，不宜直接等同于当前工作流 |
| KAG 结构化数据 | 把知识与动作之间的关系显式组织起来 | AutoResearch Artifact 与 Trace 层 |

## 5. 当前最值得迁移的两项机制

### 5.1 Research Trace Graph

把 KAG 适配为 AutoResearch 的过程图。最小事件包含：

- Evidence：本步依据的文献、数据或上游产物；
- Action：检索、归类、写作、核验或修复动作；
- Observation：新产物、工具反馈与异常；
- Verifier：检查结果、证据强度及下一步路由。

建议保留 `DERIVED_FROM`、`PRODUCES`、`SUPPORTS`、`VERIFIED_BY`、`REPAIRS`、`SUPERSEDES` 等关系，使文献、分类、大纲、草稿和引用检查能够沿同一条轨迹回放。

### 5.2 版本化失败—修复分支

首次输出冻结为 `v0`。当 A/B/C 检查点发现覆盖、分类或写作问题时，不覆盖原结果，而是创建带诊断依据的修复分支：

```text
首次产物 v0
   ↓ 红队诊断
失败节点 + 证据缺口
   ↓ 路由到对应 Step
修复产物 v1
   ↓ 对比
原始得分 / 修复增益 / 是否引入新问题
```

这能区分“系统首次能力”和“得到反馈后的修复能力”，也能积累真实的失败恢复样本。

## 6. 与当前 AutoResearch 的映射

| Agents-A1 机制 | AutoResearch 位置 | 建议产物 |
|---|---|---|
| Knowledge–Action Graph | Step 3–8 全流程 | Research Trace Graph |
| Evidence 节点 | Step 3 文献导入、Step 4 文献卡片 | Source、SourceChunk、SourceCard |
| Action / Observation | Step 5 方向归类、Step 6 大纲、Step 7 写作 | TopicCluster、Outline、Draft 与事件日志 |
| Verifier | A/B/C 红队检查点、Step 8 引用检查 | 可定位、可路由的诊断记录 |
| 失败—恢复轨迹 | 检查点未通过后的修复流程 | v0/v1 分支及修复增益 |

完整的 V2 落地设计见 [Golden Case 红队基准 V2 第 5 节](../../golden-case-red-team-benchmark-v2.md#5-v2-增量可回放研究轨迹与版本化修复)。

## 7. 迁移边界

当前阶段优先借鉴轨迹表示和失败修复，不直接引入多教师训练。原因是前两者可以在现有 AutoResearch 工作流中独立实现、回放和评测；多教师训练则依赖大规模轨迹数据、教师模型配置和训练基础设施，应作为后续模型层工作单独论证。

---

资料整理日期：2026-07-15。本文为基于项目公开技术报告、GitHub 与 Hugging Face 页面形成的结构化研究笔记。

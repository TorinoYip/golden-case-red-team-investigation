# 外部科研智能体参考

本目录整理与 AutoResearch、长程科研 Agent、科研轨迹和评测机制相关的外部工作。它们用于架构参考，不作为 Golden Case，也不直接参与红队评分。

| 项目 | 核心价值 | 本仓库材料 | 官方入口 |
|---|---|---|---|
| Agents-A1 | Knowledge–Action Graph、长程成功/失败轨迹、多领域教师训练 | [V2 第5节：Research Trace Graph 与版本化失败—修复分支](../golden-case-red-team-benchmark-v2.md#5-v2-增量可回放研究轨迹与版本化修复) | [技术报告](https://arxiv.org/abs/2606.30616) · [GitHub](https://github.com/InternScience/Agents-A1) · [Hugging Face](https://huggingface.co/InternScience/Agents-A1) |
| 求是引擎（Qiushi Engine） | 双层多智能体、角色—阶段解耦、Meta-Trace、证据驱动回流、数字与物理工具接口 | [求是引擎架构调研](qiushi-engine.md) | [官方技术页](https://oxelra.com/product/) · [研究页](https://oxelra.com/demo/) · [论文](https://arxiv.org/abs/2604.27092) |

## 使用原则

- 只迁移与当前 AutoResearch 问题直接相关、能够独立实现和验证的机制；
- 区分公开证据、合理推断和本项目自己的设计决策；
- 外部系统的排名和宣传性表述不直接作为本项目的有效性证据；
- 任何迁移机制都需要映射到明确的 Step、Artifact、检查点和验收条件。

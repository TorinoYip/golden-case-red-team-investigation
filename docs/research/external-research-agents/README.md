# 外部科研智能体调研资料库

> **目录性质说明：** 本目录记录的是外部团队公开的科研智能体与自主科研系统，**不是本项目 AutoResearch 的产品 README、系统实现说明或能力声明**。

这里集中保存可供本项目借鉴的外部架构、训练方法、研究轨迹、评测机制与工程经验。外部项目只作为参考资料，不属于 Golden Case，也不直接参与红队评分。

## 项目索引

| 外部项目 | 研究重点 | 独立调研页 | 官方入口 |
|---|---|---|---|
| Agents-A1 | Knowledge–Action Graph、长程成功/失败轨迹、多领域教师训练 | [Agents-A1 调研](agents-a1.md) | [技术报告](https://arxiv.org/abs/2606.30616) · [GitHub](https://github.com/InternScience/Agents-A1) · [Hugging Face](https://huggingface.co/InternScience/Agents-A1) |
| 求是引擎（Qiushi Engine） | 双层多智能体、角色—阶段解耦、Meta-Trace、证据驱动回流、数字与物理工具接口 | [求是引擎调研](qiushi-engine.md) | [官方技术页](https://oxelra.com/product/) · [研究页](https://oxelra.com/demo/) · [论文](https://arxiv.org/abs/2604.27092) |

## 与本项目的关系

```text
外部科研智能体公开资料
          ↓ 提炼可迁移机制
本目录中的独立调研页
          ↓ 经适配与验证后选用
AutoResearch 设计 / Golden Case 红队基准
```

已经进入本项目方案的内容，会在设计文档中单独说明来源和适配方式。例如 Agents-A1 启发的 Research Trace Graph 与版本化失败—修复分支，见 [V2 第 5 节](../../golden-case-red-team-benchmark-v2.md#5-v2-增量可回放研究轨迹与版本化修复)。

## 收录与使用原则

- 每个外部系统使用独立页面，避免与本项目自身文档混写；
- 明确区分公开证据、合理推断与本项目自己的设计决策；
- 只迁移能够映射到明确 Step、Artifact、检查点和验收条件的机制；
- 外部系统的排名、宣传表述和未公开实现，不直接作为本项目有效性证据；
- 新增项目时同步更新本索引，不在 `docs/research/` 根目录散放项目页面。

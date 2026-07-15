# Golden Case Red Team Investigation

本仓库用于沉淀综述论文的研究蓝本、写作规范与后续迭代材料。

## 核心参考蓝本

当前核心参考文献：

- 《数据驱动的电力系统暂态稳定评估方法综述》（范士雄等）

原文存放于 `references/` 目录。后续综述论文将重点参考其：

- 研究问题的组织与分类方式
- 综述框架和章节层次
- 方法归纳、比较与评价逻辑
- 图表、格式及学术表达
- 整体文风与论证节奏

## 红队验证设计

- [V2：Golden Case 红队验证基准与 AutoResearch 交互设计](docs/golden-case-red-team-benchmark-v2.md)（当前推荐）
- [V1：Golden Case 红队验证基准与 AutoResearch 交互设计](docs/golden-case-red-team-benchmark.md)（基础版本）

设计采用三个检查点：

- A：文献与证据覆盖，对应文献导入与文献卡片；
- B：知识分类完备性，对应方向归类与大纲确认；
- C：综合写作完备性，对应章节写作与引用检查。

Golden Case 只供红队评估器读取。AutoResearch 首次输出应先冻结和计分，再接收诊断反馈并进入可选修复流程。V2 进一步增加 Research Trace Graph 与版本化失败—修复分支，使首次产物、诊断依据和修复增益都可回放。

## 外部参考资料

- [外部科研智能体调研资料库（非本项目）](docs/research/external-research-agents/README.md)

该资料库独立整理 Agents-A1、求是引擎等外部系统，仅用于吸收长程科研 Agent、研究轨迹、证据核验与失败修复机制。目录中的页面不是本项目 AutoResearch 的产品 README 或能力说明，也不作为 Golden Case、红队评分依据。

## 使用边界

该文献是写作与方法蓝本，不代表后续研究将复制其主题、观点或具体表述。后续工作应结合目标问题重新开展文献检索、证据核验、分类设计与原创写作。

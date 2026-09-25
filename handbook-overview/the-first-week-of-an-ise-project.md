# ISE 项目的第一周

本文档的目的：

- 把手册中的内容组织得更便于速查和发现
- 用一条反映工程流程的逻辑主线来呈现内容
- 提供可扩展的层级，让各团队能够共享深度的领域专长


## 项目开始之前

- [ ] 讨论并着手编写团队协议。项目过程中做出的任何流程决定，都更新到这些文档中
  - [工作协议](../agile-development/team-agreements/working-agreement.md)
  - [就绪定义](../agile-development/team-agreements/definition-of-ready.md)
  - [完成定义](../agile-development/team-agreements/definition-of-done.md)
  - [估算](../agile-development/ceremonies.md#estimation)
- [ ] [建立仓库](../source-control/README.md#creating-a-new-repository)
  - 确定仓库结构
  - 添加 README.md、LICENSE、CONTRIBUTING.md、.gitignore 等
- [ ] [建立产品待办列表](../agile-development/advanced-topics/backlog-management/README.md)
  - 在选定的项目管理工具中创建项目（例如 Azure DevOps）
  - 用 [INVEST](https://en.wikipedia.org/wiki/INVEST_(mnemonic)) 原则写好用户故事与验收标准
  - [非功能需求指南](../design/design-patterns/non-functional-requirements-capture-guide.md)

## 第 1 天

- [ ] [规划第一个冲刺](../agile-development/ceremonies.md#sprint-planning)
  - 就冲刺目标和冲刺进度的衡量方式达成一致
  - 确定团队产能
  - 把用户故事分配进冲刺，并拆解为任务
  - 设定在制品（WIP）上限
- [ ] [确定测试框架并讨论测试策略](../automated-testing/README.md)
  - 讨论测试的目的与目标，以及如何衡量测试覆盖率
  - 就单元测试与集成、负载、冒烟测试如何划分达成一致
  - 设计第一批测试用例
- [ ] [确定分支命名规范](../source-control/naming-branches.md)
- [ ] [讨论安全需求，确认密钥不会进入源代码管理](../ci-cd/dev-sec-ops/secrets-management/README.md)

## 第 2 天

- [ ] [搭建源代码管理](../source-control/README.md)
  - 就[提交最佳实践](../source-control/git-guidance/README.md#commit-best-practices)达成一致
  - [ ] [搭建带 Linter 和自动化测试的基础持续集成](../ci-cd/continuous-integration.md)
  - [ ] [安排每日站会，并确定流程负责人](../agile-development/ceremonies.md#stand-up)
  - 讨论目的、目标、参与者和主持方式
  - 讨论时间安排，以及如何开一场高效的站会
- [ ] [如果项目有子团队，建立 Scrum of Scrums](../agile-development/advanced-topics/effective-organization/scrum-of-scrums.md)

## 第 3 天

- [ ] 就[代码风格](../code-reviews/README.md)和[如何分派 Pull Request](../code-reviews/pull-requests.md)达成一致
- [ ] [为 Pull Request 设置构建校验（2 名评审者、Linter、自动化测试）](../code-reviews/README.md)，并就[完成定义](../agile-development/team-agreements/definition-of-done.md)达成一致
- [ ] [确定代码合并策略](../source-control/merge-strategies.md)，并更新 CONTRIBUTING.md
- [ ] [确定日志与可观测性的框架和策略](../observability/README.md)

## 第 4 天

- [ ] [搭建持续交付](../ci-cd/continuous-delivery.md)
  - 确定适合本解决方案的环境有哪些
  - 对每个环境讨论其用途、部署触发时机、部署前审批人、晋升签核
- [ ] [确定版本管理策略](../source-control/component-versioning.md)
- [ ] 就[如何设计功能并开展设计评审](../design/design-reviews/README.md)达成一致

## 第 5 天

- [ ] 进行[冲刺演示](../agile-development/ceremonies.md#sprint-demo)
- [ ] 进行[回顾会](../agile-development/ceremonies.md#retrospectives)
  - 确定所需参与者、如何收集输入（工具）与产出
  - 设定时间线，并讨论主持方式、会议结构等
- [ ] [梳理待办列表](../agile-development/advanced-topics/backlog-management/README.md)
  - 确定所需参与者
  - 更新[就绪定义](../agile-development/team-agreements/definition-of-ready.md)
  - 更新估算，以及[估算](../agile-development/ceremonies.md#estimation)文档
- [ ] [就遇到的问题提交工程反馈](../engineering-feedback/README.md)

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/the-first-week-of-an-ise-project.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。原文中的相对链接按本仓库的目录结构做了调整。
{% endhint %}

# 工程基础检查清单

这份检查清单用于确保我们的项目满足工程基础要求。

## 源代码管理

- [ ] 默认目标分支被锁定。
- [ ] 合并一律通过 PR 完成。
- [ ] PR 引用相关工作项。
- [ ] 提交历史一致，提交信息有信息量（做了什么、为什么）。
- [ ] 分支命名规范统一。
- [ ] 仓库结构有清晰文档。
- [ ] 密钥不出现在提交历史中，也不对外公开。（见[凭据扫描](../ci-cd/dev-sec-ops/secrets-management/credential_scanning.md)）
- [ ] 公开仓库遵循 [OSS 指南](../source-control/README.md#creating-a-new-repository)，参见其中「公开仓库默认分支必须包含的文件」。

更多细节见[源代码管理](../source-control/README.md)

## 工作项跟踪

- [ ] 所有事项都在 AzDevOps（或类似工具）中跟踪。
- [ ] 看板组织有序（泳道、功能标签、技术标签）。

更多细节见[待办项管理](../agile-development/backlog-management.md)

## 测试

- [ ] 单元测试覆盖大多数组件（尽可能 >90%）。
- [ ] 运行集成测试，端到端验证解决方案。

更多细节见[自动化测试](../automated-testing/README.md)

## CI/CD

- [ ] 项目对每个 PR 执行包含自动构建与测试的 CI。
- [ ] 项目使用 CD 管理在 PR 合并前向副本环境的部署。
- [ ] 主分支始终处于可发布状态。

更多细节见[持续集成](../ci-cd/continuous-integration.md)与[持续交付](../ci-cd/continuous-delivery.md)

## 安全

- [ ] 访问权限按最小必要原则授予。
- [ ] 密钥存放在安全位置，不签入代码。
- [ ] 数据在传输中加密（必要时静态也加密），密码经哈希处理。
- [ ] 系统是否按关注点分离划分为逻辑分段？这有助于限制安全漏洞的影响面。

更多细节见[安全](../security/README.md)

## 可观测性

- [ ] 重要的业务与功能事件被跟踪，并采集相关指标。
- [ ] 应用故障与错误被记入日志。
- [ ] 系统健康状况被监控。
- [ ] 客户端与服务端的可观测性数据可以区分。
- [ ] 日志配置无需改动代码即可调整（例如 verbose 模式）。
- [ ] [传入的追踪上下文](../observability/correlation-id.md)会被继续传递，以便排查生产问题。
- [ ] 在 PII（个人可识别信息）方面满足 GDPR 合规要求。

更多细节见[可观测性](../observability/README.md)

## Agile/Scrum

- [ ] 由流程负责人（固定或轮值）主持每日站会。
- [ ] 团队内部对敏捷流程有明确定义。
- [ ] 开发负责人（以及 PO 等）负责待办项的管理与梳理。
- [ ] 团队成员与客户之间建立了工作协议。

更多细节见[敏捷开发](../agile-development/README.md)

## 设计评审

- [ ] 设计评审的流程已写入[工作协议](../agile-development/team-agreements/working-agreement.md)。
- [ ] 解决方案的每个主要组件都做过设计评审并留下文档，包括备选方案。
- [ ] 用户故事和/或 PR 中链接到设计文档。
- [ ] 每个用户故事默认包含一个设计评审任务，在冲刺计划会上决定指派或移除。
- [ ] 邀请项目顾问参加设计评审，或请他们对文档中记录的设计决策给出反馈。
- [ ] 摸清客户流程所要求的所有评审，并提前规划。
- [ ] 非功能需求被清晰记录（见[非功能需求指南](../design/design-patterns/non-functional-requirements-capture-guide.md)）
- [ ] 风险与机会被记录（见[风险/机会管理](../agile-development/advanced-topics/backlog-management/risk-management.md)）

更多细节见[设计评审](../design/design-reviews/README.md)

## 代码评审

- [ ] 团队对代码评审的作用有明确共识。
- [ ] 团队有代码评审检查清单或既定流程。
- [ ] PR 合并的最少评审人数（通常 2 人）由策略强制。
- [ ] PR 合并前要求通过 Linter/代码分析器、单元测试和构建。
- [ ] 有机制保证评审快速周转。

更多细节见[代码评审](../code-reviews/README.md)

## 回顾会

- [ ] 每周/每个冲刺结束时召开回顾会。
- [ ] 团队每周/每个冲刺提出 1-3 个待尝试的实验来改进流程。
- [ ] 实验有负责人，并加入项目待办项。
- [ ] 在里程碑和项目结束时，团队进行更长时间的回顾。

更多细节见[回顾会](../agile-development/ceremonies.md#retrospectives)

## 工程反馈

- [ ] 团队提交阻碍项目成功的业务与技术障碍反馈
- [ ] 改进建议被纳入解决方案
- [ ] 反馈足够详细且可复现

更多细节见[工程反馈](../engineering-feedback/README.md)

## 开发者体验（DevEx）

团队中的开发者能够：

- [ ] 构建/编译源码，验证没有语法错误且能编译通过。
- [ ] 运行全部自动化测试（单元、e2e 等）。
- [ ] 端到端启动，模拟在已部署环境中的运行。
- [ ] 把调试器附加到已启动的解决方案或正在运行的自动化测试上，设断点、单步执行、查看变量。
- [ ] 在 IDE 里按 F5（或等价操作）即可自动安装依赖。
- [ ] 使用本地开发配置值（如 .env、appsettings.development.json）。

更多细节见[开发者体验](../developer-experience/README.md)

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/engineering-fundamentals-checklist.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。原文中的相对链接按本仓库的目录结构做了调整。
{% endhint %}

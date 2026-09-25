# 持续集成

![image](https://user-images.githubusercontent.com/7635865/76624154-c2c12800-6502-11ea-912d-a260c821ac41.png)

我们鼓励工程团队在项目的 Sprint 0 阶段就做前期投入，建立一条自动化、可重复的流水线，持续集成代码并把系统可执行产物发布到目标云环境。每次集成都应当由自动化构建过程验证：断言一整套校验测试通过，并把任何错误暴露给整个开发团队。

我们鼓励团队在为客户端编写任何服务代码**之前**就实现 CI/CD 流水线，这通常发生在 Sprint 0(N)。这样工程团队可以在隔离的环境中开发和测试自己的工作，不影响其他开发者，并在整个合作过程中推行一致的 DevOps 工作方式。

这些[原则](https://martinfowler.com/articles/continuousIntegration.html)直接对应敏捷软件开发生命周期的[实践](https://en.wikipedia.org/wiki/Agile_software_development)。

## 目标

持续集成自动化是软件开发生命周期中不可分割的一部分，旨在减少构建集成错误、最大化开发团队的交付速度。

一条健壮的构建自动化流水线会：

* 加快团队交付速度
* 预防集成问题
* 避免发布日期的最后一刻陷入混乱
* 为本地改动对全系统的影响提供快速反馈环
* 分离构建阶段与部署阶段
* 度量并报告构建失败/成功的指标
* 提升团队内的可见性，促成更紧密的沟通
* 减少人为错误 —— 这大概是自动化构建最重要的部分

## 构建定义由 Git 管理

### 构建项目所需的代码/清单产物，应当保存在项目的 Git 仓库中

* 特定于 CI 提供方的构建流水线定义，应当位于项目的 git 仓库内。

## 构建自动化

自动化构建应当包含以下原则：

### 构建任务

* 构建流水线中的单个步骤，把代码项目编译为单一构建产物。

### 单元测试

* 构建定义包含校验步骤，执行一整套自动化单元测试，确保应用组件符合其设计、行为符合预期。

### 代码风格检查

* 工程团队内的代码必须按约定的编码标准格式化。这类标准让代码保持一致，最重要的是让团队和客户易于阅读和重构。代码风格一致鼓励项目敏捷团队与合作方形成集体所有权。
* 有多种开源代码风格校验工具可供选择（[code style checks](https://github.com/checkstyle/checkstyle)、[StyleCop](https://en.wikipedia.org/wiki/StyleCop)）。手册的[代码评审配方](../code-reviews/recipes/README.md)部分给出了多种语言的 linter 与推荐风格。
* 代码和文档应当尽可能避免使用非包容性语言。遵循[包容性 lint](./recipes/inclusive-linting.md)一节，确保你的项目为团队和客户营造包容的工作环境。
* 我们推荐在流水线的构建阶段引入安全分析工具，例如：代码凭据扫描器、安全风险检测、静态分析等。对 Azure DevOps，你可以通过安装 [Microsoft Security Code Analysis Extension](https://secdevtools.azurewebsites.net/#pills-onboard) 为流水线添加安全扫描任务。GitHub Actions 也支持类似的扩展：[RIPS security scan solution](https://github.com/marketplace/actions/rips-security-scan)。
* 代码标准集中维护在单一配置文件中。构建流水线中应当有一个步骤，断言最新提交的代码符合已知的风格定义。

### 构建脚本入口

* 应当能用一条命令构建整个系统。这在 CI 服务器上或开发者本机上都应成立。

### 不依赖 IDE

* 构建必须能通过独立脚本运行，而不依赖于特定 IDE。构建流水线的目标可以在开发者本机通过他们选择的 IDE 触发，构建过程也应保持足够的灵活性，以便同样能在 CI 服务器上运行。例如，把构建过程 Docker 化就提供了这种灵活性，因为 VS Code 和 IntelliJ 都支持 [docker 插件](https://code.visualstudio.com/docs/containers/overview)扩展。

### DevOps 安全检查

* 在项目早期就引入安全。遵循 [DevSecOps 一节](./dev-sec-ops/README.md)引入安全实践、自动化、工具与架，作为 CI 的一部分。

## 构建环境依赖

### 本地环境自动化安装

* 我们鼓励为所有团队成员维护一致的开发者体验。应当有一份中心化的自动化清单/流程，简化任何软件依赖的安装与配置。这样开发者就能在本地复现与 CI 服务器上相同的构建环境。
* 构建自动化脚本往往要求运行环境的操作系统预装特定软件包与版本。这会带来一些挑战，因为构建过程通常会锁定这些依赖的版本。
* 团队中所有开发者都应当能不受操作系统限制，从本地桌面环境模拟出构建环境。
* 对于使用 VS Code 的项目，利用 [Dev Container](https://github.com/microsoft/code-with-engineering-playbook/blob/main/docs/developer-experience/devcontainers-getting-started.md) 确实有助于在整个团队中统一本地开发体验。
* 设计构建自动化工具链时，应当考虑 Docker、Maven、npm 这类成熟的软件打包工具。

### 记录本地安装步骤

* 本地构建环境的安装配置过程应当有良好文档，开发者容易跟随。

## 基础设施即代码

尽可能把以下内容都当作代码管理：

* 配置文件
* 配置管理（即通过 [terraform](https://github.com/microsoft/cobalt/blob/master/infra/modules/providers/azure/app-service/main.tf#L49) 做环境变量自动化）
* 密钥管理（即通过 [terraform](https://github.com/microsoft/cobalt/blob/master/infra/templates/az-isolated-service-single-region/app.tf#L84) 创建 Azure 密钥）
* 云资源供给
* 角色分配
* 负载测试场景
* 可用性告警/监控规则与条件

把基础设施与应用代码库解耦，能简化工程团队向云原生应用迁移。

像 [Azure DevOps](https://github.com/microsoft/terraform-provider-azuredevops) 这样的 Terraform 资源提供者，让开发者更容易管理构建流水线变量、服务连接和 CI/CD 流水线定义。

### 使用 Terraform 与 Cobalt 的示例 DevOps 工作流

![image](https://user-images.githubusercontent.com/7635865/76626035-652eda80-6506-11ea-8870-6070365f10d6.png)

### 为什么

* 对基础设施做可重复、可审计的改动，更容易回滚到已知良好的配置，也更容易快速扩展到新的阶段和区域，而不用手工接线云资源
* 像 [Cobalt](https://github.com/microsoft/cobalt) 和 [Bedrock](https://github.com/microsoft/bedrock) 这样久经实战、模板化的 IAC 参考项目，能让更多工程团队以快得多的节奉部署安全且可扩展的解决方案
* 把云原生计算的复杂性从应用开发团队抽象掉，从而简化“平移搬迁”场景。

### IAC DevOps：以拉取请求驱动运维

* 基础设施部署过程围绕一个仓库构建，该仓库保存系统/Azure 环境的当前预期状态。
* 对运行中系统的运维改动，通过在该仓库上提交来完成。
* Git 也为审计部署和回滚到前一状态提供了简单的模型。

### 推荐的基础设施模式

* 用 Terraform / ARM / Ansible 模板把基础设施定义为代码
* 模板是可重复的云资源栈，聚焦于与应用伸缩和吞吐需求相匹配的配置集。

### IAC 原则

#### 自动化 Azure 环境

* 所有云资源都通过一套基础设施即代码模板供给。这还包括密钥、服务配置设置、角色分配与监控条件。
* Azure 门户应当只提供环境资源的只读视图。对环境的任何改动都应当只通过 IAC CI 工具链完成。
* 云环境的供给应当是可重复的过程，由检入我们 git 仓库的基础设代码产物驱动。

#### IAC CI 工作流

* 当 IAC 模板文件通过基于 git 的工作流发生变更时，CI 构建流水线会构建、校验目标基础设施环境的当前状态与预期状态，并做协同对账。对这些固定环境的执行计划候选，在流水线进入部署阶段、应用执行计划之前，由云管理员作为门禁检查来评审。

#### 开发者对云资源的只读访问

* Azure 门户中的开发者账户应当对 Azure 中的 IAC 环境资源具有只读访问权限。

#### 密钥自动化

* IAC 模板通过集成了密钥自动化的 CI/CD 系统部署。避免直接在 Azure 门户中修改密钥和/或证书。

#### 基础设施集成测试自动化

* 作为 IAC CI 过程的一部分运行端到端集成测试，检查并验证 Azure 环境已可供使用。

#### 基础设施文档

* 部署与云资源模板拓扑应当在 IAC git 仓库的 README 中有文档记录并被充分理解。
* 本地环境与 CI 工作流的配置步骤应当有文档记录。

## 配置校验

应用使用配置来实现不同的运行时行为，而用文件存储这些设置非常常见。作为开发者，编辑这些文件时可能引入错误，导致应用无法正确启动和/或运行。通过对配置的语法与语义应用校验技术，我们能在应用部署并执行之前发现错误，从而改善开发者（用户）体验。

### 应用配置文件举例

* JSON，支持复杂数据类型与数据结构
* YAML，JSON 的超集，支持复杂数据类型与结构
* TOML，JSON 的超集，且是有正式规范的配置文件格式

### 为什么要将应用配置校验作为单独的步骤？

* **更易调试、节省时间** —— 流水线中有配置校验步骤后，我们不必先运行应用才发现它跑不起来。它省去了部署并运行、等待、然后才发现配置有错的时间。此外，它也省去了翻日志去搞清楚什么失败、为什么失败的时间。
* **更好的用户/开发者体验** —— 一句简单的提醒告诉用户配置中某项格式不对，可能就是“部署成功的喜悦”与“不得不猜哪里出错的那种强烈挫败感”之间的差别。例如当期望的是布尔值时，它可能是 `"True"` 或 `"False"` 这样的字符串，也可能是 `"0"` 或 `"1"` 这样的整数。有了配置校验，我们就能确保其含义对我们的应用是正确的。
* **避免数据损坏与安全泄漏** —— 因为数据来自不可信来源（例如用户或外部 Web 服务），校验输入就特别重要。否则就会冒着执行出错、损坏数据，甚至更糟 —— 容易遭受一整类注入攻击的风险。

### 什么是 JSON Schema？

[JSON-Schema](https://json-schema.org/) 是描述 JSON 数据结构与要求的 JSON 文档标准。虽然叫 JSON-Schema，但由于 YAML 是 JSON 的超集，把这种方法用于 YAML 也很常见。 该 schema 非常简单：指出哪些字段可能存在、哪些必填或可选、它们用什么数据格式。在这个基本前提之上还可以加入其他校验规则，以及供人阅读的信息。元数据存放在同样为 .json 文件的 schema 中。 此外，JSON Schema 在 JSON 校验的所有标准中采用最为广泛，覆盖了很大一部分校验场景。它使用易于解析的 JSON 文档作为 schema，且易于扩展。

### 如何实现 Schema 校验？

实现 schema 校验分为两部分 —— 生成 schema，以及用这些 schema 校验 yaml/json 文件。

### 生成

有两种生成 schema 的方式：

* [从代码生成](https://json-schema.org/tools?query=#code-to-schema) —— 可以复用代码中已有的模型与对象，生成定制化的 schema。
* [从数据生成](https://json-schema.org/tools?query=#data-to-schema) —— 可以拿总体反映配置的 yaml/json 样例，使用各种在线工具生成 schema。

### 校验

JSON Schema 针对不同语言有 30 多个[校验器](https://json-schema.org/tools?query=#validator)，其中 JavaScript 就有 10 多个，所以不必自己写。

## 集成校验

快速发现构建中 bug 的一个有效方式，是尽早投入建立一套可靠的自动化测试，校验系统的基线功能：

### 端到端集成测试

* 在流水线中加入测试，校验构建候选符合自动化的业务功能断言。任何 bug 或损坏的代码都应当在测试结果中报告，包括失败的测试与相关堆栈跟踪。所有测试都应当通过一条命令调用。
* 让构建保持快速。在决定把数据库、外部服务和模拟数据加载这类依赖引入测试环境时，要考虑自动化测试的运行时长。当无法在 CI 服务器上做并行构建时，缓慢的构建往往成为开发团队的瓶颈。对于耗时很长的校验，考虑设置最大超时限制，以便快速失败、保持团队的高效交付。

### 避免检入损坏的构建

* 自动化构建检查、测试、lint 运行等，在把改动提交到源代码管理仓库之前应当在本地验证通过。开发团队应当考虑采用[测试驱动开发](https://martinfowler.com/bliki/TestDrivenDevelopment.html)，以帮助在开发生命周期中尽早发现 bug 和失败。

### 报告构建失败

* 如果构建步骤失败，构建流水线运行状态应当报告为失败，并附上相关日志与堆栈跟踪。

### 测试自动化的数据依赖

* 用于单元测试和端到端集成测试的任何模拟数据集，都应当检入主线仓库。尽量减少构建过程对外部数据的依赖。

### 代码覆盖率检查

* 我们推荐在构建阶段集成代码覆盖率工具。大多数覆盖率工具会在测试覆盖率低于最低阈值（80% 覆盖率）时使构建失败。覆盖率报告应当发布到 CI 系统，以便跟踪随时间的变化趋势。

## Git 驱动的工作流

### 提交即构建

* 对基线仓库的每次提交，都应当触发 CI 流水线创建新的构建候选。
* 构建产物按提交被持续构建、打包、校验并部署到非生产环境。对仓库的每次提交都会产生一次 CI 运行：在集成机器上检出代码、启动构建，并把构建结果通知给提交者。

### 避免注释掉失败的测试

* 避免在主线分支中把测试注释掉。注释掉测试会让我们得到关于构建状态的不正确信号。

### 强制分支策略

* 主分支上应当设置受保护的[分支策略](https://help.github.com/en/github/administering-a-repository/about-protected-branches)，确保在开始代码评审之前 CI 阶段已经通过。代码评审者只会在最新推送的 git 提交通过 CI 流水线后才开始评审拉取请求。
* 损坏的构建应当阻断拉取请求评审。
* 禁止直接向主分支提交。

### 分支策略

* 发布分支应当自动触发把构建产物部署到其目标云环境。更多指导见 Azure DevOps 文档站点的[管理部署](https://learn.microsoft.com/en-us/azure/devops/repos/git/git-branching-guidance?view=azure-devops#manage-deployments)一节。

## 快速且每日交付

> “通过定期提交，每位提交者都能减少冲突的改动数量。检入一周的工作量，就有与其他功能冲突的风险，且可能很难解决。系统中一个区域早期的小冲突，会促使团队成员就他们正在做的改动进行沟通。”

本着透明和拥抱开发团队频繁沟通的精神，我们鼓励开发者以每日节奉提交代码。这种做法提供了功能进展的可见性，也能加速团队内的结对编程。可以考虑以下原则：

### 每个人每天都向 Git 仓库提交

* 每天结束时检入的代码至少应当包含单元测试。
* 检入之前在本地运行构建，避免 CI 流水线失败饱满。你应当查明是什么导致了错误，并尽快解决，而不是把代码提交上去。我们鼓励开发者遵循[精益 SDLC 原则](https://leankit.com/learn/lean/principles-of-lean-development/)。
* 把工作切分成小8块，直接对应业务价值，并增量重构。

## 隔离环境

构建校验的关键目标之一，是在 staging 环境中隔离并定位故障，最小化对线上生产流量的干扰。我们的端到端自动化测试应当在尽可能模拟生产环境的环境中运行，包括一致的软件版本、操作系统、测试数据量模拟、与生产一致的网络流量等。

### 在生产环境的克隆中测试

* 生产环境至少应当复制一份到 staging 环境（QA 和/或预生产）。

### 拉取请求更新触发分阶段发布

* 与拉取请求相关的新提交应当触发一次构建/发布到集成环境。生产环境应当与该过程完全隔离。

### 在固定环境间提升基础设施改动

* 基础设施即代码改动应当在集成环境测试，提升到所有 staging 环境，然后迁移到生产，且对系统用户零停机。

### 在生产环境测试

* 有多种[方法](https://medium.com/@copyconstruct/testing-in-production-the-safe-way-18ca102d0ef1)可以安全生产部署做自动化测试。其中一些包括：
  * 功能开关（feature flag）
  * A/B 测试
  * 流量切换

## 开发者可获取最新发布产物

我们的 DevOps 工作流应当让开发者能够获取、安装并运行最新的系统可执行文件。发布可执行文件应当作为 CI/CD 流水线的一部分自动生成。

### 开发者可以获取最新的可执行文件

* 团队中所有开发者都能获取最新的系统可执行文件。应当有一个众所周知的位置，开发者可以在那里找到发布产物。

### 每个拉取请求或合入主分支都会发布发布产物

## 集成可观测性

对主线构建应用的每次状态变更都应当在团队内可见并被沟通。集中构建与发布流水线失败的日志和状态，对排查损坏构建的开发者至关重要。

我们推荐把 Teams 或 Slack 与 CI/CD 流水线运行集成，这有助于让团队持续关注失败与构建候选状态。

### 持续集成顶层看板

* 现代 CI 提供方都有能力在给定看板上汇总并报告构建状态。
* 你的 CI 看板应当能把构建失败与某个 git 提交关联起来。

### 项目 README 中的构建状态徽标

* 项目根 README 中应当包含构建状态徽标。

### 构建通知

* 你的 CI 过程应当配置为在构建完成后向 Teams / Slack 这类消息平台发送通知。我们推荐创建单独的频道，方便汇总和隔离这些通知。

## 相关资源

* [Martin Fowler 的持续集成最佳实践](https://martinfowler.com/articles/continuousIntegration.html)
* [Bedrock 入门快速指南](https://github.com/microsoft/bedrock#getting-started)
* [Cobalt 快速开始指南](https://github.com/microsoft/cobalt/blob/master/docs/2_QUICK_START_GUIDE.md)
* [Terraform Azure DevOps Provider](https://github.com/microsoft/terraform-provider-azuredevops)
* [Azure DevOps 多阶段流水线](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/multi-stage-pipelines-experience?view=azure-devops)
* [Azure Pipeline 核心概念](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/key-pipelines-concepts?view=azure-devops)
* [Azure Pipeline 环境](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/environments?view=azure-devops)
* [Azure Pipelines 中的产物](https://learn.microsoft.com/en-us/azure/devops/pipelines/artifacts/artifacts-overview?view=azure-devops)
* [Azure Pipeline 权限与安全角色](https://learn.microsoft.com/en-us/azure/devops/pipelines/policies/permissions?view=azure-devops)
* [Azure 环境审批与检查](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/approvals?view=azure-devops\&tabs=check-pass)
* [Terraform 配合 Azure 的入门指南](https://learn.hashicorp.com/terraform?track=azure#azure)
* [Terraform 远程状态 Azure 配置](https://learn.microsoft.com/en-us/azure/terraform/terraform-backend)
* [Terratest —— 单元与集成基础设施测试架](https://terratest.gruntwork.io/)

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/continuous-integration.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

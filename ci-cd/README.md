# 持续集成与持续交付

[**持续集成（CI）**](./continuous-integration.md)是一种工程实践：频繁地把代码提交到共享仓库（理想情况是每天多次），并对其执行自动化构建。这些改动会与系统中其他同时进行的改动一起构建，从而能在多名开发者协作的项目里尽早发现集成问题。因集成失败导致的构建中断，被团队所有开发者视为最高优先级问题，通常要停下来直到修复。

配合自动化测试方法，持续集成还让我们能测试集成后的构建，从而不仅验证代码库仍能正确构建，还能验证它在功能上仍然正确。这也是构建健壮、灵活软件系统的最佳实践。

[**持续交付（CD）**](./continuous-delivery.md) 把[**持续集成**](./continuous-integration.md)的概念向前推进一步：它还会在最终部署环境的副本上测试集成后代码库的部署。这让我们能尽早发现改动带来的、未曾预料到的运维问题，也能发现测试覆盖上的缺口。

这一切的目标是确保主分支始终处于**可发布**状态 —— 也就是说，必要时我们可以从代码库主分支取一次构建直接发布到生产环境。

如果这些概念对你来说还陌生，花几分钟读一下 [Continuous Integration](https://www.martinfowler.com/articles/continuousIntegration.html) 和 [Continuous Delivery](https://martinfowler.com/bliki/ContinuousDelivery.html)。

我们的期望是：在与客户合作的**所有**工程项目中都使用 CI/CD，对我们构建的任何软件系统的每一次改动都进行构建、测试和部署。

如果想更深入地理解这些概念，[Continuous Integration](https://www.amazon.com/Continuous-Integration-Improving-Software-Reducing/dp/0321336380) 和 [Continuous Delivery](https://www.amazon.com/gp/product/0321601912) 这两本书提供了全面的背景。

## 为什么要 CI/CD

* 我们希望软件有自动化的构建与部署
* 我们希望所有组件都有自动化配置
* 我们希望发生灾难时能快速从零重建环境
* 我们希望开发/测试环境始终部署最新版本的代码
* 我们希望有可靠的发布策略，发布规则被所有人清楚理解

## 基本功

* 每个 PR / 主分支的每次更新，都跑一条质量流水线（含 lint、单元测试等）
* 所有云资源（包括密钥与权限）都通过基础设施即代码模板来供给 —— 例如 Terraform、Bicep（ARM）、Pulumi 等
* 所有发布候选版本都通过自动化过程（例如 Azure DevOps 或 GitHub 流水线）部署到非生产环境
* 发布通过自动化过程部署到生产环境
* 发布回滚通过可重复的过程执行
* 发布流水线运行自动化测试，在非生产环境上对发布候选产物做端到端验证

## 工具

### Azure Pipelines

微软内部的工具链让搭建这样的集成与交付体系变得容易。如果你不熟悉它，现在花点时间读一下 [Azure Pipelines](https://azure.microsoft.com/en-us/services/devops/pipelines/)（原名 VSTS）；想看一下实际如何运作的完整演示，可以读 [CI/CD on Kubernetes with VSTS](https://medium.com/@timfpark/application-ci-cd-on-kubernetes-with-visual-studio-team-services-ccacecdea8a5)。

### Jenkins

Jenkins 是开源社区最常用的工具之一。它知名度很高，拥有数百个插件，可以满足各种构建需求。 Jenkins 免费，但需要一台专用服务器。 你可以用这个[模板](https://ms.portal.azure.com/#create/azure-oss.jenkinsjenkins)轻松创建一台 Jenkins 虚拟机。

### TravisCI

Travis CI 对开源项目可以免费使用，但私有项目需要购买企业版。 这个服务非常适合在 GitHub 上验证 PR，因为它轻量、易于配置，无需搭建专用服务器。 它还支持 Build matrix 功能，把构建与测试拆成多部分并行执行，从而加速整个过程。

### CircleCI

CircleCI 对开源项目是免费服务，不需要专用服务器。它同样非常适合在 GitHub 上验证 PR。 CircleCI 还支持工作流、并行，并可以把测试拆分到任意数量的容器上，构建容器中预装了大量的包。

### AppVeyor

AppVeyor 是另一个面向开源项目的免费 CI 服务，它也支持基于 Windows 的构建。

## AI 辅助编写 CI/CD

AI 工具可以加速编写 CI/CD 流水线 YAML、作业和脚本片段，但必须配合明确的护栏使用。

建议的工作流：

* 用 AI 起草 CI/CD 流水线模板或作业步骤作为起点（例如生成一个最小化的 GitHub Actions 工作流）。
* 在安全的非生产环境或 CI 沙箱中运行草稿流水线，验证语法与基本行为。
* 要求人工评审者按正确性、幂等性和安全影响（尤其是密钥、权限和外部 action 相关）逐项验证生成的步骤。
* 为流水线添加测试或冒烟检查，使流水线运行时能自动验证改动。
* 把审核通过的模板提升到中心位置（例如 `.github/workflows/` 或共享的流水线模板仓库），让团队复用经过审核、审计过的流水线。

护栏与检查清单（合并 AI 生成的流水线改动之前）：

* [ ] 已完成人工评审并在 PR 中记录
* [ ] 没有硬编码任何密钥或凭据
* [ ] 必需的 lint 与语法检查在本地和 CI 中都通过
* [ ] 安全与许可证扫描已运行且未报告严重问题
* [ ] 流水线步骤是幂等的，且在适用处有明确的回滚策略
* [ ] 生成内容已在 PR 描述中标注（例如“AI 辅助草稿”），让评审者知道要额外把关

注意事项：

* AI 生成的流水线非常适合减少样板代码、加速迭代，但它不能取代领域知识和安全评审。
* 维护一小组经过审核的流水线模板，以降低风险、提高可复现性。

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/README.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

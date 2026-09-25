# 证据与度量

## 证据

许多代码质量保证项都可以在现代版本控制与工作项跟踪系统中自动化，或通过策略强制执行。例如，在 [Azure DevOps](https://azure.microsoft.com/en-us/services/devops/)（AzDO）或 [GitHub](https://github.com/) 上验证主分支的策略，或许就足以证明项目团队确实在进行代码评审。

* [ ] 所有仓库的主分支都配置了分支策略 —— 见[工具](../tools.md)页的“配置分支策略”。
* [ ] 项目仓库产出的所有构建都包含合适的 linter，并运行单元测试。
* [ ] 每个缺陷工作项都应当在错误被诊断后，附上引入该缺陷的拉取请求链接。这有助于学习。
* [ ] 每个缺陷工作项都应当注明：这个缺陷在代码评审中\*\*本可以（或本来不会）\*\*被发现的原因。
* [ ] 项目团队定期更新他们的代码评审检查清单，以反映他们遇到过的常见问题。
* [ ] 开发负责人应当抽查一部分拉取请求，和/或其他开发者共同担任评审者，帮助所有人提升代码评审能力。

## 度量

团队可以收集代码评审的度量数据，来衡量其效率。一些有用的指标包括：

* 缺陷清除效率（DRE）—— 衡量开发团队在发布前清除缺陷的能力
* 时间指标：
  * 准备代码检查会议所花的时间
  * 评审会议所花的时间
* 单位时间/每次会议检查的代码行数（LOC）

手工跟踪这些指标（例如用 Excel 表格）是完全合理的做法。也可以利用项目管理平台的功能 —— 例如 AzDO 就支持指标看板，包括[跟踪 bug](https://learn.microsoft.com/en-us/azure/devops/boards/backlogs/manage-bugs?view=azure-devops\&tabs=new-web-form)。你可能会找到各种平台的现成插件 —— 例如看看 [GitHub Marketplace](https://github.com/marketplace) —— 也可以选择自己实现这些功能。

记住：靠评审清除缺陷的成本，远低于到生产环境才发现缺陷的成本，因此做代码评审的成本其实是**负的**！

## 相关资源

* [A Guide to Code Inspections](http://www.ganssle.com/inspections.pdf)

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/code-reviews/evidence-and-measures/README.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

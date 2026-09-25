# 评审流程指南

## 通用指导

无论采用哪种开发模式，代码评审都应当是软件工程团队流程的一部分。此外，团队应当学会及时完成评审。被搁置的[拉取请求](../pull-requests.md)会带来额外的合并问题，并逐渐陈旧、导致工作白做。合格的 PR 应当反映定义明确、简洁的任务，因此内容也很紧凑。评审单个任务所需的时间应当相对较少。

为确保代码评审流程健康、包容，并达成上述目标，可以考虑遵循以下指导：

* 为代码评审建立[服务级别协议（SLA）](https://en.wikipedia.org/wiki/Service-level_agreement)，并写入团队工作约定。
* 尽管现代 DevOps 环境已经具备管理 PR 的工具，给待评审任务打上标签、或在[任务板](../tools.md)上为它们留出专门的位置，仍然很有帮助。
* 在每日站会上检查待评审任务，确保它们都已指派评审者。
* 初级团队和刚接触该流程的团队，可以考虑在创建任务的同时，为评审单独建一个任务。
* 利用[工具](../tools.md)来简化评审流程。
* 培育包容的代码评审 —— 见[评审中的包容性](../inclusion-in-code-review.md)。

## 度量代码评审流程

如果团队发现代码评审需要很长时间才能合并、并已成为阻塞点，可以考虑以下额外建议：

1. 度量每个冲刺周期内合并一个 PR 的平均耗时。
2. 在回顾会上讨论如何改进合并耗时，并为其排定优先级。
3. 跨多个冲刺评估合并耗时，看流程是否在改善。
4. 直接提醒（ping）尚未处理的必需批准人。

## 代码评审不应包含过多行代码

说开发者能评审几百行代码很容易，但当代码超过一定行数后，缺陷发现的效率会下降，做出高质量评审的可能性也会变小。这不是要不要设定代码行数上限的问题，而是要凭常识判断。要评审的代码越多，让 bug 溜过去的可能性就越大。见[拉取请求](../pull-requests.md)的“规模建议”。

## 在合理的情况下尽量自动化

用自动化（lint、代码分析等）来避免产生“[nit](https://en.wikipedia.org/wiki/Nitpicking)”（吹毛求疵的意见），让评审者能更专注于 PR 的功能层面。通过配置自动化构建、测试和检查（这在 [CI 流程](https://github.com/microsoft/code-with-engineering-playbook/blob/main/docs/CI-CD/continuous-integration.md)中可以做到），团队能节省人力评审者的时间，让他们专注于设计和功能等需要恰当评估的方面。这样团队把精力放在真正重要的事情上，成功的机会也更高。

## 按角色划分的指导

* [作者指南](author-guidance.md)
* [评审者指南](reviewer-guidance.md)

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/code-reviews/process-guidance/README.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

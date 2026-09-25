# 代码评审

在项目上工作的开发者，应当对**每一个**拉取请求（pull request，PR）都进行同行代码评审；对共享分支的每一次检入也应如此。

## 目标

代码评审是一种围绕代码展开的对话，参与者可以借此：

* **提升代码质量** —— 在缺陷进入共享代码分支之前发现并清除它们。
* **学习与成长** —— 让别人评审自己的代码，我们会接触到陌生的设计模式或语言，也能改掉一些坏习惯。
* **形成共识** —— 开发者之间就项目代码达成共同理解。

## 本章页面

* [常见问题](../code-reviews/faq.md) —— 代码评审与 PR 的区别、PR 太大怎么办、如何加快评审等
* [拉取请求](../code-reviews/pull-requests.md) —— 通用流程、规模建议、描述规范
* [拉取请求模板](../code-reviews/pull-request-template.md) —— 可直接使用的 PR 模板
* [评审流程指南](../code-reviews/process-guidance/README.md) —— 团队级流程与度量
  * [作者指南](../code-reviews/process-guidance/author-guidance.md)
  * [评审者指南](../code-reviews/process-guidance/reviewer-guidance.md) —— 评审的主体清单
* [评审中的包容性](../code-reviews/inclusion-in-code-review.md) —— 非包容行为的识别与避免
* [证据与度量](../code-reviews/evidence-and-measures/README.md) —— 如何证明评审确实在做、如何度量
* [工具](../code-reviews/tools.md) —— AzDO / GitHub / VS Code / Visual Studio / 网页端
* [语言配方](../code-reviews/recipes/README.md) —— 按语言/技术的专项清单

## 相关资源

* [Google 工程实践文档：如何做代码评审](https://google.github.io/eng-practices/review/reviewer/)
* [Best Kept Secrets of Peer Code Review](https://static1.smartbear.co/smartbear/media/pdfs/best-kept-secrets-of-peer-code-review_redirected.pdf)

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/code-reviews/README.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。本章页清单由译者添加，非原文内容。
{% endhint %}

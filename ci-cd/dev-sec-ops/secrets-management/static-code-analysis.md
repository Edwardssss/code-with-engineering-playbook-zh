# 静态代码分析

静态代码分析是一种通过检查应用源代码来发现安全问题的方法。

## 为什么要做静态代码分析

与代码评审相比，静态代码分析工具更快、更准确、更全面。 由于它直接作用于源代码本身，因此是非常早的问题指示器，而越早发现的编码错误修复成本越低。

## 如何实施静态代码分析

静态代码分析应当集成到你的构建过程中。 可用于静态代码分析的工具[很多](https://owasp.org/www-community/Source_Code_Analysis_Tools)，选择符合你所用编程语言与开发技术的即可。

## 静态代码分析架与工具

[SonarCloud](https://sonarcloud.io) —— 基于云的软件即服务形式的静态代码分析产品。 [OWASP 源代码分析](https://owasp.org/www-community/Source_Code_Analysis_Tools) —— OWASP 对源代码分析工具的推荐。

## 结论

静态代码分析对于找出代码中的潜在问题和安全问题必不可少。它让你能在早期阶段发现 bug 和安全问题。

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/dev-sec-ops/secrets-management/static-code-analysis.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

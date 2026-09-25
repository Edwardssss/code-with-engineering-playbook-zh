# 评估开源软件

鉴于[开源软件供应链攻击](https://devblogs.microsoft.com/engineering-at-microsoft/the-journey-to-secure-the-software-supply-chain-at-microsoft/)威胁上升，开发者应当事先找出可能作为开源依赖的候选，并对照你的需求与所需安全态势对它们做评估。

## 为什么要评估开源软件

开源软件是现代软件开发的关键组成部分。重要的是评估所用的开源软件，确保它满足需求且是安全的。 安全不是开源软件的天然属性；更重要的是，今天安全的软件明天未必安全，所以只对依赖做已知漏洞扫描并不总能覆盖所有情形。 这就是为什么我们需要寻找证据，证明我们所使用的开源软件的维护者有良好的安全态势和对安全的承诺。

## 何时评估开源软件

你应当在项目中使用开源软件**之前**就评估它。如果该软件是你的项目依赖，这一点尤其重要，因为它可能把安全漏洞和其他问题引入你的项目。 代码评审者也应当了解项目中使用的开源软件，并能使用下面提到的工具与资源来评估正在加入项目的开源软件的安全性。

## 如何实施开源软件评估

评估开源软件时，考虑以下几点：

* 能否避免把它作为依赖加入？最好的依赖就是你没有的依赖。
* 它有人维护吗？维护频率如何、工程严格程度如何（即代码评审、分支保护、测试）？
* 有没有证据表明有人在为它的安全投入努力？
* 能否找到证据表明它被大量下游项目使用、或被已知可信的文档引用？它在 GitHub 上有多少 star 和 fork？
* 它是否易于安全地使用？
* 它的许可证是否允许你在项目中使用？
* 是否有如何报告漏洞的说明？
* 它是否存在已知漏洞或安全问题？
* 它的依赖安全吗？或者至少是最新的、有人在积极维护？
* 它是否经过第三方审计，例如 [OpenSSF 安全审查](https://github.com/ossf/security-reviews/blob/main/Overview.md#readme)？

## 评估开源软件的工具

* [OpenSSF Scorecards](https://github.com/ossf/scorecard) —— 这个工具实际上自动化了上面清单中的部分检查，可用于评估开源项目的安全态势。它可以作为 GitHub action 或命令行工具运行，为开源项目给出安全评分卡。注意哪些指标对你、你的组织和客户是重要的。这个工具被[知名的开源项目办公室（OSPO）](https://securityscorecards.dev/#part-of-the-oss-community)用于度量员工对开源的贡献。
* [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/) —— 一个软件成分分析工具，识别项目依赖并检查其中是否存在已知的、已公开披露的漏洞。
* [评估开源软件的简明指南](https://github.com/ossf/wg-best-practices-os-developers/blob/main/docs/Concise-Guide-for-Evaluating-Open-Source-Software.md) —— 一份指南，帮你把本页的知识扩展到对开源软件的评估实践中。

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/dev-sec-ops/evaluate-open-source-software.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

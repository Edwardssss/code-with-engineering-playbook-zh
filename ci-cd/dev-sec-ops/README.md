# DevSecOps

## DevSecOps 的概念

DevSecOps（或 DevOps 安全）指的是在应用开发生命周期中更早地引入安全（也就是所谓“左移”），从而把漏洞的影响降到最小，并让安全更贴近开发团队。

## 为什么

通过拥抱左移思维，DevSecOps 鼓励组织弥合开发团队与安全团队之间常见的隔阂，直到许多安全流程被自动化、并由开发团队有效承担。

## DevSecOps 实践

本节涵盖各种工具、框架与资源，让你能在开发早期就把 DevSecOps 最佳实践引入项目。 涵盖主题：

1. [**凭据扫描**](./secrets-management/credential_scanning.md) —— 自动检查项目，确保项目源代码中不包含任何密钥。
2. [**密钥轮换**](./secrets-management/secrets_rotation.md) —— 一种自动化过程：把应用使用的密钥刷新并替换为新密钥。
3. [**静态代码分析**](./secrets-management/static-code-analysis.md) —— 分析源代码或代码的编译版本，以帮助发现安全缺陷。
4. [**渗透测试**](./penetration-testing.md) —— 对你的应用做模拟攻击，以检查可被利用的漏洞。
5. [**容器依赖扫描**](./dependency-and-container-scanning.md) —— 在容器操作系统、语言包与应用依赖中搜找漏洞。
6. [**评估开源库**](./evaluate-open-source-software.md) —— 通过评估你所使用的库，让开源供应链攻击更难实施。

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/dev-sec-ops/README.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

# 渗透测试

渗透测试是对你的应用做一次模拟攻击，以检查可利用的安全问题。

## 为什么要做渗透测试

渗透测试在运行中的应用上进行。因此它是对应用及其所有层次的端到端测试。它的输出是一次对应用真实成功的模拟攻击 —— 这是一个严重问题，应当尽快处理。

## 如何实施渗透测试

许多组织做人工渗透测试。但每天都有新漏洞被发现，因此自动化渗透测试是一个好做法。 要实现这种自动化，可用渗透测试工具发现漏洞，例如可被代码注入攻击利用的未净化输入。 渗透测试提供的洞察可以用来调优你的 WAF 安全策略，并修补检测到的漏洞。

## 渗透测试架与工具

[OWASP Zed Attack Proxy (ZAP)](https://www.zaproxy.org/) —— 针对 Web 应用的 OWASP 渗透测试工具。

## 结论

渗透测试对于检查应用中的漏洞、保护应用免受模拟攻击必不可少。渗透测试提供的洞察可以识别组织安全态势中的弱点，也能度量其安全策略的合规性、检验员工对安全问题的意识，并判断组织是否、以及如何会遭遇安全灾难。

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/dev-sec-ops/penetration-testing.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

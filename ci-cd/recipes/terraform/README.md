# Terraform 配方

本子节汇总 Terraform 在 CI/CD 中的具体做法，译自原手册的 `docs/CI-CD/recipes/terraform/`。

* **把 Terraform 输出保存到变量组** —— 用 `terraform output` + `az pipelines variable-group` 把输出值传给其他流水线
* **在 Terraform 模块间共享通用变量与命名约定** —— 用 common 模块集中管理命名与共享变量
* **Terraform 配置的结构与测试指南** —— 模块目录结构、测试分层与命名规范

{% hint style="info" %}
本索引页由译者添加（原手册中 `recipes/terraform/` 目录没有索引页）；下方各页均为原文档的直接翻译。译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/recipes/terraform/`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

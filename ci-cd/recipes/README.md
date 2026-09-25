# 配方

本节汇总 CI/CD 场景下的具体做法，译自原手册的 `docs/CI-CD/recipes/`。

* **低代码方案的持续交付** —— Power Platform 环境与三阶段部署
* **为更好的文档做 CI 流水线** —— 用 markdownlint / lychee / write-good 自动检查文档
* **用 Jupyter Notebook 做 CI** —— 提交时自动把 notebook 转为脚本，以便 PR 评论
* **GitHub Actions 中的运行时变量** —— 用提交信息或 PR 描述注入变量
* **包容性 lint** —— 用 `woke` 检查非包容性用语
* **在流水线中复用 Dev Container** —— 三种构建方式与利弊对比
* **Terraform** —— 输出传递、变量共享与结构规范

{% hint style="info" %}
本索引页由译者添加（原手册中 `recipes/` 目录没有索引页）；下方各页均为原文档的直接翻译。译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/recipes/`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

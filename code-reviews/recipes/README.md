# 语言配方

本节的配方译自原手册的 `docs/code-reviews/recipes/`，针对具体语言或技术，给出该场景下的评审检查清单、lint 与格式化工具配置，以及构建校验方式。

使用方式：先读通用的[评审者指南](../process-guidance/reviewer-guidance.md)，再叠加你所评审代码对应的配方清单。

* [Azure Pipelines YAML](azure-pipelines-yaml.md) —— 流水线结构与 YAML 结构、权限与密钥
* [Bash](bash.md) —— shellcheck、shfmt、依赖与路径处理
* [C#](csharp.md) —— 分析器、异步、依赖注入、资源释放
* [Go](go.md) —— go vet、golangci-lint、错误处理与并发
* [Java](java.md) —— Checkstyle、PMD、FindBugs
* [JavaScript / TypeScript](javascript-and-typescript.md) —— ESLint、Prettier、测试与异步
* [Markdown](markdown.md) —— 文档类改动的语法、链接与写作规范
* [Python](python.md) —— PEP8、flake8、black、类型标注
* [Terraform](terraform.md) —— provider 版本、状态管理、变量与命名

{% hint style="info" %}
本索引页由译者添加（原手册中 `recipes/` 目录没有索引页）；下方各页均为原文档的直接翻译。译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/code-reviews/recipes/`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

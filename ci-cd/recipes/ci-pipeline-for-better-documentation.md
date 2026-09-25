# 为更好的文档做 CI 流水线

## 引言

大多数项目开头都有探索阶段（spike），开发者和分析师会产出大量文档。

有时这些文档没有统一标准，每位团队成员按自己的偏好来写。再加上评审者要花时间确认语法、找错别字或非包容性用语。

这条流水线就是为了解决这个问题！

## 这条流水线

流水线使用以下 `npm` 模块：

* [markdownlint](https://github.com/DavidAnson/markdownlint)：用[规则](https://github.com/DavidAnson/markdownlint#rules--aliases)做标准化
* [lychee](https://github.com/lycheeverse/lychee)：检查文档中的链接并报告失效链接
* [write-good](https://github.com/btford/write-good)：英文行文 linter

我们在多个项目中已经使用这条流水线一年多了，客户的反馈一直很好！

## 怎么用

要开始使用这条流水线：

1. 从[这个仓库](https://github.com/squassina/doc-pipeline/tree/main/repo-root)下载文件
2. 如果仓库是空的，把目录和文件解压到仓库根目录
   * 如果不是全新的，就把文件复制过去并做必要调整：
     * 检查 `.azdo`，使其符合你的仓库规范
     * 检查 `package.json`，避免覆盖你已有的文件。如果你改了 `.azdo` 目录名，也要同步更新该文件。
3. 在 Azure DevOps 或 GitHub 中创建流水线

## 相关资源

[工程基础手册中的 Markdown 代码评审](https://microsoft.github.io/code-with-engineering-playbook/code-reviews/recipes/markdown/#code-review-checklist)

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/recipes/ci-pipeline-for-better-documentation.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

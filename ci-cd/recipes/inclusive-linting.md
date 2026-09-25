# 包容性 lint

作为软件从业者，我们应当致力于营造包容的工作环境，这自然会延伸到我们编写的代码与文档上。在整个项目或仓库中保持一致地使用包容性语言很重要。

为此，我们推荐使用文本文件分析工具（例如包容性 linter），并把它作为 CI 流水线的一个步骤。

## 要 lint 什么

包容性 linter 的主要目标，是标出源代码中任何非包容性用语的出现位置（并可选地给出替代建议）。项目中的非包容性词句可能出现在任何地方，从注释、文档到变量名。

包容性 linter 可能自带一份“默认”的非包容性词句字典供运行，作为良好的起点。这类工具通常也可定制，往往提供忽略某些词条和/或添加自定义词条的能力。

能向 linter 添加额外词条还有一个附带好处：在包容性 lint 之外，还能实现敏感语言的 lint。例如，这可以防止客户名称或其他非公开信息流入你的 git 历史。

## 开始使用包容性 linter

### woke

我们推荐的一个包容性 linter 是 `woke`。它是一个与语言无关的 CLI 工具，能检测源代码中的非包容性语言并推荐替代说法。`woke` 会自动应用一套含非包容性词条的默认规则集来 lint，你也可以通过 yaml 文件应用自定义规则配置，添加额外的词条。

在本地对文件或目录运行该工具相当简单：

```sh
$ woke test.txt

test.txt:2:2-6: `guys` may be insensitive, use `folks`, `people` instead (warning)
* guys
  ^
```

`woke` 可以在你的本机或 CI/CD 系统上通过 CLI 运行，也提供两个 GitHub Actions：

* Run woke
* Run woke with Reviewdog

要在 CI 流水线中使用带默认规则集的标准 “Run woke” GitHub Action：

1. 把 `woke` action 作为一步加入你项目的 CI 流水线 yaml：

   ```yaml
   name: ci
   on:
     - pull_request
   jobs:
     woke:
       name: woke
       runs-on: ubuntu-latest
       steps:
         - name: Checkout
           uses: actions/checkout@v2

         - name: woke
           uses: get-woke/woke-action@v0
           with:
             # Cause the check to fail on any broke rules
             fail-on-error: true
   ```
2. 运行你的流水线
3. 在仓库主页的 “Actions” 标签中查看输出

## 相关资源

* [woke](https://github.com/get-woke/woke)
* [默认规则集](https://github.com/get-woke/woke/blob/main/pkg/rule/default.yaml)
* [example.yaml](https://github.com/get-woke/woke/blob/main/example.yaml)
* [Run woke](https://github.com/marketplace/actions/run-woke)
* [Run woke with reviewdog](https://github.com/marketplace/actions/run-woke-with-reviewdog)
* [文档](https://docs.getwoke.tech/)

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/recipes/inclusive-linting.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

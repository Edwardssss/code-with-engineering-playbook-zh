# Markdown

## 风格指南

开发者应当把文档当作其他源代码一样对待，在评审“文档即代码”时遵循同样的规则与检查清单。

文档既要使用良好的 Markdown 语法以确保能被正确解析，也要遵循良好的[写作风格指南](#写作风格指南)，确保文档易读易懂。

## Markdown

Markdown 是一种轻量级标记语言，用于为纯文本文档添加格式化元素。它由 John Gruber 在 2004 年创造，如今是世界上最流行的标记语言之一。

使用 Markdown 与使用所见即所得（WYSIWYG）编辑器不同。在 Microsoft Word 这类应用中，你点击按钮为词句设置格式，改动立即可见。Markdown 不是这样。创建 Markdown 格式的文件时，你在文本中加入 Markdown 语法，以指明哪些词句应当以不同方式呈现。

更多信息与完整文档见[这里](https://www.markdownguide.org/)。

## Linter

Markdown 有特定的格式要求。尊重这些格式很重要，否则一些严格的解释器将无法正确显示文档。人们常借助 linter 帮助开发者正确创建文档 —— 它既能校验 Markdown 语法，也能检查语法、拼写与英语表达。

一套好的配置包括：编辑期间使用 Markdown linter，并在 PR 构建验证中运行；编辑文档时使用语法检查 linter。以下是可用于这套配置的 linter 列表。

### markdownlint

[`markdownlint`](https://github.com/markdownlint/markdownlint) 是一个 Markdown linter，它校验 Markdown 语法，并强制一批让文本更易读的规则。[Markdownlint-cli](https://github.com/igorshubovych/markdownlint-cli) 是基于 Markdownlint 的易用 CLI。

它以 [Ruby gem](https://github.com/markdownlint/markdownlint)、[npm 包](https://github.com/DavidAnson/markdownlint)、[Node.js CLI](https://github.com/igorshubovych/markdownlint-cli) 和 [VS Code 扩展](https://github.com/DavidAnson/vscode-markdownlint)的形式提供。VS Code 扩展 [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) 也能捕获所有 markdownlint 错误。

安装 Node.js CLI：

```bash
npm install -g markdownlint-cli
```

在 Node.js 项目上运行 markdownlint：

```bash
markdownlint **/*.md --ignore node_modules
```

自动修复错误：

```bash
markdownlint **/*.md --ignore node_modules --fix
```

markdownlint 规则的完整列表见[这里](https://github.com/DavidAnson/markdownlint/blob/main/doc/Rules.md)。

### write-good

[`write-good`](https://github.com/btford/write-good) 是一个英语文本 linter，帮助写出更好的文档。

```bash
npm install -g write-good
```

运行 write-good：

```bash
write-good *.md
```

不安装直接运行 write-good：

```bash
npx write-good *.md
```

Write Good 也提供 [VS Code 扩展](https://marketplace.visualstudio.com/items?itemName=travisthetechie.write-good-linter)。

## VS Code 扩展

### Write Good Linter

[`Write Good Linter 扩展`](https://marketplace.visualstudio.com/items?itemName=travisthetechie.write-good-linter) 与 VS Code 集成，在编辑文档时给出语法与语言建议。

### markdownlint 扩展

[`markdownlint 扩展`](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint) 检查 Markdown 文档，在编辑时对违规规则给出警告。

## 构建校验

### Lint

要在 GitHub Actions 中用 `markdownlint` 自动化 lint 以做 PR 验证，你既可以使用 linter 聚合器（就像[本仓库中使用 MegaLinter 那样](https://github.com/microsoft/code-with-engineering-playbook/blob/main/.github/workflows/mega-linter.yml)），也可以使用下面的 YAML。

```yaml
name: Markdownlint

on:
  push:
    paths:
      - "**/*.md"
  pull_request:
    paths:
      - "**/*.md"

jobs:
  lint:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Use Node.js
      uses: actions/setup-node@v1
      with:
        node-version: 12.x
    - name: Run Markdownlint
      run: |
        npm i -g markdownlint-cli
        markdownlint "**/*.md" --ignore node_modules
```

### 检查链接

要自动化 Markdown 文件中的链接检查，请把 `lycheeverse/lychee-action` action 加入你的验证流水线：

```yaml
  markdown-link-check:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Link Checker
      id: lychee
      uses: lycheeverse/lychee-action@v2
```

关于该 action 选项的更多信息见 [`lychee-action` 主页](https://github.com/lycheeverse/lychee-action)。

## 代码评审检查清单

除了通用的代码评审检查清单，你还应当检查以下文档特有的条目：

* [ ] 文档是否易读易懂，是否遵循[良好的写作指南](#写作风格指南)？
* [ ] 是否存在唯一事实来源，还是内容在多个文档中重复？
* [ ] 文档是否与代码保持同步？
* [ ] 文档在技术上和伦理上是否正确？

## 写作风格指南

以下是一些写作风格指南的示例。

在团队内就哪些指南适用于你的项目文档达成一致。 把指南与文档放在一起，方便回头查阅。

### 措辞

* 使用包容性语言，避免行话和生僻词。文档应当易于理解
* 清晰、简洁，紧扣文档目标
* 使用主动语态
* 对文本做拼写与语法检查
* 始终按时间顺序叙述
* 访问 [Plain English](https://plainenglish.co.uk/how-to-write-in-plain-english.html) 获取如何写出易懂文档的建议。

### 文档组织

* 按主题而不是按类型组织文档，这样更容易找到文档
* 每个文件夹都应当有顶层 README.md，该文件夹内的其他文档都应当直接或间接地从该 README.md 链接过去
* 超过一个词的文档名应当用下划线而不是空格，例如 `machine_learning_pipeline_design.md`。图片同理

### 标题

* 以 H1 开头（markdown 中单个 `#`），并遵守 H1 > H2 > H3 的顺序
* 每个标题之后先写正文，再进入下一个标题
* 避免在标题中使用数字。数字会变动，可能造成标题过时
* 避免在标题中使用符号和特殊字符，这会给锚点链接带来问题
* 避免在标题中放链接

### 资源引用

* 避免内容重复，改为链接到唯一事实来源
* 链接但不要概括。在另一个页面概括内容，会让内容同时存在于两处
* 使用有意义的链接文本。例如不要写“按这里的说明操作”，而应写“遵循 Markdown 指南”
* 确保指向 Microsoft 文档的链接不包含语言标记 `/en-us/` 或 `/fr-fr/`，因为这由站点自身自动决定

### 列表

* 列表项尽量以大写字母开头
* 当项目描述的是需要遵循的顺序时使用有序列表，否则使用无序列表
* 对于有序列表，每项都以 `1.` 开头。渲染时会按顺序编号，这样能避免列表中的编号断档
* 不要在列表项末尾加逗号 `,` 或分号 `;`；除非列表项是一个完整句子，否则也避免句号 `.`

### 图片

* 把图片放在名为 `img` 的单独目录中
* 给图片起合适的名字，避免 `screenshot.png` 这类通用名
* 避免把大图片或视频加入版本控制，改为链接到外部位置

### 强调与特殊区块

* 用**粗体**或*斜体*来强调

  > 对于本文档的所有读者都需要知悉的内容，使用引用块
* 用 `反引号` 标示代码：行内代码用单个反引号（如 `pip install flake8`），代码块用 3 个反引号并在其后标注语言以启用语法高亮

  ```python
  def add(num1: int, num2: int):
    return num1 + num2
  ```
* 任务清单使用复选框
  * [ ] 事项 1
  * [ ] 事项 2
  * [x] 事项 3
* 在本页末尾添加“参考资料”一节，附上外部引用链接
* 做对比和汇报时优先用表格而不是列表，让调研与结果更易读

  | 方案   | 优点   | 缺点   |
  | ---- | ---- | ---- |
  | 方案 1 | 一些优点 | 一些缺点 |
  | 方案 2 | 一些优点 | 一些缺点 |

### 通用

* 始终使用 Markdown 语法，不要与 HTML 混用
* 确保文件扩展名是 `.md` —— 扩展名缺失时，linter 可能会忽略这些文件

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/code-reviews/recipes/markdown.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

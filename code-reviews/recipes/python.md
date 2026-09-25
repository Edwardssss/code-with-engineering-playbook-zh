# Python 代码评审

## 风格指南

开发者应当遵循 [PEP8 风格指南](https://pep8.org/)，并使用[类型标注（type hints）](https://www.python.org/dev/peps/pep-0484/)。全程使用类型标注，配合 lint 与类型标注检查，可以避免一些难以调试的常见错误。

项目应当用自动化工具检查 Python 代码。

Lint 应当加入构建校验，而 lint 与代码格式化都可以加入提交前钩子和 VS Code。

## 代码分析 / Lint

最流行的两个 Python linter 是 [Pylint](https://pypi.org/project/pylint/) 和 [Flake8](https://pypi.org/project/flake8/)。两者都检查对 `PEP8` 的遵循程度，但在其他检查规则上略有差异。总体而言 `Pylint` 更严格一些、误报也更多，但两者都是 lint Python 代码的好选择。

`Pylint` 和 `Flake8` 都可以通过 VS Code 的 `python 扩展`在 VS Code 中配置。

### Flake8

Flake8 是对 [`Pyflakes`](https://github.com/PyCQA/pyflakes)（检测编码错误）和 [`pycodestyle`](https://github.com/PyCQA/pycodestyle)（检查 pep8）的简单快速封装。

安装 `Flake8`：

```bash
pip install flake8
```

为 flake8 添加 [`pydocstyle`](https://github.com/PyCQA/pydocstyle)（检查[文档字符串](https://www.python.org/dev/peps/pep-0257/)）工具的扩展。

```bash
pip install flake8-docstrings
```

为 flake8 添加 [`pep8-naming`](https://github.com/PyCQA/pep8-naming)（检查 pep8 中的[命名约定](https://www.python.org/dev/peps/pep-0008/#naming-conventions)）工具的扩展。

```bash
pip install pep8-naming
```

运行 `Flake8`：

```bash
flake8 .    # lint the whole project
```

### Pylint

安装 `Pylint`：

```bash
pip install pylint
```

运行 `Pylint`：

```bash
pylint src  # lint the source directory
```

## 自动格式化

### Black

[`Black`](https://github.com/psf/black) 是一个“不道歉”的代码格式化工具。它免除了 `pycodestyle` 对格式的吹毛求疵，让团队可以专注于内容而不是风格。你无法为 black 配置自己的风格。

```bash
pip install black
```

格式化 Python 代码：

```bash
black [file/folder]
```

### autopep8

[`Autopep8`](https://github.com/hhatto/autopep8) 更宽松，如果你不想要那么严格的格式，它允许更多配置。

```bash
pip install autopep8
```

格式化 Python 代码：

```bash
autopep8 [file/folder] --in-place
```

### yapf

[yapf](https://github.com/google/yapf)——Yet Another Python Formatter，是 Google 基于 gofmt 思路做的一个 Python 格式化工具。它同样更可配置，是自动代码格式化的好选择。

```bash
pip install yapf
```

格式化 Python 代码：

```bash
yapf [file/folder] --in-place
```

### Bandit

[Bandit](https://github.com/PyCQA/bandit) 是 Python Code Quality Authority（PyCQA）设计的工具，用于对 Python 代码做静态分析，专门针对安全问题。 它扫描 Python 代码库中的常见安全问题。

* **安装**：用以下命令把 Bandit 加入开发环境：

  ```bash
  pip install bandit
  ```

## VS Code 扩展

### Python

[`Python 语言扩展`](https://marketplace.visualstudio.com/items?itemName=ms-python.python) 是用 VS Code 做 Python 开发应当安装的基础扩展。它提供智能提示、调试、lint（使用上述 linter）、基于 pytest 或 unittest 的测试，以及用上述格式化工具做代码格式化。

### Pyright

[`Pyright 扩展`](https://marketplace.visualstudio.com/items?itemName=ms-pyright.pyright) 在你使用类型标注时，为 VS Code 增加静态类型检查能力。

```python
def add(first_value: int, second_value: int) -> int:
    return first_value + second_value
```

## 构建校验

要在 Azure DevOps 中用 `flake8` 自动化 lint、用 `pytest` 做测试，可以把下面的片段加入你的 `azure-pipelines.yaml`。

```yaml
trigger:
  branches:
    include:
    - develop
    - master
  paths:
    include:
    - src/*

pool:
  vmImage: 'ubuntu-latest'

jobs:
- job: LintAndTest
  displayName: Lint and Test

  steps:

  - checkout: self
    lfs: true

  - task: UsePythonVersion@0
    displayName: 'Set Python version to 3.6'
    inputs:
      versionSpec: '3.6'

  - script: pip3 install --user -r requirements.txt
    displayName: 'Install dependencies'

  - script: |
      # Install Flake8
      pip3 install --user flake8
      # Install PyTest
      pip3 install --user pytest
    displayName: 'Install Flake8 and PyTest'

  - script: |
      python3 -m flake8
    displayName: 'Run Flake8 linter'

  - script: |
      # Run PyTest tester
      python3 -m pytest --junitxml=./test-results.xml
    displayName: 'Run PyTest Tester'

  - task: PublishTestResults@2
    displayName: 'Publish PyTest results'
    condition: succeededOrFailed()
    inputs:
      testResultsFiles: '**/test-*.xml'
      testRunTitle: 'Publish test results for Python $(python.version)'
```

要在 GitHub 上做 PR 验证，你可以用 [GitHub Actions](https://help.github.com/en/actions/language-and-framework-guides/using-python-with-github-actions) 搭配类似的 YAML 配置。

## 提交前钩子

提交前钩子让你在提交拉取请求之前，就在本地完成代码格式化与 lint。

用 pre-commit 包为你的 Python 仓库添加提交前钩子很简单。

1. 安装 pre-commit 并加入 requirements.txt

   ```sh
   pip install pre-commit
   ```
2. 在仓库根目录添加 `.pre-commit-config.yaml` 文件，写入想要的提交前动作

   ```yaml
   repos:
   -   repo: https://github.com/ambv/black
       rev: stable
       hooks:
       - id: black
       language_version: python3.6
   -   repo: https://github.com/pre-commit/pre-commit-hooks
       rev: v1.2.3
       hooks:
       - id: flake8
   ```
3. 每位想配置提交前钩子的开发者随后运行

   ```sh
   pre-commit install
   ```

下一次尝试提交时，任何 lint 失败都会阻止提交。

> 注意：安装提交前钩子是自愿的，由每位开发者自行完成。因此它不能替代服务端的构建校验。

## 代码评审检查清单

除了通用的[代码评审检查清单](../process-guidance/reviewer-guidance.md)，你还应当检查以下 Python 特有的条目：

* [ ] 所有新使用的包是否都已写入 requirements.txt
* [ ] 代码是否通过全部 lint 检查？
* [ ] 函数是否使用类型标注，是否存在类型标注错误？
* [ ] 代码是否易读，是否尽可能使用 Pythonic 写法？

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/code-reviews/recipes/python.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

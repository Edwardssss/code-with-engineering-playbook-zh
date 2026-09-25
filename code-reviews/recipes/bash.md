# Bash

## 风格指南

开发者应当遵循 [Google 的 Bash 风格指南](https://google.github.io/styleguide/shell.xml)。

## 代码分析 / Lint

项目必须在 [CI 流程](https://github.com/microsoft/code-with-engineering-playbook/blob/main/docs/CI-CD/continuous-integration.md)中，用 [shellcheck](https://github.com/koalaman/shellcheck) 检查 Bash 代码。 除了 lint，还可以用 [shfmt](https://github.com/mvdan/sh) 自动格式化 shell 脚本。有几个基于 shfmt 的 VS Code 扩展，例如 shell-format，可以用来自动格式化 shell 脚本。

## 项目配置

### vscode-shellcheck

VS Code 中应当使用 Shellcheck 扩展，它提供静态代码分析能力，并能自动修复 lint 问题。在 VS Code 中使用 vscode-shellcheck 的步骤如下：

#### 在你的机器上安装 shellcheck

macOS：

```bash
brew install shellcheck
```

Ubuntu：

```bash
apt-get install shellcheck
```

#### 在 VS Code 中安装 shellcheck

在 VS Code 中找到 vscode-shellcheck 扩展并安装。

## 自动格式化

### shell-format

shell-format 扩展可以自动格式化你的 Bash 脚本、Docker 文件以及若干配置文件。它依赖 shfmt，而 shfmt 可以按 Google 风格指南检查 Bash。 在 VS Code 中使用 shell-format 的步骤如下：

#### 在你的机器上安装 shfmt

需要 Go 1.13 或更高版本。

```bash
GO111MODULE=on go get mvdan.cc/sh/v3/cmd/shfmt
```

#### 在 VS Code 中安装 shell-format

在 VS Code 中找到 shell-format 扩展并安装。

## 构建校验

要在 Azure DevOps 中自动化这一过程，可以把下面的片段加入你的 `azure-pipelines.yaml`。它会 lint `./scripts/` 文件夹下的所有脚本。

```yaml
- bash: |
    echo "This checks for formatting and common bash errors. See wiki for error details and ignore options: https://github.com/koalaman/shellcheck/wiki/SC1000"
    export scversion="stable"
    wget -qO- "https://github.com/koalaman/shellcheck/releases/download/${scversion?}/shellcheck-${scversion?}.linux.x86_64.tar.xz" | tar -xJv
    sudo mv "shellcheck-${scversion}/shellcheck" /usr/bin/
    rm -r "shellcheck-${scversion}"
    shellcheck ./scripts/*.sh
  displayName: "Validate Scripts: Shellcheck"
```

同样，你也可以用 `shfmt` 在构建流水线中格式化 shell 脚本：

```yaml
- bash: |
    echo "This step does auto formatting of shell scripts"
    shfmt -l -w ./scripts/*.sh
  displayName: "Format Scripts: shfmt"
```

基于 [shunit2](https://github.com/kward/shunit2) 的单元测试也可以加进构建流水线：

```yaml
- bash: |
    echo "This step unit tests shell scripts by using shunit2"
    ./shunit2
  displayName: "Format Scripts: shfmt"
```

## 提交前钩子（pre-commit）

所有开发者都应当把 shellcheck 和 shfmt 作为提交前钩子来运行。

### 第 1 步 —— 安装 pre-commit

运行 `pip install pre-commit` 安装 pre-commit。 如果你用 homebrew，也可以运行 `brew install pre-commit`。

### 第 2 步 —— 添加 shellcheck 和 shfmt

在 Go 项目根目录添加 .pre-commit-config.yaml 文件。像下面这样把 shfmt 加入 .pre-commit-config.yaml 即可在提交前运行它。

```yaml
-   repo: git://github.com/pecigonzalo/pre-commit-fmt
    sha: master
    hooks:
      -   id: shell-fmt
          args:
            - --indent=4
```

```yaml
-   repo: https://github.com/shellcheck-py/shellcheck-py
    rev: v0.7.1.1
    hooks:
    -   id: shellcheck
```

### 第 3 步

运行 `$ pre-commit install` 来设置 git 钩子脚本。

## 依赖

Bash 脚本常被用来把其他系统和工具“粘”在一起。因此 Bash 脚本往往会有很多、和/或很复杂的依赖。可以考虑使用 Docker 容器，确保脚本在一个可移植、可复现、且保证包含全部正确依赖的环境中执行。为了让容器化的脚本仍然易于执行，可以考虑对调用者隐藏 Docker 的存在：用一个“bootstrap”包裹脚本，检查脚本是否运行在 Docker 中，如果不在则重新在 Docker 中执行自身。这样两头的好处都有：脚本易于执行，环境又保持一致。

```bash
if [[ "${DOCKER}" != "true" ]]; then
  docker build -t my_script -f my_script.Dockerfile . > /dev/null
  docker run -e DOCKER=true my_script "$@"
  exit $?
fi

# ... my_script 的实现写在这里，它可以假定所有依赖都存在，因为它总是在 Docker 中运行 ...
```

## 代码评审检查清单

除了通用的代码评审检查清单，你还应当检查以下 Bash 特有的条目：

* [ ] 代码是否使用了 shell 的[内置选项](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html)（如 set -o、set -e、set -u）来控制 shell 脚本的执行？
* [ ] 代码是否模块化？shell 脚本可以像 Python 模块那样模块化。在复杂的 Bash 项目中，Bash 脚本的各个部分应当通过 source 引入。
* [ ] 所有异常都被正确处理了吗？异常应当通过退出码或捕获信号来正确处理。
* [ ] 代码是否通过了 shellcheck 的全部 lint 检查、以及 shunit2 的单元测试？
* [ ] 代码使用的是相对路径还是绝对路径？应当避免相对路径，因为它们容易遭受环境攻击。如果确实需要相对路径，请检查 `PATH` 变量是否已设置。
* [ ] 代码是否把凭据作为用户输入？脚本中的凭据是否被掩码或加密？

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/code-reviews/recipes/bash.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

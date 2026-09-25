# Go

## 风格指南

开发者应当遵循 [Effective Go](https://golang.org/doc/effective_go.html) 风格指南。

## 代码分析 / Lint

### 项目配置

下面是你希望在 VS Code 中拥有的项目配置。

#### VS Code Go 扩展

使用 Visual Studio Code 的 Go 扩展，你可以获得 IntelliSense、代码导航、符号搜索、括号匹配、代码片段等语言特性。这个扩展为 VS Code 中的 Go 提供了丰富的语言支持。

#### go vet

`go vet` 是一个静态分析工具，用于检查 Go 的常见错误，例如 range 循环变量使用不当、printf 参数不匹配等。Go 代码应当能在没有任何 `go vet` 错误的情况下构建。它已包含在 vscode-go 扩展中。

#### golint

> **注意：** golint 库已废弃并归档。

下面的 revive linter 可能是合适的替代方案。

[golint](https://github.com/golang/lint) 在发现大量问题上是个有效工具，但它倾向于产生误报。最好由开发者在写代码时使用，而不是作为自动化构建流程的一部分。它是 vscode-go 扩展默认配置的 linter。

#### revive

[Revive](https://revive.run/) 是一个 Go 的 linter，它提供开发自定义规则的框架，并允许你定义严格的预设，以增强你的开发与代码评审流程。

## 自动格式化

### gofmt

`gofmt` 是 Go 的自动代码格式化风格指南。它是 VS Code 扩展的一部分，默认在保存每个文件时运行。

## 聚合器

### golangci-lint

[golangci-lint](https://github.com/golangci/golangci-lint/) 是现已废弃的 `gometalinter` 的替代品。它比 `gometalinter` 快 2-7 倍，[还有其他诸多好处](https://github.com/golangci/golangci-lint/#comparison)。

golangci-lint 是一个强大、可定制的 linter 聚合器。默认启用了若干 linter，但不是全部。完整的 linter 列表及其用途见[这里](https://github.com/golangci/awesome-go-linters)。

它允许你配置每个 linter，并选择要在项目中启用哪些。

`golangci-lint` 的一个很棒的特性是，它可以很轻松地引入到已有的大型代码库中，使用 `--new-from-rev COMMITID`。开启这个设置后，只有新引入的问题会被标记出来，团队因此可以在不修复大型代码库全部历史问题的情况下改进新代码。这为改进既有解决方案的代码评审提供了一条很好的路径。golangci-lint 也可以设为 VS Code 的默认 linter。

golangci-lint 的安装方式见 [golangci-lint](https://github.com/golangci/golangci-lint#binary)。

在 VS Code 中使用 golangci-lint，推荐如下设置：

```json
"go.lintTool":"golangci-lint",
   "go.lintFlags": [
     "--fast"
   ]
```

## 提交前钩子

所有开发者都应当在提交前钩子中运行 `gofmt`，以确保格式统一。

### 第 1 步 —— 安装 pre-commit

运行 `pip install pre-commit` 安装 pre-commit。 如果你用 homebrew，也可以运行 `brew install pre-commit`。

### 第 2 步 —— 在 pre-commit 中添加 go-fmt

在 Go 项目根目录添加 .pre-commit-config.yaml 文件。像下面这样添加即可在提交前运行 go-fmt。

```yaml
- repo: git://github.com/dnephin/pre-commit-golang
  rev: master
  hooks:
    - id: go-fmt
```

### 第 3 步

运行 `$ pre-commit install` 来设置 git 钩子脚本。

## 构建校验

每次构建都应当运行 `gofmt`，以强制统一标准。

要在 Azure DevOps 中自动化这一过程，可以把下面的片段加入你的 `azure-pipelines.yaml`。它会格式化 `./scripts/` 文件夹下的所有脚本。

```yaml
- script: go fmt
  workingDirectory: $(System.DefaultWorkingDirectory)/scripts
  displayName: "Run code formatting"
```

每次构建也应当运行 `govet` 来检查代码 lint。

要在 Azure DevOps 中自动化这一过程，可以把下面的片段加入你的 `azure-pipelines.yaml` 文件。它会检查 `./scripts/` 文件夹下所有脚本的 lint。

```yaml
- script: go vet
  workingDirectory: $(System.DefaultWorkingDirectory)/scripts
  displayName: "Run code linting"
```

或者你可以把 golangci-lint 作为流水线的一个步骤，一次完成多项已启用的校验（包括 go vet 和 go fmt）。

```yaml
- script: golangci-lint run --enable gofmt --fix
  workingDirectory: $(System.DefaultWorkingDirectory)/scripts
  displayName: "Run code linting"
```

## Azure DevOps 中的示例构建校验流水线

```yaml
trigger: master

pool:
   vmImage: 'ubuntu-latest'

steps:

- task: GoTool@0
  inputs:
    version: '1.13.5'

- task: Go@0
  inputs:
    command: 'get'
    arguments: '-d'
    workingDirectory: '$(System.DefaultWorkingDirectory)/scripts'


- script: go fmt
  workingDirectory: $(System.DefaultWorkingDirectory)/scripts
  displayName: "Run code formatting"

- script: go vet
  workingDirectory: $(System.DefaultWorkingDirectory)/scripts
  displayName: 'Run go vet'

- task: Go@0
  inputs:
    command: 'build'
    workingDirectory: '$(System.DefaultWorkingDirectory)'

- task: CopyFiles@2
  inputs:
    TargetFolder: '$(Build.ArtifactStagingDirectory)'
- task: PublishBuildArtifacts@1
  inputs:
     artifactName: drop
```

## 代码评审检查清单

Go 语言团队维护了一份常见的[代码评审意见](https://github.com/golang/go/wiki/CodeReviewComments)列表，对于一个使用 Go 的团队来说，它是可靠的检查清单基础，应当与通用的代码评审检查清单一起使用。

* [ ] 代码是否[正确处理错误](https://golang.org/doc/effective_go.html#errors)？这包括不通过 `_` 赋值丢弃错误，以及返回错误、而不是[带内错误值](https://github.com/golang/go/wiki/CodeReviewComments#in-band-errors)。
* [ ] 代码是否遵循 Go 关于方法[接收者类型](https://github.com/golang/go/wiki/CodeReviewComments#receiver-type)的标准？
* [ ] 该传值的时候，代码是否[传值](https://github.com/golang/go/wiki/CodeReviewComments#pass-values)？
* [ ] 代码中的接口是否定义在[正确的包](https://github.com/golang/go/wiki/CodeReviewComments#interfaces)中？
* [ ] 代码中的 goroutine 是否有[清晰的生命周期](https://github.com/golang/go/wiki/CodeReviewComments#goroutine-lifetimes)？
* [ ] 代码中的并行是否通过 goroutine 和 channel 配合[同步方法](https://github.com/golang/go/wiki/CodeReviewComments#synchronous-functions)来处理？
* [ ] 代码是否有有意义的[文档注释](https://github.com/golang/go/wiki/CodeReviewComments#doc-comments)？
* [ ] 代码是否有有意义的[包注释](https://github.com/golang/go/wiki/CodeReviewComments#package-comments)？
* [ ] 代码是否正确使用了 [Context](https://github.com/golang/go/wiki/CodeReviewComments#contexts)？
* [ ] 单元测试失败时是否给出[有意义的错误信息](https://github.com/golang/go/wiki/CodeReviewComments#useful-test-failures)？

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/code-reviews/recipes/go.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

# Azure Pipelines YAML

## 风格指南

开发者应当遵循 [YAML schema 参考](https://learn.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops\&tabs=schema%2Cparameter-schema)。

## 代码分析 / Lint

最流行的 YAML linter 是 [YAML](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml) 扩展。这个扩展提供 YAML 校验、文档大纲、自动补全、悬停提示和格式化功能。

## VS Code 扩展

有一个 [Azure Pipelines for VS Code](https://marketplace.visualstudio.com/items?itemName=ms-azure-devops.azure-pipelines) 扩展，可以为 VS Code 中的 Azure Pipelines YAML 提供语法高亮和自动补全。它还能让你不离开 VS Code 就为 Azure WebApps 配置持续构建与部署。

## Azure Pipelines 中的 YAML 概览

流水线被触发后、在运行之前，会经历几个阶段，例如[排队时、编译时和运行时](https://adamtheautomator.com/azure-devops-variables/#Pipeline_Execution_Phases)，在这些阶段中变量会按照其[运行时表达式语法](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops\&tabs=yaml%2Cbatch#runtime-expression-syntax)被解释。

流水线被触发时，所有嵌套的 YAML 文件都会被展开以在 Azure Pipelines 中运行。本检查清单包含评审所有嵌套 YAML 文件的一些技巧。

评审 YAML 文件时，以下文档可能有用：

* [Azure Pipelines YAML 文档](https://learn.microsoft.com/en-us/azure/devops/pipelines/yaml-schema)
* [流水线运行顺序](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/runs?view=azure-devops)
* [Azure Pipelines 新用户核心概念](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/key-pipelines-concepts?view=azure-devops)

**核心概念概览** ![Azure Pipelines 核心概念](https://2540885121-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FdjAcIWTODFml0GX4YWCQ%2Fuploads%2F1eOeE9cTXnWdIOEfumt6%2Fkey-concepts-overview.png?alt=media)

* 触发器（trigger）告诉流水线何时运行。
* 流水线由一个或多个阶段（stage）组成。一条流水线可以部署到一个或多个环境。
* 阶段是在流水线中组织作业（job）的方式，每个阶段可以有一个或多个作业。
* 每个作业在一个代理（agent）上运行。作业也可以是无代理的。
* 每个代理运行一个作业，作业包含一个或多个步骤（step）。
* 步骤可以是任务（task）或脚本，是流水线最小的构建块。
* 任务是预先打包好的脚本，执行某个动作，例如调用 REST API 或发布构建产物。
* 产物（artifact）是一次运行发布出来的文件或包集合。

## 代码评审检查清单

除了通用的代码评审检查清单，你还应当检查以下 Azure Pipelines YAML 特有的条目。

### 流水线结构

* [ ] 步骤易于理解、组件易于识别。确保流水线中每个步骤都有合适的 `displayName:` 描述。
* [ ] 在 Azure Pipelines 中查看流水线的步骤/阶段，以便更好地理解各组件。
* [ ] 如果有复杂的嵌套 YAML 文件，在 Azure Pipelines 中编辑流水线，以找到触发根文件。
* [ ] 访问所有模板文件引用，确认一处小改动不会造成破坏性变更 —— 改一个文件可能影响多条流水线。
* [ ] YAML 文件中过长的内联脚本已被移入脚本文件。

### YAML 结构

* [ ] 可复用的组件已拆分为独立的 YAML 模板。
* [ ] 变量按环境分开，存放在模板或变量组中。
* [ ] 已考虑变量值在**排队时**、**编译时**和**运行时**的变化。
* [ ] 已考虑配合 `Macro Syntax`、`Template Expression Syntax` 和 `Runtime Expression Syntax` 使用的变量语法取值。
* [ ] 变量可以在流水线运行期间变化，参数则不能。
* [ ] 流水线中未使用的变量/参数已被移除。
* [ ] 流水线是否满足阶段/作业的 `Conditions` 条件？

### 权限检查与安全

* [ ] 密钥值不应被打印到流水线输出中；调试打印密钥时应使用 `issecret`。
* [ ] 如果流水线使用 Library 中的变量组，确保流水线有权访问所创建的变量组。
* [ ] 如果流水线在其他仓库/组织中有远程任务，它有权限访问吗？
* [ ] 如果流水线试图访问安全文件（secure file），它有权限吗？
* [ ] 如果流水线部署到环境需要审批，审批人是谁？
* [ ] 是否需要保存和管理密钥？是否考虑过使用 Azure KeyVault？

### 排错技巧

* 考虑流水线中变量语法与[运行时表达式](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops\&tabs=yaml%2Cbatch#runtime-expression-syntax)的组合。这里有一个很好的示例帮助理解[变量的展开](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops\&tabs=yaml%2Cbatch#expansion-of-variables)。
* 当我们像下面这样赋值变量时，它在初始化阶段不会被设置，而是在运行时才赋值，于是我们可以根据模板运行的时机推断出一些错误。

  ```yaml
  - task: AzureWebApp@1
    displayName: 'Deploy Azure Web App : $(webAppName)'
    inputs:
      azureSubscription: '$(azureServiceConnectionId)'
      appName: '$(webAppName)'
      package: $(Pipeline.Workspace)/drop/Application$(Build.BuildId).zip
      startUpCommand: 'gunicorn --bind=0.0.0.0 --workers=4 app:app'
  ```

  错误：

  ![因初始化时机导致的授权问题](https://2540885121-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FdjAcIWTODFml0GX4YWCQ%2Fuploads%2FsucjG8brkSx6fl6DeeIH%2Fauthorization_issue_due_to_initialize_time.png?alt=media)

  把这些变量改为通过参数传入后，取值就正常加载了。

  ```yaml
    - template: steps-deployment.yaml
      parameters:
        azureServiceConnectionId: ${{ variables.azureServiceConnectionId  }}
        webAppName: ${{ variables.webAppName  }}
  ```

  ```yaml
  - task: AzureWebApp@1
    displayName: 'Deploy Azure Web App :${{ parameters.webAppName }}'
    inputs:
      azureSubscription: '${{ parameters.azureServiceConnectionId }}'
      appName: '${{ parameters.webAppName }}'
      package: $(Pipeline.Workspace)/drop/Application$(Build.BuildId).zip
      startUpCommand: 'gunicorn --bind=0.0.0.0 --workers=4 app:app'
  ```
* 调试时用 `issecret` 打印密钥

  ```bash
  echo "##vso[task.setvariable variable=token;issecret=true]${token}"
  ```

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/code-reviews/recipes/azure-pipelines-yaml.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

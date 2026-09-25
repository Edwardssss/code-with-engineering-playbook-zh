# 配方：在 Azure DevOps 流水线中运行 detect-secrets

## 概览

本文介绍如何把 Yelp detect-secrets 集成到你的 Azure DevOps 流水线中。给出的代码可以作为经典 CI 过程的一部分，或者（更推荐）作为合入 `main` 分支之前对 PR 的构建校验。

## Azure DevOps 流水线

下面给出的 Azure DevOps 流水线包含以下步骤：

1. 把 Python 3 设为默认
2. 用 pip 安装 detect-secrets
3. 运行 detect-secrets 工具
4. 把结果发布为流水线产物

   > **注意：** 这是可选的步骤，但包含结果的 .json 文件对后续排查可能有用。
5. 分析 detect-secrets 结果

   > **注意：** 这一步对 .json 文件做简单分析。如果检测到任何密钥，就以退出码 1 中断构建。

> **注意：** 下面的例子有 2 个作业：分别面向 Linux 和 Windows 代理。你不必两个都用 —— 按需调整流水线即可。
>
> **注意：** Windows 的例子没有使用最新版本的 detect-secrets。这与 detect-secrets 工具的一个 bug 有关（详见 [Issue#452](https://github.com/Yelp/detect-secrets/issues/452)）。强烈建议关注该问题的修复，并在可能时去掉 pip install 命令中的版本标签 `==1.0.3`，改用最新版本。

```yaml
trigger:
  - none

jobs:
  - job: ubuntu
    displayName: "detect-secrets on Ubuntu Linux agent"
    pool:
      vmImage: ubuntu-latest
    steps:
      - task: UsePythonVersion@0
        displayName: "Set Python 3 as default"
        inputs:
          versionSpec: "3"
          addToPath: true
          architecture: "x64"

      - bash: pip install detect-secrets
        displayName: "Install detect-secrets using pip"

      - bash: |
          detect-secrets --version
          detect-secrets scan --all-files --force-use-all-plugins --exclude-files FETCH_HEAD > $(Pipeline.Workspace)/detect-secrets.json
        displayName: "Run detect-secrets tool"

      - task: PublishPipelineArtifact@1
        displayName: "Publish results in the Pipeline Artifact"
        inputs:
          targetPath: "$(Pipeline.Workspace)/detect-secrets.json"
          artifact: "detect-secrets-ubuntu"
          publishLocation: "pipeline"

      - bash: |
          dsjson=$(cat $(Pipeline.Workspace)/detect-secrets.json)
          echo "${dsjson}"

          count=$(echo "${dsjson}" | jq -c -r '.results | length')

          if [ $count -gt 0 ]; then
            msg="Secrets were detected in code. ${count} file(s) affected."
            echo "##vso[task.logissue type=error]${msg}"
            echo "##vso[task.complete result=Failed;]${msg}."
          else
            echo "##vso[task.complete result=Succeeded;]No secrets detected."
          fi
        displayName: "Analyzing detect-secrets results"

  - job: windows
    displayName: "detect-secrets on Windows agent"
    pool:
      vmImage: windows-latest
    steps:
      - task: UsePythonVersion@0
        displayName: "Set Python 3 as default"
        inputs:
          versionSpec: "3"
          addToPath: true
          architecture: "x64"

      - script: pip install detect-secrets==1.0.3
        displayName: "Install detect-secrets using pip"

      - script: |
          detect-secrets --version
          detect-secrets scan --all-files --force-use-all-plugins > $(Pipeline.Workspace)/detect-secrets.json
        displayName: "Run detect-secrets tool"

      - task: PublishPipelineArtifact@1
        displayName: "Publish results in the Pipeline Artifact"
        inputs:
          targetPath: "$(Pipeline.Workspace)/detect-secrets.json"
          artifact: "detect-secrets-windows"
          publishLocation: "pipeline"

      - pwsh: |
          $dsjson = Get-Content $(Pipeline.Workspace)/detect-secrets.json
          Write-Output $dsjson

          $dsObj = $dsjson | ConvertFrom-Json
          $count = ($dsObj.results | Get-Member -MemberType NoteProperty).Count

          if ($count -gt 0) {
            $msg = "Secrets were detected in code. $count file(s) affected. "
            Write-Host "##vso[task.logissue type=error]$msg"
            Write-Host "##vso[task.complete result=Failed;]$msg"
          }
          else {
            Write-Host "##vso[task.complete result=Succeeded;]No secrets detected."
          }
        displayName: "Analyzing detect-secrets results"
```

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/dev-sec-ops/secrets-management/recipes/detect-secrets-ado.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

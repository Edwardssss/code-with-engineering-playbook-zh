# 工具

## 自定义 ADO

### 任务板

* AzDO：[自定义卡片](https://learn.microsoft.com/en-us/azure/devops/boards/boards/customize-cards?view=azure-devops)
* AzDO：[在任务板上添加列](https://learn.microsoft.com/en-us/azure/devops/boards/sprints/customize-taskboard?view=azure-devops#add-columns)

### 评审者策略

* 在 AzDO 中设置必需的评审者组 —— [自动包含代码评审者](https://learn.microsoft.com/en-us/azure/devops/repos/git/branch-policies?view=azure-devops#automatically-include-code-reviewers)

## 配置分支策略

1. AzDO：[配置分支策略](https://learn.microsoft.com/en-us/azure/devops/repos/git/branch-policies?view=azure-devops#configure-branch-policies)
2. AzDO：用 CLI 工具配置分支策略：
   1. [创建策略配置文件](https://learn.microsoft.com/en-us/azure/devops/cli/policy-configuration-file?view=azure-devops#create-a-policy-configuration-file)
   2. [批准数量策略](https://learn.microsoft.com/en-us/rest/api/azure/devops/policy/configurations/create?view=azure-devops-rest-5.1#approval-count-policy)
3. GitHub：[配置受保护分支](https://help.github.com/en/github/administering-a-repository/about-protected-branches)

## VS Code

### GitHub：[GitHub Pull Requests](https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-pull-request-github)

支持在 VS Code 内处理 GitHub 拉取请求。

1. 从**活动栏**打开该插件
2. 选择 **Assigned To Me**
3. 选择一个 PR
4. 在 **Description** 下，你可以选择 **Check Out** 该分支，进入**评审模式**，获得更集成的体验

### Azure DevOps：[Azure DevOps Pull Requests](https://marketplace.visualstudio.com/items?itemName=ankitbko.vscode-pull-request-azdo)

支持在 VS Code 内处理 Azure DevOps 拉取请求。

1. 从**活动栏**打开该插件
2. 选择 **Assigned To Me**
3. 选择一个 PR
4. 在 **Description** 下，你可以选择 **Check Out** 该分支，进入**评审模式**，获得更集成的体验

## Visual Studio

以下扩展可以在 Visual Studio 中配合 GitHub 或 Azure DevOps，打造集成的代码评审体验。

### GitHub：[GitHub Extension for Visual Studio](https://marketplace.visualstudio.com/items?itemName=GitHub.GitHubExtensionforVisualStudio)

提供直接在 Visual Studio 中处理 GitHub 拉取请求的扩展能力。

1. View -> Other Windows -> GitHub
2. 点击任务栏上的 **Pull Requests** 图标
3. 双击一个待处理的拉取请求

### Azure DevOps：[Pull Requests for Visual Studio](https://marketplace.visualstudio.com/items?itemName=VSIDEVersionControlMSFT.pr4vs)

直接在 Visual Studio 中处理 Azure DevOps 上的拉取请求。

1. 打开 Team Explorer
2. 点击 **Pull Requests**
3. 双击一个拉取请求 —— 打开 **Pull Request Details**
4. 如果你想在本地拿到完整改动、获得更集成的体验，点击 **Checkout**
5. 过一遍改动并留下评论

## 网页端

### Reviewable：[无缝的多轮 GitHub 评审](https://home.reviewable.io/)

支持多轮 GitHub 代码评审，带键盘快捷键等。VS Code 扩展开发中。

1. 访问 [Review Dashboard](https://reviewable.io/reviews)，查看等待你处理的评审、有新评论给你的评审等。
2. 从列表中选一个拉取请求。
3. 在浏览器、Visual Studio Code 或你配置过的任意编辑器中打开文件 —— 点击右上角的个人头像即可配置。
4. 在 "External editor link template" 下选择一个编辑器。VS Code 是选项之一，任何支持 URI 的编辑器都可以。
5. 从整体或按文件评审 diff，留下评论、代码建议等。

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/code-reviews/tools.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

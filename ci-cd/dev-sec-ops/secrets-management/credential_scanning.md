# 凭据扫描

凭据扫描是一种自动检查项目的实践，确保项目源代码中不包含任何密钥。密钥包括数据库密码、存储连接字符串、管理员登录凭据、服务主体等。

## 为什么要做凭据扫描

把密钥包含在项目源代码中风险很大，因为它可能把这些密钥暴露给不想让其获得的人。即使看起来源码只有那些本来就知道密钥的人能访问，随着项目成长这种局面也很可能改变。密钥散落在不同地方，会让它们难以管理、难以做访问控制、难以有效撤销。已提交到版本控制的密钥也更难丢弃，因为它们会永久留在源码历史中。 另一个考量是，把项目代码与基础设施和部署细节耦合起来是有局限性的，被视为不良实践。从软件设计角度看，代码应当独立于运行它所用的运行时配置，而运行时配置中包含密钥。 因此，代码与密钥之间应当有明确边界：密钥应当在源代码之外管理，并应采用凭据扫描确保这条边界永不被突破。

## 如何实施凭据扫描

理想情况下，凭据扫描应当在开发者的工作流中运行（例如通过 [git 提交前钩子](https://pre-commit.com/)）；但为了防止开发者失误，凭据扫描还必须作为持续集成过程的一部分强制执行，确保没有任何凭据被合入项目主分支。 要为项目实现凭据扫描，可以考虑：

1. 把密钥存放在专门用于保存敏感信息的外部安全存储中
2. 使用密钥扫描工具扫描仓库的完整历史，评估仓库当前状态
3. 把自动化密钥扫描工具集成到 CI 流水线，发现无意的密钥提交
4. 避免在 git 上使用 `git add .` 命令
5. 把敏感文件加入 .gitignore

## 凭据扫描架与工具

配方与场景 ——

1. [detect-secrets](./recipes/detect-secrets.md) 是一个名字很贴切的模块，用于在代码库中检测密钥。
2. 在 Azure DevOps 流水线中使用 [detect-secrets](./recipes/detect-secrets-ado.md)
3. [Microsoft Security Code Analysis 扩展](https://learn.microsoft.com/en-us/azure/security/develop/security-code-analysis-overview)

其他工具 ——

1. [CodeQL](https://securitylab.github.com/tools/codeql) —— GitHub 安全。CodeQL 让你像查询数据一样查询代码，可以写查询找出某个漏洞的所有变体
2. [Git-secrets](https://github.com/awslabs/git-secrets) —— 防止你把密码和其他敏感信息提交到 git 仓库。

## 结论

密钥管理对每个项目都必不可少。把密钥存放在外部密钥库，并把这种思维融入你的工作流，会提升你的安全水平，也会带来更干净的代码。

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/dev-sec-ops/secrets-management/credential_scanning.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

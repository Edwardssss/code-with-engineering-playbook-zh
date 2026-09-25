# 按分支管理 Azure DevOps 设置

用 [Azure DevOps Pipelines](https://azure.microsoft.com/en-us/services/devops/pipelines/)做 CI/CD 时，利用内置的[流水线变量](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/variables)来做[密钥管理](https://github.com/microsoft/code-with-engineering-playbook/blob/main/docs/CI-CD/gitops/secret-management/README.md)很方便，但用流水线变量做密钥管理有其缺点：

* *流水线变量是在引用它们的代码之外管理的。* 这很容易在源代码与密钥之间引入偏差，例如在代码中新增了对一个新密钥的引用、却忘了把它加到流水线变量里（导致令人困惑的构建中断）；或者在代码中删除了对某个密钥的引用、却忘了从流水线变量中删除它（导致令人困惑的残留流水线变量）。
* *流水线变量是全局共享状态。* 当开发者并发修改流水线变量、可能相互覆盖时，这会导致令人困惑的局面和难以排查的问题。只有一套全局流水线变量，也使得密钥无法随环境而不同（例如使用基于分支的部署模型时：`master` 用生产密钥部署，`development` 用 staging 密钥部署，依此类推）。

针对这些局限的一个解决方案，是把密钥与项目源代码一起在 Git 仓库中管理。如[密钥管理](https://github.com/microsoft/code-with-engineering-playbook/blob/main/docs/CI-CD/gitops/secret-management/README.md)中所述，不要把密钥以明文检入仓库。反过来，我们可以把密钥的加密版本加入仓库，并让 CI/CD 代理和开发者用一个预共享密钥解密密钥供本地使用。这样两头的好处都有：密钥有安全的存储，同时密钥与代码可以并列管理。

```sh
# 首先，确保我们永远不会提交明文密钥，并生成一个强加密密钥
echo ".env" >> .gitignore
ENCRYPTION_KEY="$(LC_ALL=C < /dev/urandom tr -dc '_A-Z-a-z-0-9' | head -c128)"

# 现在向 .env 文件里添加一些密钥
echo "MY_SECRET=..." >> .env

# 同时更新我们的密钥文档文件
cat >> .env.template <<< "
# enter description of your secret here
MY_SECRET=
"

# 接下来，加密明文密钥；生成的 .env.enc 文件可以安全地提交到仓库
echo "${ENCRYPTION_KEY}" | openssl enc -aes-256-cbc -md sha512 -pass stdin -in .env -out .env.enc
git add .env.enc .env.template
git commit -m "Update secrets"
```

运行 CI/CD 时，构建服务器可以通过解密来访问密钥。例如对 Azure DevOps，把 `ENCRYPTION_KEY` 配置为[密钥型流水线变量](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/variables#secret-variables)，然后在 `azure-pipelines.yml` 中加入以下步骤：

```yaml
steps:
  - script: echo "$(ENCRYPTION_KEY)" | openssl enc -aes-256-cbc -md sha512 -pass stdin -in .env.enc -out .env -d
    displayName: Decrypt secrets
```

你也可以为流水线使用[直接链接到 Azure Key Vault 的变量组](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups?view=azure-devops\&tabs=yaml#link-secrets-from-an-azure-key-vault)，把所有密钥集中在一处管理。

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/gitops/secret-management/azure-devops-secret-management-per-branch.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

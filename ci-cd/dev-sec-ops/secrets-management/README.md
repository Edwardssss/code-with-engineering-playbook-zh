# 密钥管理

密钥管理指的是用于管理数字认证凭据（如 API key、token、密码和证书）的工具与实践。这些密钥用于保护对敏感数据和服务的访问，因此它们的管理对安全至关重要。

我们应当假定所做工作的任何仓库都可能随时转为公开，并相应地保护密钥，即使仓库最初是私有的。

## 密钥管理的重要性

在现代软件开发中，应用往往需要与其他软件组件、API 和服务交互。这些交互通常需要认证，而认证通常用密钥来处理。如果这些密钥没有被恰当管理，它们就可能被暴露，导致潜在的安全泄漏。

## 密钥管理最佳实践

1. **集中式密钥存储：** 把所有密钥存放在一个集中、加密的位置。这能降低密钥丢失或暴露的风险。
2. **访问控制：** 实施严格的访问控制策略。只有被授权的实体才能访问密钥。
3. **密钥轮换：** 定期更换密钥，以降低密钥被泄露时的风险。
4. **审计跟踪：** 记录何时、由谁访问了哪个密钥。这有助于识别可疑活动。
5. **自动化密钥管理：** 把密钥的创建、轮换和删除过程自动化。这能降低人为失误的风险。

记住，密钥管理的目标，是保护敏感信息不被未授权访问和潜在的安全威胁。

## 通用做法

通用做法是把密钥存放在单独的配置文件中，不检入仓库。把这些文件加入 [.gitignore](https://git-scm.com/docs/gitignore) 以防止被检入。

每位开发者维护自己本地的文件版本；如有需要，通过私有渠道（例如 Teams 聊天）传递。

在生产系统中（以 Azure 为例），在运行进程的环境中创建密钥。我们可以通过手工编辑资源的“应用程序设置”一节来完成，但用 Azure CLI 写脚本做同样的事是省时的实用工具。更多细节见 [az webapp config appsettings](https://learn.microsoft.com/en-us/cli/azure/webapp/config/appsettings?view=azure-cli-latest)。

最佳实践是为每个你运行的环境维护独立的密钥配置，例如 dev、test、prod、local 等。

[按分支管理密钥的配方](https://github.com/microsoft/code-with-engineering-playbook/blob/main/docs/CI-CD/gitops/secret-management/azure-devops-secret-management-per-branch.md)描述了为每个环境管理独立密钥配置的简单方法。

> 注意：即使密钥只被推送到某个功能分支、从未被合并，它仍然会成为 git 历史的一部分。按[这些说明](https://help.github.com/en/github/authenticating-to-github/removing-sensitive-data-from-a-repository)移除任何敏感数据，并重新生成任何已加入仓库的密钥和其他敏感信息。如果某个密钥已经进入代码库，请轮换该密钥，使其不再有效。

## 让密钥保持机密

保护密钥所需的谨慎，既适用于我们如何获取和存储它们，也适用于我们如何使用它们。

* **不要把密钥写进日志**
* 不要把密钥放进报表
* 不要把密钥发送给其他应用 —— 无论是作为 URL 的一部分、表单的一部分，还是任何其他形式；除了用于向需要该密钥的服务发起请求之外，不要以任何方式外传

## 增强安全要求的应用

下面列出的技术提供了*良好*的安全性，也是很多语言通用的模式。它们依赖于这样一个事实：Azure 会把应用设置（环境）加密保存，直到你的应用运行。

它们*不能*防止密钥在运行时以明文形式存在于内存中。尤其是对带垃圾回收的语言，这些值可能存在得比变量的生命周期更长，在调试进程的内存转储时可能可见。

> 如果你在做有增强安全要求的应用，应当考虑使用额外技术，在应用整个生命周期中保持密钥加密。

始终定期轮换加密密钥。

## 密钥管理技术

这些技术让密钥的加载对开发者透明。

### C#/.NET

#### 现代 .NET 方案

对 .NET SDK（2.0 或更高版本），我们有 `dotnet secrets` —— .NET SDK 提供的工具，让你在开发期间管理和保护敏感信息，例如 API key、连接字符串和其他密钥。这些密钥安全地存储在你机器上，可被你的 .NET 应用访问。

```shell
# Initialize dotnet secret
dotnet user-secrets init

# Adding secret
# dotnet user-secrets set <KEY> <VALUE>
dotnet user-secrets set ExternalServiceApiKey my-api-key-12345

# Update Secret
dotnet user-secrets set ExternalServiceApiKey updated-api-key-67890

```

访问密钥：

```csharp
using Microsoft.Extensions.Configuration;

var builder = new ConfigurationBuilder()
    .AddUserSecrets<Startup>();

var configuration = builder.Build();
var externalServiceApiKey = configuration["ExternalServiceApiKey"];

```

**部署考量**

把应用部署到生产时，必须确保密钥被安全管理。以下是一些与部署相关的注意事项：

* **移除开发密钥**：部署到生产之前，从应用配置中移除任何开发用密钥。生产环境可以使用环境变量，或更安全的密钥管理方案如 Azure Key Vault 或 AWS Secrets Manager。
* **安全部署**：确保你的生产服务器是安全的，对密钥的访问是受控的。绝不要把密钥直接存放在源代码或配置文件中。
* **密钥轮换**：考虑实施密钥轮换策略，定期更新生产环境中的密钥。

#### .NET Framework 方案

使用 appSettings 元素的 [`file`](https://learn.microsoft.com/en-us/dotnet/framework/configure-apps/file-schema/appsettings/appsettings-element-for-configuration) 属性从本地文件加载密钥。

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <appSettings file="..\..\secrets.config">
  …
  </appSettings>
  <startup>
      <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.6.1" />
  </startup>
  …
</configuration>
```

访问密钥：

```c#
static void Main(string[] args)
{
    String mySecret = System.Configuration.ConfigurationManager.AppSettings["mySecret"];
}
```

在 Azure 中运行时，ConfigurationManager 会从进程环境加载这些设置。我们不需要向服务器上传密钥文件，也不需要改动任何代码。

### Node

把密钥存放在环境变量或 `.env` 文件中

```bash
$ cat .env
MY_SECRET=mySecret
```

用 [dotenv](https://www.npmjs.com/package/dotenv) 包加载并访问环境变量

```node
require('dotenv').config()
let mySecret = process.env("MY_SECRET")
```

### Python

把密钥存放在环境变量或 `.env` 文件中

```bash
$ cat .env
MY_SECRET=mySecret
```

用 [dotenv](https://pypi.org/project/python-dotenv/) 包加载并访问环境变量

```python
import os
from dotenv import load_dotenv


load_dotenv()
my_secret = os.getenv('MY_SECRET')
```

另一个读取环境变量的好库是 `environs`

```python
from environs import Env


env = Env()
env.read_env()
my_secret = os.environ["MY_SECRET"]
```

### Databricks

Databricks 可以选择用 dbutils 作为一种安全方式获取凭据，而不在 Databricks 上运行的 notebook 中暴露它们。

以下步骤给出了在 Databricks 上创建新密钥并在 notebook 中使用它们的清晰路径：

1. 在本地机器上[安装并配置 Databricks CLI](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html#set-up-the-cli)
2. [获取 Databricks 个人访问令牌](https://docs.databricks.com/api/latest/authentication.html#token-management)
3. [为密钥创建 scope](https://learn.microsoft.com/azure/databricks/security/secrets/secret-scopes)
4. [创建密钥](https://learn.microsoft.com/azure/databricks/security/secrets/)

### 校验

无论用什么编程语言，都可以对代码执行自动化的[凭据扫描](https://github.com/microsoft/code-with-engineering-playbook/blob/main/docs/CI-CD/dev-sec-ops/secrets-management/credential_scanning.md)。

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/dev-sec-ops/secrets-management/README.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

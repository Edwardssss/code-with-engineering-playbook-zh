# Terraform 代码评审

## 风格指南

开发者应当遵循 [Terraform 风格指南](https://github.com/jonbrouse/terraform-style-guide/blob/master/README.md)。

项目应当用自动化工具检查 Terraform 脚本。

## 代码分析 / Lint

### TFLint

[`TFLint`](https://github.com/terraform-linters/tflint) 是一个 Terraform linter，关注潜在错误、最佳实践等。环境中安装好 TFLint 后，可以通过 VS Code 的 [`terraform 扩展`](https://marketplace.visualstudio.com/items?itemName=mauve.terraform)调用它。

## VS Code 扩展

以下 VS Code 扩展被广泛使用。

### [`Terraform 扩展`](https://marketplace.visualstudio.com/items?itemName=mauve.terraform)

该扩展提供语法高亮、lint、格式化与校验能力。

### [`Azure Terraform 扩展`](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureterraform)

该扩展在 VS Code 内提供 Terraform 命令支持、资源关系图可视化以及 CloudShell 集成。

## 构建校验

确保在构建期间强制执行风格指南。下面的示例脚本可用于安装 Terraform 和一个 linter，然后用它们检查格式与常见错误。

```shell
#! /bin/bash
set -e

SCRIPT_DIR=$(dirname "$BASH_SOURCE")
cd "$SCRIPT_DIR"

TF_VERSION=0.12.4
TF_LINT_VERSION=0.9.1

echo -e "\n\n>>> Installing Terraform 0.12"
# Install terraform tooling for linting terraform
wget -q https://releases.hashicorp.com/terraform/${TF_VERSION}/terraform_${TF_VERSION}_linux_amd64.zip -O /tmp/terraform.zip
sudo unzip -q -o -d /usr/local/bin/ /tmp/terraform.zip

echo ""
echo -e "\n\n>>> Install tflint (3rd party)"
wget -q https://github.com/wata727/tflint/releases/download/v${TF_LINT_VERSION}/tflint_linux_amd64.zip -O /tmp/tflint.zip
sudo unzip -q -o -d /usr/local/bin/ /tmp/tflint.zip

echo -e "\n\n>>> Terraform version"
terraform -version

echo -e "\n\n>>> Terraform Format (if this fails use 'terraform fmt -recursive' command to resolve"
terraform fmt -recursive -diff -check

echo -e "\n\n>>> tflint"
tflint

echo -e "\n\n>>> Terraform init"
terraform init

echo -e "\n\n>>> Terraform validate"
terraform validate
```

## 代码评审检查清单

除了通用的[代码评审检查清单](../process-guidance/reviewer-guidance.md)，你还应当检查以下 Terraform 特有的条目。

### Provider

* [ ] 脚本中用到的所有 provider 是否都[标注了版本](https://www.terraform.io/language/providers/requirements#best-practices-for-provider-versions)，以防止未来出现破坏性变更？

### 仓库组织

* [ ] 代码是否拆分为可复用的模块？
* [ ] 模块是否在合适处拆分为独立的 `.tf` 文件？
* [ ] 仓库是否包含描述所供给架构的 `README.md`？
* [ ] 如果 Terraform 代码与应用源码混在一起，Terraform 代码是否已隔离到专门的文件夹？

### Terraform 状态

* [ ] Terraform 项目是否配置为使用 Azure Storage 作为远程状态后端？
* [ ] 远程状态后端的存储账户密钥是否存放在安全位置（例如 Azure Key Vault）？
* [ ] 项目是否配置为按环境使用不同的状态文件，且部署流水线配置为动态提供状态文件名？

### 变量

* [ ] 如果基础设施随环境不同而不同（例如 Dev、UAT、Production），环境相关参数是否通过 `.tfvars` 文件提供？
* [ ] 所有变量是否都有 `type` 信息？例如 `list(string)` 或 `string`。
* [ ] 所有变量是否都有 `description`，说明变量的用途与用法？
* [ ] 对于必须由用户提供的变量，是否**没有**设置 `default` 值？

### 测试

* [ ] 是否存在覆盖 Terraform 代码的单元测试与集成测试（例如 [`Terratest`](https://terratest.gruntwork.io/)、[`terratest-abstraction`](https://github.com/microsoft/terratest-abstraction)）？

### 命名与代码结构

* [ ] Terraform 脚本中是否正确使用了资源定义与数据源？
  * **resource：** 告知 Terraform 当前配置负责管理该对象的生命周期
  * **data：** 告知 Terraform 你只想取得对已有对象的引用，而**不**希望把它作为本配置的一部分来管理
* [ ] 资源名是否以所属 provider 的名称加下划线开头？例如来自 `postgresql` provider 的资源可能命名为 `postgresql_database`？
* [ ] `try` 函数是否只用于简单的属性引用和类型转换函数？滥用 `try` 函数来抑制错误，会导致配置难以理解和维护。
* [ ] 用于归一化类型的显式类型转换函数，是否只在模块输出中返回？显式类型转换在 Terraform 中很少有必要，因为它会在需要时自动转换类型。
* [ ] 对于包含敏感信息的字段，schema 上的 `Sensitive` 属性是否设为 `true`？这能防止字段值出现在 CLI 输出中。

### 一般建议

* 尽量避免在资源内部嵌套子配置。即使资源可以声明子元素，也请为这些资源单独建立资源块。例如在 Azure 上，把子网声明在虚拟网络内部，与把子网声明为独立资源，是两种不同做法。
* 永远不要在配置中硬编码任何值。如果某个变量需要多次作为静态值使用、且属于配置内部，请在 `locals` 中声明它们。
* 在 Azure 上创建的资源，其 `name` 不应硬编码或写死。这些名称应当是动态的、由用户通过 `variable` 块提供。这在单元测试中尤其有用 —— 多个测试并行运行时都需要在 Azure 上创建资源，但需要不同的名称（Azure 上少数资源需要全局唯一命名，例如存储账户）。
* 把在 Azure 上创建的资源 ID 通过 `output` 输出是一个好做法。在为父资源的子元素添加 dynamic block 时尤其有帮助。
* 使用 `required_providers` 块为 provider 建立依赖关系，并指定预定版本。
* 使用 `terraform` 块声明 provider 依赖的确切版本，以及本配置所需的 Terraform CLI 版本。
* 根据变量的用途和类型校验所传入的变量取值。可以通过为变量添加 `validation` 块来完成校验。
* 校验各组件的 SKU 是否正确，例如 standard 与 premium。

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/code-reviews/recipes/terraform.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

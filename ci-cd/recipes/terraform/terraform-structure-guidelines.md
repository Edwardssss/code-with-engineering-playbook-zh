# Terraform 配置的结构与测试指南

## 背景

创建基础设施配置时，遵循一致且有组织性的结构很重要，以确保代码的可维护性、可扩展性与可复用性。本节的目的是简要说明如何组织你的 Terraform 配置以实现这一目标。

## 组织 Terraform 配置

推荐的结构如下：

1. 把你想要配置的每个组件放在各自的模块文件夹中。分析你的基础设施代码，找出可拆分为可复用模块的逻辑组件。这会带给你清晰的关注点分离，并使将来加入新资源、更新已有资源或复用它们变得直截了当。关于模块及其使用时机，更多细节见 [Terraform 指南](https://developer.hashicorp.com/terraform/language/modules/develop#when-to-write-a-module)。
2. 把 `.tf` 模块文件放在每个文件夹的根下，并确保包含一个 markdown 格式的 [`README`](#生成文档)文件，它可以根据模块代码自动生成。推荐采用这种做法，因为这种文件结构会被 [Terraform Registry](https://registry.terraform.io/browse/modules) 自动识别。
3. 使用一套一致的文件来组织你的模块。虽然这取决于项目的具体需求，但一个好的例子如下：
   * **provider.tf**：根据所用插件定义 provider 列表
   * **data.tf**：定义从不同数据源读取的信息
   * **main.tf**：定义配置所需的基础设施对象（例如资源组、角色分配、容器镜像仓库）
   * **backend.tf**：后端配置文件
   * **outputs.tf**：定义导出的结构化数据
   * **variables.tf**：定义静态、可复用的取值
4. 在每个模块中包含用于文档、示例和测试的子文件夹。 文档包含模块的基本信息：它在安装什么、有哪些选项、一个用例示例等。你也可以在这里加入任何其他相关细节。 示例文件夹可以包含一个或多个如何使用该模块的示例，每个示例都使用上一步确定的那套配置文件。推荐同时提供一个 README，让人清楚理解它在实践中如何使用。 测试文件夹包含一个或多个测试示例模块的文件，以及一个说明这些测试如何[执行](https://www.hashicorp.com/blog/testing-hashicorp-terraform)的文档文件。
5. 把根模块放在一个名为 `main` 的单独文件夹中：这是配置的主要入口点。与其他模块一样，它会包含对应的配置文件。

按以上指南得到的一个配置结构示例如下：

```sh
modules
├── mlops
│   ├── doc
│   ├── example
│   ├── test
│   ├── backend.tf
│   ├── data.tf
│   ├── main.tf
│   ├── outputs.tf
│   ├── provider.tf
│   ├── variables.tf
│   ├── README.md
├── common
├── main
```

## 测试配置

测试 Terraform 配置使用 [Terratest 库](https://terratest.gruntwork.io/)。关于 Terratest 最佳实践（包括单元测试、集成测试和端到端测试）的完整指南，可以参考[这里](https://terratest.gruntwork.io/docs/testing-best-practices/unit-integration-end-to-end-test/)。

### 测试类型

* **模块 / 资源的单元测试**：为单个模块 / 资源编写单元测试，确保每个模块在隔离状态下行为符合预期。在更大、更复杂的 Terraform 配置中，它们尤其有价值（因为单个模块可被复用），而且执行时间通常更快。
* **集成测试**：这些测试验证不同模块与资源是否能按预期协同工作。

对于简单的 Terraform 配置，做大量单元测试可能过度。那种情况下集成测试可能就够了。但随着复杂度上升，单元测试会变得更有价值。

### 需要考虑的关键方面

* **语法与校验**：在开发期间或部署脚本/流水线中，使用 `terraform fmt` 与 `terraform validate` 检查语法并校验 Terraform 配置。这能确保配置格式正确、没有语法错误。
* **部署与存在性**：Terraform provider（例如 Azure provider）在 terraform apply 执行期间会做某些检查。如果 Terraform 成功应用了配置，通常就意味着指定的资源已按预期被创建或修改。你可以在代码中跳过这项校验，把重点放在下面几点更关键的特定资源配置上。
* **可能破坏功能的资源属性**：这里的预期是，我们不关心测试资源的每个属性，而是找出那些一旦被改变就可能造成系统问题的属性，例如访问策略、网络策略、服务主体权限等。
* **Key Vault 内容的校验**：确保作为资源配置的一部分存放于 Azure Key Vault 中的必需密钥、证书或密钥存在。
* **可能影响成本或位置的属性**：这可以通过断言位置、服务层级、存储设置来完成，具体取决于资源有哪些可用属性。

## 命名规范

给 Terraform 变量命名时，必须使用清晰一致、易于理解和遵循的命名规范。一般约定是使用小写字母与数字，用下划线而不是短横线，例如：“azurerm\_resource\_group”。 给资源命名时，以 provider 的名称开头，后接目标资源，用下划线分隔。例如，“azurerm\_postgresql\_server” 是 Azure provider 资源的合适命名。对于数据源，使用类似的命名规范，但列表类型要确保用复数名。例如，“azurerm\_resource\_groups” 是表示一组资源组的数据源的好名字。 变量名与输出名应当具有描述性，反映变量的用途或使用方式。用共同的前缀把相关项分组也很有帮助。例如，所有与存储账户相关的变量都可以以 “storage\_” 开头。请记住，输出应当在其作用域之外也可理解。一个有用的命名模式是 “{name}\_{attribute}”，其中 “name” 代表资源或数据源名称，“attribute” 是输出返回的属性。例如，“storage\_primary\_connection\_string” 可以是一个合法的输出名。

确保为输出和变量都加上描述，并在适用时把取值标记为 ‘default’ 或 ‘sensitive’。这些信息会被记录在生成的文档中。

## 生成文档

借助 [terraform-docs](https://terraform-docs.io/)，可以根据模块中的配置代码自动生成文档。要生成 Terraform 模块文档，进入模块文件夹并输入这条命令：

```sh
terraform-docs markdown table --output-file README.md --output-mode inject .
```

随后文档就会生成在组件根目录内。

## 结论

本节展示的方式被设计为灵活而易于使用，使添加新资源或更新已有资源变得直截了当。关注点分离也让在其他项目中复用已有组件变得容易，因为所有信息（模块、示例、文档和测试）都集中在一处。

## 相关资源

* [Terraform-docs](https://github.com/terraform-docs/terraform-docs)
* [Terraform Registry](https://registry.terraform.io/browse/modules)
* [Terraform 模块指南](https://developer.hashicorp.com/terraform/language/modules/develop#when-to-write-a-module)
* [Terratest](https://terratest.gruntwork.io/)
* [测试 HashiCorp Terraform](https://www.hashicorp.com/blog/testing-hashicorp-terraform)
* [构建基础设施 - Terraform Azure 示例](https://developer.hashicorp.com/terraform/tutorials/azure-get-started/azure-build)

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/recipes/terraform/terraform-structure-guidelines.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

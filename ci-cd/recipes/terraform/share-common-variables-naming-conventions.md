# 在 Terraform 模块间共享通用变量

## 我们要解决什么问题？

用代码部署基础设施时，把代码拆成不同模块（每个负责部署基础设施的一部分或一个组件）是常见做法。在 Terraform 中，可以用[模块](https://www.terraform.io/language/modules/develop)来实现。

在这种情况下，能共享一些通用变量并集中管理不同资源的命名约定就很有用 —— 这样即使模块之间存在依赖，将来需要修改时也能容易地重构。

举个例子，考虑两个模块：

* Network 模块，负责部署虚拟网络、子网、NSG 和私有 DNS 区域
* Azure Kubernetes Service 模块，负责部署 AKS 集群

这些模块之间存在依赖，例如：Kubernetes 集群会被部署到 Network 模块的虚拟网络中。为此，它必须引用虚拟网络的名称以及它所在的资源组。理想情况下，我们希望这些依赖尽可能松耦合，以保持模块部署上的灵活性、保持各自独立的生命周期。

本页介绍用 Terraform 解决这个问题的一种方式。

## 怎么做？

### 背景

假设我们的模块结构如下：

```sh
modules
├── kubernetes
│   ├── main.tf
│   ├── provider.tf
│   └── variables.tf
├── network
│   ├── main.tf
│   ├── provider.tf
│   └── variables.tf
```

现在假设你为开发环境部署了虚拟网络，属性如下：

* name: vnet-dev
* resource group: rg-dev-network

然后到了某个时间点，你需要把这些值注入 Kubernetes 模块，以便通过数据源获取对它的引用，例如：

```tf
data "azurem_virtual_network" "vnet" {
    name                = var.vnet_name
    resource_group_name = var.vnet_rg_name
}
```

在上面的代码片段中，虚拟网络名与资源组是通过变量定义的。这很好，但如果将来发生变化，这些变量的取值也必须随之变化 —— 而且在每个用到它们的模块里都要改。

能在集中的地方管理命名，就能确保代码将来易于重构，而无需更新所有模块。

### 关于 Terraform 变量

在 Terraform 中，每个[输入变量](https://www.terraform.io/language/values/variables)都必须在配置（或模块）级别用 `variable` 块定义。按约定，这通常在模块中的 `variables.tf` 文件里完成。该文件包含变量声明与默认值。取值可以通过变量配置文件（.tfvars）、环境变量或使用 terraform `plan` / `apply` 命令时的 CLI 参数来设置。

变量声明的一个局限是无法组合变量；为此需要使用 [locals](https://www.terraform.io/language/values/locals) 或 Terraform [内置函数](https://www.terraform.io/language/functions)。

### 通用 Terraform 模块

绕过这些局限的一种方式，是引入一个“common”模块：它不部署任何资源，只计算并输出资源名称与共享变量，并作为依赖被其他所有模块使用。

```sh
modules
├── common
│   ├── output.tf
│   └── variables.tf
├── kubernetes
│   ├── main.tf
│   ├── provider.tf
│   └── variables.tf
├── network
│   ├── main.tf
│   ├── provider.tf
│   └── variables.tf
```

*variables.tf：*

```tf
variable "environment_name" {
  type = string
  description = "The name of the environment."
}

variable "location" {
  type = string
  description = "The Azure region where the resources will be created. Default is westeurope."
  default = "westeurope"
}
```

*output.tf：*

```tf
# Shared variables
output "location" {
  value = var.location
}

output "subscription" {
  value = var.subscription
}

# Virtual Network Naming

output "vnet_rg_name" {
  value = "rg-network-${var.environment_name}"
}

output "vnet_name" {
  value = "vnet-${var.environment_name}"
}

# AKS Naming

output "aks_rg_name" {
  value = "rg-aks-${var.environment_name}"
}

output "aks_name" {
  value = "aks-${var.environment_name}"
}
```

现在，如果你对 common 模块执行 Terraform apply，就能在输出中得到所有共享/通用变量：

```sh
$ terraform plan -var environment_name="dev" -var subscription="$(az account show --query id -o tsv)"

Changes to Outputs:
  + aks_name     = "aks-dev"
  + aks_rg_name  = "rg-aks-dev"
  + location     = "westeurope"
  + subscription = "01010101-1010-0101-1010-010101010101"
  + vnet_name    = "vnet-dev"
  + vnet_rg_name = "rg-network-dev"

You can apply this plan to save these new output values to the Terraform state, without changing any real infrastructure.
```

### 使用通用 Terraform 模块

在其他任何模块中使用 common 模块极其简单。例如，你可以在 Azure Kubernetes 模块的 `main.tf` 文件中这样做：

```tf
module "common" {
  source           = "../common"
  environment_name = var.environment_name
  subscription     = var.subscription
}

data "azurerm_subnet" "aks_subnet" {
  name                 = "AksSubnet"
  virtual_network_name = module.common.vnet_name
  resource_group_name  = module.common.vnet_rg_name
}

resource "azurerm_kubernetes_cluster" "aks" {
  name                = module.common.aks_name
  resource_group_name = module.common.aks_rg_name
  location            = module.common.location
  dns_prefix          = module.common.aks_name

  identity {
    type = "SystemAssigned"
  }

  default_node_pool {
    name           = "default"
    vm_size        = "Standard_DS2_v2"
    vnet_subnet_id = data.azurerm_subnet.aks_subnet.id
  }
}
```

然后你就可以执行 `terraform plan` 和 `terraform apply` 命令进行部署！

```sh
terraform plan -var environment_name="dev" -var subscription="$(az account show --query id -o tsv)"
data.azurerm_subnet.aks_subnet: Reading...
data.azurerm_subnet.aks_subnet: Read complete after 1s [id=/subscriptions/01010101-1010-0101-1010-010101010101/resourceGroups/rg-network-dev/providers/Microsoft.Network/virtualNetworks/vnet-dev/subnets/AksSubnet]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # azurerm_kubernetes_cluster.aks will be created
  + resource "azurerm_kubernetes_cluster" "aks" {
      + dns_prefix                          = "aks-dev"
      + fqdn                                = (known after apply)
      + id                                  = (known after apply)
      + kube_admin_config                   = (known after apply)
      + kube_admin_config_raw               = (sensitive value)
      + kube_config                         = (known after apply)
      + kube_config_raw                     = (sensitive value)
      + kubernetes_version                  = (known after apply)
      + location                            = "westeurope"
      + name                                = "aks-dev"
      + node_resource_group                 = (known after apply)
      + portal_fqdn                         = (known after apply)
      + private_cluster_enabled             = (known after apply)
      + private_cluster_public_fqdn_enabled = false
      + private_dns_zone_id                 = (known after apply)
      + private_fqdn                        = (known after apply)
      + private_link_enabled                = (known after apply)
      + public_network_access_enabled       = true
      + resource_group_name                 = "rg-aks-dev"
      + sku_tier                            = "Free"

      [...] truncated

      + default_node_pool {
          + kubelet_disk_type    = (known after apply)
          + max_pods             = (known after apply)
          + name                 = "default"
          + node_count           = (known after apply)
          + node_labels          = (known after apply)
          + orchestrator_version = (known after apply)
          + os_disk_size_gb      = (known after apply)
          + os_disk_type         = "Managed"
          + os_sku               = (known after apply)
          + type                 = "VirtualMachineScaleSets"
          + ultra_ssd_enabled    = false
          + vm_size              = "Standard_DS2_v2"
          + vnet_subnet_id       = "/subscriptions/01010101-1010-0101-1010-010101010101/resourceGroups/rg-network-dev/providers/Microsoft.Network/virtualNetworks/vnet-dev/subnets/AksSubnet"
        }

      + identity {
          + principal_id = (known after apply)
          + tenant_id    = (known after apply)
          + type         = "SystemAssigned"
        }

      [...] truncated
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

> **注意：** 如果你决定从一个主 Terraform 配置文件中一次性部署所有模块，使用 common 模块同样有效，例如：

```tf
module "common" {
  source           = "./common"
  environment_name = var.environment_name
  subscription     = var.subscription
}

module "network" {
  source           = "./network"
  vnet_name        = module.common.vnet_name
  vnet_rg_name     = module.common.vnet_rg_name
}

module "kubernetes" {
  source           = "./kubernetes"
  aks_name         = module.common.aks_name
  aks_rg           = module.common.aks_rg_name
}
```

### 集中定义输入变量

如果你选择用[变量定义文件](https://www.terraform.io/language/values/variables#variable-definitions-tfvars-files)（`.tfvars`）直接在版本控制中定义变量值（例如 GitOps 场景），拥有一个 common 模块也能帮你避免在所有模块中重复定义通用变量。实际上，可以有一个在 common 模块级别定义一次的全局文件，并在 Terraform `plan` 或 `apply` 时与模块专属的变量定义文件合并。

假设结构如下：

```sh
modules
├── common
│   ├── dev.tfvars
│   ├── prod.tfvars
│   ├── output.tf
│   └── variables.tf
├── kubernetes
│   ├── dev.tfvars
│   ├── prod.tfvars
│   ├── main.tf
│   ├── provider.tf
│   └── variables.tf
├── network
│   ├── dev.tfvars
│   ├── prod.tfvars
│   ├── main.tf
│   ├── provider.tf
│   └── variables.tf
```

common 模块与其他所有模块都包含 `dev` 和 `prod` 环境的变量文件。common 模块的 `tfvars` 文件会定义与其他模块共享的所有全局变量（例如 subscription、环境名等），而每个模块的 `.tfvars` 文件只定义模块专属的取值。

然后，在执行 `terraform apply` 或 `terraform plan` 命令时，可以用以下语法合并这些文件：

```bash
terraform plan -var-file=<(cat ../common/dev.tfvars ./dev.tfvars)
```

> **注意：** 使用这种方式时，非常重要的一点是确保两个文件中没有同名变量，否则会产生错误。

## 结论

把共享变量与命名约定交给一个 common 模块统一持有之后，重构 Terraform 配置的代码库就轻松多了。设想某天你需要改变虚拟网络名称所用的命名模式：只要改 common 模块的输出文件，然后把所有模块重新 apply 一遍即可。

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/recipes/terraform/share-common-variables-naming-conventions.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

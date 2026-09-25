# 把 Terraform 输出保存到变量组

本配方仅适用于配合 Azure DevOps 使用 [terraform](https://www.terraform.io/) 的场景。它假定你熟悉 terraform 命令与 Azure Pipelines。

## 背景

当用 [terraform](https://www.terraform.io/) 自动化基础设施的供给时，通常会有一条 [Azure Pipeline](https://learn.microsoft.com/en-us/azure/devops/pipelines/?view=azure-devops) 专门用于应用 terraform 配置文件。它会创建、更新、删除 Azure 资源，以供给你的基础设施改动。

文件应用完成后，terraform 可以引用并输出一些[输出值](https://developer.hashicorp.com/terraform/language/values/outputs)（例如资源组名、应用服务名）。这些值随后通常必须被取回，并作为输入变量用于发生在其他流水线中的服务部署。

```tf
output "core_resource_group_name" {
  description = "The resource group name"
  value       = module.core.resource_group_name
}

output "core_key_vault_name" {
  description = "The key vault name."
  value       = module.core.key_vault_name
}

output "core_key_vault_url" {
  description = "The key vault url."
  value       = module.core.key_vault_url
}
```

本配方的目的就是回答这个问题：如何让 terraform 的输出值在多个流水线之间可用？

## 解决方案

一个建议的解决方案，是把输出值存放在 Library 的一个[变量组](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups?view=azure-devops\&tabs=yaml)中。变量组是存放你可能想传进 YAML 流水线的值的便利方式。此外，Library 中定义的所有资产共享同一套安全模型。你可以控制谁能定义新的库项、谁能使用已有库项。

为此，我们使用以下命令：

* [terraform output](https://developer.hashicorp.com/terraform/cli/commands/output)，从状态文件中取出输出变量的值（由 [Terraform CLI](https://developer.hashicorp.com/terraform/cli) 提供）
* [az pipelines variable-group](https://learn.microsoft.com/en-us/cli/azure/pipelines/variable-group?view=azure-cli-latest)，管理变量组（由 [Azure DevOps CLI](https://learn.microsoft.com/en-us/azure/devops/cli/?view=azure-devops) 提供）

你可以在 `terraform apply` 完成后使用以下脚本来创建/更新变量组。

### 脚本（update-variablegroup.sh）

#### 参数

| 名称                   | 说明                      |
| -------------------- | ----------------------- |
| DEVOPS\_ORGANIZATION | Azure DevOps 组织的 URI。   |
| DEVOPS\_PROJECT      | Azure DevOps 项目的名称或 ID。 |
| GROUP\_NAME          | 目标变量组的名称。               |

实现上的选择：

* 如果变量组已存在，一个可行的选择是删除并从头重建。但由于授权可能在组级别被更新过，我们倾向于避开这种做法。脚本改为移除目标组中的所有变量，再用最新值把它们加回去。权限不受影响。
* 变量组不能为空，必须至少包含一个变量。因此先创建一个临时 uuid 值来绕开这个问题，等变量更新完成后再删掉它。

```bash
#!/bin/bash

set -e

export DEVOPS_ORGANIZATION=$1
export DEVOPS_PROJECT=$2
export GROUP_NAME=$3

# configure the azure devops cli
az devops configure --defaults organization=${DEVOPS_ORGANIZATION} project=${DEVOPS_PROJECT} --use-git-aliases true

# get the variable group id (if already exists)
group_id=$(az pipelines variable-group list --group-name ${GROUP_NAME} --query '[0].id' -o json)

if [ -z "${group_id}" ]; then
    # create a new variable group
    tf_output=$(terraform output -json | jq -r 'to_entries[] | "\(.key)=\(.value.value)"')
    az pipelines variable-group create --name ${GROUP_NAME} --variables ${tf_output} --authorize true
else
    # get existing variables
    var_list=$(az pipelines variable-group variable list --group-id ${group_id})

    # add temporary uuid variable (a variable group cannot be empty)
    uuid=$(cat /proc/sys/kernel/random/uuid)
    az pipelines variable-group variable create --group-id ${group_id} --name ${uuid}

    # delete existing variables
    for row in $(echo ${var_list} | jq -r 'to_entries[] | "\(.key)"'); do
        az pipelines variable-group variable delete --group-id ${group_id} --name ${row} --yes
    done

    # create variables with latest values (from terraform)
    for row in $(terraform output -json | jq -c 'to_entries[]'); do
        _jq()
        {
            echo ${row} | jq -r ${1}
        }

        az pipelines variable-group variable create --group-id ${group_id} --name $(_jq '.key') --value $(_jq '.value.value') --secret $(_jq '.value.sensitive') 
    done

    # delete temporary uuid variable
    az pipelines variable-group variable delete --group-id ${group_id} --name ${uuid} --yes
fi
```

## 向 Azure DevOps 认证

上一条脚本中使用的大多数命令都要与 Azure DevOps 交互，因此需要认证。你可以用运行中流水线使用的 `System.AccessToken` 安全令牌认证，把它赋给名为 `AZURE_DEVOPS_EXT_PAT` 的环境变量，如下例所示（更多信息见 [Azure Pipeline YAML 中的 Azure DevOps CLI](https://learn.microsoft.com/en-us/azure/devops/cli/azure-devops-cli-in-yaml?view=azure-devops#authenticate-with-azure-devops)）。

此外你会注意到，我们还使用了[预定义变量](https://learn.microsoft.com/en-us/azure/devops/pipelines/build/variables)来指定 Azure DevOps 组织与项目（分别是 `System.TeamFoundationCollectionUri` 与 `System.TeamProjectId`）。

```yaml
  - task: Bash@3
    displayName: 'Update variable group using terraform outputs'
    inputs:
      targetType: filePath
      arguments: $(System.TeamFoundationCollectionUri) $(System.TeamProjectId) "Platform-VG"
      workingDirectory: $(terraformDirectory)
      filePath: $(scriptsDirectory)/update-variablegroup.sh
    env:
      AZURE_DEVOPS_EXT_PAT: $(System.AccessToken)
```

| 系统变量                                                                                                                                                                         | 说明                    |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------- |
| [System.AccessToken](https://learn.microsoft.com/en-us/azure/devops/pipelines/build/variables?view=azure-devops\&tabs=yaml#systemaccesstoken)                                | 携带运行中构建所用安全令牌的特殊变量。   |
| [System.TeamFoundationCollectionUri](https://learn.microsoft.com/en-us/azure/devops/pipelines/build/variables?view=azure-devops\&tabs=yaml#system-variables-devops-services) | Azure DevOps 组织的 URI。 |
| [System.TeamProjectId](https://learn.microsoft.com/en-us/azure/devops/pipelines/build/variables?view=azure-devops\&tabs=yaml#system-variables-devops-services)               | 本次构建所属项目的 ID。         |

## Library 安全

Library 项的角色已定义，而这些角色的成员资格决定你可以在这些项上执行什么操作。

| Library 项角色   | 说明                                                                                                                                                                  |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Reader        | 可以查看该项。                                                                                                                                                             |
| User          | 在编写构建或发布流水线时可以使用该项。例如，要让变量组在发布流水线中使用，你必须是它的 ‘User’。                                                                                                                 |
| Administrator | 还可以管理该项所有其他角色的成员资格。创建某项的用户会自动被加入该项的 Administrator 角色。默认情况下，以下组会被加入 Library 的 Administrator 角色：Build Administrators、Release Administrators 和 Project Administrators。 |
| Creator       | 可以在 Library 中创建新项，但该角色不包含 Reader 或 User 权限。Creator 角色无法管理其他用户的权限。                                                                                                   |

使用 `System.AccessToken` 时，将用服务账户 `<ProjectName> Build Service` 的身份访问 Library。

请确保在 `Pipelines > Library > Security` 一节中，该服务账户在 `Library` 或 `Variable Group` 级别拥有 `Administrator` 角色，以便创建/更新/删除变量（更多信息见 [Library of assets](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/?view=azure-devops)）。

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/recipes/terraform/save-output-to-variable-group.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

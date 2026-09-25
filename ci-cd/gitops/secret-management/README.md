# 用 GitOps 做密钥管理

GitOps 项目以 git 仓库为中心，被视为管理基础设施与应用的唯一事实来源。这些基础设施和应用需要通过密钥来安全地访问系统的其他资源。 把明文密钥提交到 git 仓库是不可接受的，即使这些仓库只对你的团队和组织私有。团队在使用 GitOps 时需要一种安全的方式处理密钥。

用 GitOps 管理密钥有很多方式，高层上可分为两类：

1. 在 git 仓库中存放加密后的密钥
2. 引用存放在外部密钥库中的密钥

> **长话短说**：引用外部密钥库中的密钥是推荐做法。它更容易编排密钥轮换，也更容易随多集群和/或多个团队扩展。

## 在 Git 仓库中存放加密密钥

在这种方式下，开发者用公钥手动加密密钥，而只有运行在目标集群中的自定义 Kubernetes 控制器才能解密。这种方式的流行工具有 [Bitnami Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets)、[Mozilla SOPS](https://github.com/mozilla/sops)。

所有密钥加密工具都有以下共同点：

* 密钥变更通过在 GitOps 仓库中做改动来管理，提供很好的可追溯性
* 所有密钥都可以通过在 GitOps 中做改动来轮换，无需访问集群
* 它们支持完全断网的 GitOps 场景
* 密钥以加密形式存放在 GitOps 仓库中；如果私钥泄露且攻击者能访问仓库，所有密钥都可被解密

### [Bitnami Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets)

Sealed Secrets 使用非对称加密来加密密钥。Kubernetes 控制器生成一对密钥（私钥-公钥），并把私钥以 Kubernetes secret 的形式存放在集群的 `etcd` 数据库中。开发者用 [Kubeseal CLI](https://github.com/bitnami-labs/sealed-secrets#public-key--certificate) 在提交到 git 仓库之前封存密钥。

使用 Sealed Secrets 的一些要点：

* [支持私钥自动轮换](https://github.com/bitnami-labs/sealed-secrets#sealing-key-renewal)，可用于强制重新加密密钥
  * 由于[封存密钥会自动更新](https://github.com/bitnami-labs/sealed-secrets#sealing-key-renewal)，密钥需要从集群预取，或者把集群配置为在更新时把封存密钥存到备用位置
* 控制器可以强制实施命名空间级别的多租户支持
* 封存密钥时，开发者需要能够连接集群控制面以获取公钥，或者必须把公钥显式共享给开发者
* 如果集群中的私钥由于某种原因丢失，所有密钥都需要重新加密，随后生成新的密钥对
* 无法随多集群扩展，因为每个集群都需要一个拥有自己密钥对的控制器
* 只能加密 `secret` 资源类型
* Flux 文档在 Azure Key Vault 示例上存在不一致

### [Mozilla SOPS](https://github.com/mozilla/sops)

SOPS（Secrets OPerationS）是一个加密工具，支持 YAML、JSON、ENV、INI 和 BINARY 格式，可用 AWS KMS、GCP KMS、Azure Key Vault、age 和 PGP 加密，且不限于 Kubernetes。它支持与一些常见的密钥管理系统集成，包括 Azure Key Vault —— 用一个或多个密钥管理系统存放**用于加密密钥的密钥**，而不是密钥本身。

使用 SOPS 的一些要点：

* [Flux](https://toolkit.fluxcd.io/guides/mozilla-sops/#azure) 对 SOPS 有原生支持，可在集群侧解密
* 提供额外一层安全，因为用于解密的私钥被保护在外部密钥库中
* 要用 Helm CLI 做加密，需要（[Helm Secrets](https://github.com/jkroepke/helm-secrets)）插件
* 需要（[KSOPS](https://github.com/viaduct-ai/kustomize-sops)）（[kustomize-sopssecretgenerator](https://github.com/goabout/kustomize-sopssecretgenerator)）插件才能与 Kustomization 配合
* 无法随更大的团队扩展，因为每位开发者都必须自己加密密钥
* 公钥足以创建全新文件。但解密和编辑已有文件需要私钥，因为 SOPS 会对所有值计算 MAC。如果只用公钥去添加或删除一个字段，整个文件都需要删除重建
* 支持多种类型的密钥，可在连接与断网状态下使用。一个密钥可以有一组密钥，并会尝试用它们全部解密。

## 引用存放在外部密钥库中的密钥（推荐）

这种方式依靠 [Azure Key Vault](https://learn.microsoft.com/en-us/azure/key-vault/general/overview) 这类密钥管理系统来保存密钥，而仓库中的 git 清单引用密钥库中的密钥。开发者不对仓库中的文件做任何密码学操作。运行在目标集群中的 Kubernetes operator 负责从密钥库拉取密钥，并让其以 Kubernetes secret 或挂载到 pod 的密钥卷的形式可用。

以下所有工具都有共同点：

* 密钥不存放在仓库中
* 支持 Prometheus 指标以提升可观测性
* 支持与 Kubernetes Secrets 同步
* 支持 Linux 与 Windows 容器
* 提供企业级外部密钥管理
* 易随多集群与更大的团队扩展
* 两种方案都支持用 Azure Active Directory（Azure AD）[服务主体](https://learn.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals)或[托管身份](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview)来做[密钥库认证](https://learn.microsoft.com/en-us/azure/key-vault/general/authentication)。

关于密钥轮换的思路，见 [Pod 中环境变量与挂载密钥的轮换](https://github.com/microsoft/code-with-engineering-playbook/blob/main/docs/CI-CD/gitops/secret-management/secret-rotation-in-pods.md)。

关于如何用服务主体认证私有容器镜像仓库，见：[已认证的私有容器镜像仓库](#已认证的私有容器镜像仓库)。

### [Azure Key Vault Provider for Secrets Store CSI Driver](https://github.com/Azure/secrets-store-csi-driver-provider-azure)

[Azure Key Vault](https://docs.microsoft.com/en-us/azure/key-vault/general/overview) Provider（AKVP）配合 [Kubernetes secret store CSI Driver](https://github.com/kubernetes-sigs/secrets-store-csi-driver)，让你能获取存放在 Azure Key Vault 实例中的密钥内容，并使用 Secrets Store CSI driver 接口把它们挂载到 Kubernetes pod。它通过 CSI Inline volume 把密钥/密钥/证书挂载到 pod。

Azure Key Vault Provider for Secrets Store CSI Driver 的[安装指南](https://secrets-store-csi-driver.sigs.k8s.io/getting-started/installation.html)。

CSI driver 需要通过服务主体或托管身份（推荐）访问 Azure Key Vault。要让这种访问安全，你可以利用 [Azure AD Workload Identity](https://github.com/Azure/azure-workload-identity)（推荐）或 [AAD Pod Identity](https://github.com/Azure/aad-pod-identity)。请注意 AAD pod identity 即将被 workload identity 取代。

关于 AKVP 配合 SSCSID 的产品组链接：

1. ESO / SSCSID 之间的差异（[GitHub Issue](https://github.com/external-secrets/external-secrets/issues/478)）
2. K8S 上的密钥管理演讲（[这里](https://www.youtube.com/watch?v=EW25WpErCmA)）（原生 Secrets、Vault.io，以及 ESO 与 SSCSID 的对比）

优点：

* 通过 SecretProviderClass CRD 支持 pod 的可移植性
* 支持密钥自动轮换，可按**集群**自定义同步间隔。
* 似乎微软的选择（Secrets Store CSI driver 由[微软](https://github.com/kubernetes-sigs/secrets-store-csi-driver)和 Kubernetes-SIG 大量贡献）

缺点：

* [缺少断网场景支持](https://github.com/kubernetes-sigs/secrets-store-csi-driver/issues/446)：节点离线时 SSCSID 无法拉取密钥，因此挂载卷失败，导致断网时无法扩缩和重启 pod
* AKVP 在非 Azure 环境中只能通过[服务主体](https://azure.github.io/secrets-store-csi-driver-provider-azure/docs/configurations/identity-access-modes/service-principal-mode/#configure-service-principal-to-access-keyvault)访问 Key Vault
* [包含服务主体凭据的 Kubernetes Secret](https://azure.github.io/secrets-store-csi-driver-provider-azure/docs/configurations/identity-access-modes/service-principal-mode/)需要创建在与应用 pod 相同的命名空间中。如果多个命名空间中的 pod 需要用同一个 SP 访问 Key Vault，就需要在每个命名空间中都创建这个 Kubernetes Secret。
* GitOps 仓库必须在 SecretProviderClass 中包含 Key Vault 的名称
* 必须把密钥作为卷挂载，才能同步为 Kubernetes Secrets
* 占用更多资源（4 个 pod：CSI Storage driver 与 provider），且是 daemonset —— 未测试 RPS/资源占用

### [External Secrets Operator 配合 Azure Key Vault](https://external-secrets.io/)

External Secrets Operator（ESO）是一个开源的 Kubernetes operator，可以从外部密钥库（例如 Azure Key Vault）读取密钥，并把它们同步为 Kubernetes Secrets。与 CSI Driver 不同，ESO 控制器在集群上把密钥创建为 K8s secret，而不是把它们作为卷挂载到 pod。

使用 ESO Azure Key Vault provider 的文档见[这里](https://external-secrets.io/v0.8.1/provider/azure-key-vault/)。

ESO 需要通过服务主体或托管身份（通过 [Azure AD Workload Identity](https://github.com/Azure/azure-workload-identity)（推荐）或 [AAD Pod Identity](https://github.com/Azure/aad-pod-identity)）访问 Azure Key Vault。

优点：

* 支持密钥自动轮换，可按**密钥**自定义同步间隔。
* 组件拆分为面向命名空间（ExternalSecret、SecretStore）与集群级（ClusterSecretStore、ClusterExternalSecret）的不同 CRD，让同步在面对不同部署/pod 时更好管理
* （Cluster）SecretStore 的服务主体密钥可以放在只有 ESO 能访问的命名空间中（见 [Shared ClusterSecretStore](https://external-secrets.io/v0.7.1/guides/multi-tenancy/)）。
* 资源占用低（单个 pod）—— 未测试 RPS/资源占用。
* 开源且贡献度高（[GitHub](https://github.com/external-secrets/external-secrets)）
* 通过 K8s 自身 API 支持把 Secrets 作为卷挂载（见[这里](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-files-from-a-pod)）
* 部分支持断网场景：由于 ESO 使用的是原生 K8s secret，集群可以离线，且断网对重启和扩缩 pod 没有影响

缺点：

* GitOps 仓库必须在 SecretStore / ClusterSecretStore 中包含 Key Vault 的名称，或者用一个 ConfigMap 链接到它
* 必须以 K8s secret 形式创建密钥

## 相关资源

* [Sealed Secrets 配合 Flux v2](https://toolkit.fluxcd.io/guides/sealed-secrets/)
* [Mozilla SOPS 配合 Flux v2](https://toolkit.fluxcd.io/guides/mozilla-sops/)
* [用 Argo CD 做密钥管理](https://argo-cd.readthedocs.io/en/stable/operator-manual/secret-management/)
* [密钥管理工作流](https://www.youtube.com/watch?v=-k6HEXaE75k)

## 附录

### 已认证的私有容器镜像仓库

一种认证私有容器镜像仓库（例如 ACR）的做法：

1. 在 Pod 级别使用 `dockerconfigjson` 类型的 Kubernetes Secret 配合 `ImagePullSecret`（这也可以在[命名空间级别](https://kubernetes.io/docs/concepts/containers/images/#referring-to-an-imagepullsecrets-on-a-pod)定义）

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/gitops/secret-management/README.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。原文附录仅列出 1 条即中断。
{% endhint %}

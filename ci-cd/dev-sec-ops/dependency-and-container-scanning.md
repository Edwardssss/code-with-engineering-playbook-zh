# 依赖与容器扫描

依赖与容器扫描的目的，是在操作系统、语言包与应用包中搜找漏洞。

## 为什么要做依赖与容器扫描

容器镜像是云原生环境中标准的应用交付形式。 由于社区有大量镜像可选，我们往往会选择一个社区基础镜像，然后向其中添加我们需要的包，而这些包也可能来自社区。 这些来源不限的依赖可能会给我们的镜像和应用引入漏洞。

## 如何实施依赖与容器扫描

包含有安全漏洞的软件的镜像，在运行时就变得可被利用。在 CI 流水线中构建镜像时，镜像扫描必须作为构建通过的必要条件。未通过扫描的镜像，绝不应被推送到生产环境的容器镜像仓库。

依赖与容器扫描最佳实践：

1. **基础镜像** —— 如果你的镜像是基于第三方基础镜像构建的，请验证以下几点：
   * 镜像来自知名公司或开源组织。
   * 它托管在声誉良好的镜像仓库上。
   * Dockerfile 可得，并检查其中安装的依赖。
   * 镜像更新频繁 —— 旧镜像可能不包含最新的安全更新。
2. **移除非必需软件** —— 从最小基础镜像开始，只安装应用所需的工具、库和配置文件。 避免安装以下工具，或如果已存在就移除它们：
   * 网络工具与客户端：例如 wget、curl、netcat、ssh。
   * Shell：例如 sh、bash。注意，移除 shell 也会阻止在运行时使用 shell 脚本。可能时应改用可执行文件。
   * 编译器与调试器。它们只应当用于构建和开发容器，绝不应用于生产容器。
3. **容器镜像应当是不可变的** —— 在镜像构建期间下载并包含所有必需依赖。
4. **扫描软件依赖中的漏洞** —— 如今几乎没有哪个软件项目不包含某种形式的外部库、依赖或开源代码。 虽然它让开发团队能专注于应用代码，但依赖同时也带来了一个可预期的缺点：真实应用的安全态势如今躺在了它身上。 要在项目依赖中检测漏洞，可使用容器扫描工具 —— 它们在分析过程中会扫描软件依赖（见“依赖与容器扫描架与工具”）。

## 依赖与容器扫描架与工具

1. [Trivy](https://github.com/aquasecurity/trivy) —— 一个简单而全面的容器漏洞扫描器（不支持 Windows 容器）
2. [Aqua](https://www.aquasec.com/solutions/azure-container-security/) —— 面向 AKS、ACI 与 Windows 容器上运行应用的依赖与容器扫描。可与 AzDO 流水线集成。
3. [Dependency-Check Plugin for SonarQube](https://github.com/dependency-check/dependency-check-sonar-plugin) —— 本地部署的依赖扫描
4. [Mend（原名 WhiteSource）](https://www.mend.io/) —— 开源扫描软件

## 结论

容器这样强大的技术应当谨慎使用。只安装应用所需的最小依赖，了解你的应用正在使用的软件依赖，并用容器与依赖扫描工具保持长期维护。

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/dev-sec-ops/dependency-and-container-scanning.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

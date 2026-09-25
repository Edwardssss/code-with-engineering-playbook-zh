# 在流水线中复用 Dev Container

假设有个仓库带有本地开发容器（dev container），其中包含开发所需的全部工具。那么把该容器复用于持续集成流水线中运行这些工具，说得通吗？

## 在流水线中构建 Dev Container 的选项

在流水线中构建 devcontainer 有三种方式：

* 用 [GitHub - devcontainers/ci](https://github.com/devcontainers/ci)，根据 `devcontainer.json` 构建容器。示例见：[devcontainers/ci · Getting Started](https://github.com/devcontainers/ci/blob/main/docs/github-action.md#getting-started)。
* 用 [GitHub - devcontainers/cli](https://github.com/devcontainers/cli)，与上面相同，但直接使用底层 CLI，不经过 tasks。
* 用 `docker build` 构建 `DockerFile`。这种方式会忽略 `devcontainer.json` 中指定的所有配置/特性。

## 待选方案

* 在原生环境中运行 CI 流水线
* 在本地构建镜像、在 dev container 中运行 CI 流水线
* 配合容器镜像仓库、在 dev container 中运行 CI 流水线

以下是各种做法的利弊：

### 在原生环境中运行 CI 流水线

| 优点             | 缺点                                              |
| -------------- | ----------------------------------------------- |
| 可以使用任何可用的流水线任务 | 需要保持两套工具及其版本同步                                  |
| 不需要容器镜像仓库      | 根据所需工具/依赖，启动可能耗时                                |
| 代理会始终带着最新的安全补丁 | 每次运行 CI 流水线都应构建 dev container，以验证分支内的改动没有破坏任何东西 |

### 不使用镜像缓存的 Dev Container 中运行 CI 流水线

| 优点                                   | 缺点                           |
| ------------------------------------ | ---------------------------- |
| 实用工具脚本开箱即可使用                         | 考虑到被构建的分支可能有改动，每次运行都需要重新构建容器 |
| CI 上使用的（lint 或单元测试）规则与本地相同           | 容器里并非所有东西都是 CI 流水线需要的¹       |
| 对开发者没有意外：本地输出（例如 lint）在 CI 上也会相同     | 部分流水线任务将不可用                  |
| 所有工具及其版本定义在一处                        | 每次流水线运行都构建镜像会很慢²             |
| 工具/依赖已经就位                            |                              |
| dev container 本身也在被测试：既包含所有新工具，也没有损坏 |                              |

> ¹：可以通过导出只含 CI 流水线所需工具的那一层来减小容器体积
>
> ²：可以通过加入镜像缓存（不使用容器镜像仓库）来缓解

### 配合镜像仓库、在 Dev Container 中运行 CI 流水线

| 优点                                                                                                                                                                                 | 缺点                           |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| 实用工具脚本开箱即可使用                                                                                                                                                                       | 考虑到被构建的分支可能有改动，每次运行都需要重新构建容器 |
| 对开发者没有意外：本地输出（例如 lint）在 CI 上也会相同                                                                                                                                                   | 容器里并非所有东西都是 CI 流水线需要的¹       |
| CI 上使用的（lint 或单元测试）规则与本地相同                                                                                                                                                         | 部分流水线任务将不可用²                 |
| 所有工具及其版本定义在一处                                                                                                                                                                      | 需要访问容器镜像仓库来在流水线中托管镜像³        |
| 工具/依赖已经就位                                                                                                                                                                          |                              |
| dev container 本身也在被测试：既包含所有新工具，也没有损坏                                                                                                                                               |                              |
| 发布由 `devcontainer.json` 构建出的容器后，你可以在 `devcontainer.json` 的 cacheFrom 中引用它（见[文档](https://containers.dev/implementors/json_reference/#image-specific)）。这样 VS Code 在构建时会使用已发布的镜像作为层缓存 |                              |

> ¹：可以通过导出只含 CI 流水线所需工具的那一层来减小容器体积。这需要在不使用 tasks 的情况下构建镜像
>
> ²：在 AzDO 中使用容器作业（container jobs）就可以用上所有任务（据我所知如此）。参考：[Dockerizing DevOps V2 - AzDO container jobs - DEV Community](https://dev.to/eliises/dockerizing-devops-v2-azdo-container-jobs-3hbf)
>
> ³：在 GH Actions 中，可以直接用默认的 GitHub Actions token 访问 GHCR，无需单独配置镜像仓库，见下面的例子。 **注意：** 这不会把 `Dockerfile` 与 `devcontainer.json` 一起构建

```yaml
    - uses: whoan/docker-build-with-cache-action@v5
        id: cache
        with:
          username: $GITHUB_ACTOR
          password: "${{ secrets.GITHUB_TOKEN }}"
          registry: docker.pkg.github.com
          image_name: devcontainer
          dockerfile: .devcontainer/Dockerfile
```

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/recipes/reusing-devcontainers-within-a-pipeline.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

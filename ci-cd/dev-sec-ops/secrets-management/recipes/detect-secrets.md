# 配方：detect-secrets

## 背景

[`detect-secrets`](https://github.com/Yelp/detect-secrets) 是一个开源项目，它用启发式规则与规则集扫描[很宽范围](https://github.com/Yelp/detect-secrets#currently-supported-plugins)的密钥。我们可以通过简单的 [Python 插件 API](https://github.com/Yelp/detect-secrets/blob/a9dff60/detect_secrets/plugins/base.py#L27-L49) 用自定义规则和启发式扩展这个工具。

与其他凭据扫描工具不同，`detect-secrets` 被调用时不会尝试检查项目完整的 git 历史，而是扫描项目当前状态。这意味着它运行得很快，因此非常适合用在持续集成流水线中。

`detect-secrets` 引入了“基线文件”（baseline file）的概念，也就是仓库中已存在的已知密钥列表；我们可以配置它在运行时忽略这些预先存在的密钥。这让把该工具渐进地引入已有项目变得容易。

基线文件还提供了简单方便地处理误报的方式。我们可以在基线文件中把误报加入白名单，让后续调用忽略它。

## 安装

```sh
# 安装系统依赖：diff、jq、python3（Linux 系操作系统）
apt-get install -y diffutils jq python3 python3-pip

# 安装系统依赖：diff、jq、python3（Windows）
winget install Python.Python.3
choco install diffutils jq -y

# 安装 detect-secrets 工具
python3 -m pip install detect-secrets

# 运行工具，建立已知密钥列表
# 仔细审查这个文件并把它检入仓库
detect-secrets scan > .secrets.baseline
```

## 提交前钩子

推荐在开发环境中把 `detect-secrets` 作为 Git 提交前钩子使用。

首先，按 [`pre-commit` 安装说明](https://pre-commit.com/#install)在开发环境中安装该工具。

然后，把以下内容加入你的 `.pre-commit-config.yaml`：

```yaml
repos:
-   repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
    -   id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
```

## 在 CI 流水线中使用

```sh
# 备份已知密钥列表
cp .secrets.baseline .secrets.new

# 找出仓库中的所有密钥
detect-secrets scan --baseline .secrets.new $(find . -type f ! -name '.secrets.*' ! -path '*/.git*')

# 如果已知密钥与新检测到的密钥之间存在差异，则中断构建
list_secrets() { jq -r '.results | keys[] as $key | "\($key),\(.[$key] | .[] | .hashed_secret)"' "$1" | sort; }

if ! diff <(list_secrets .secrets.baseline) <(list_secrets .secrets.new) >&2; then
  echo "Detected new secrets in the repo" >&2
  exit 1
fi
```

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/dev-sec-ops/secrets-management/recipes/detect-secrets.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

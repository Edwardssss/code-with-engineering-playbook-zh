# 拉取请求模板

下面是原手册提供的 PR 模板，可直接放到仓库的 `.github/pull_request_template.md` 中使用。

```markdown
# [工作项 ID](工作项链接)

关于如何为本仓库贡献的更多信息，请访问这个[页面](https://github.com/microsoft/code-with-engineering-playbook/blob/main/CONTRIBUTING.md)

## 描述

---

> 应当包含对改动的简洁描述（缺陷修复还是功能特性）、改动的影响，以及解决方案概要

## 重现缺陷与验证解决方案的步骤

---
> 仅当本次工作是修复缺陷时才适用。如果本次工作是功能或用户故事，请删除本节。
> 提供发现缺陷的环境细节，以及重现缺陷的详细步骤。
> 描述应当详细到团队成员足以确认该缺陷不再出现。

## PR 检查清单

---

> 用下面的检查清单确认你的分支已准备好提交 PR。如果某项不适用，留空即可。

- [ ] 我已相应更新了文档。
- [ ] 我已为改动添加了测试。
- [ ] 所有新旧测试均已通过。
- [ ] 我的代码符合本项目的代码风格。
- [ ] 我运行了 lint 检查，我的改动没有产生新的错误或警告。
- [ ] 我已确认没有其他针对同一改动/变更的开启状态 PR。

## 是否引入破坏性变更？

---

- [ ] 是
- [ ] 否

> 如果引入了破坏性变更，请在下方描述影响以及现有应用的迁移路径。

## 测试

---

> - 测试与验证你的代码的说明：
>   - 测试使用的操作系统。
>   - 使用了哪些测试集。
>   - 已尝试的测试场景描述。

## 相关日志或输出

---

> - 用本节附上能证明你的改动正常工作/健康的图片
> - 如果打印了某些内容，请提供截图
> - 需要分享长日志时，请上传到：
>  `(存储账户)/pr-support/attachments/(PR 编号)/(你的文件)，使用 [Azure 存储资源管理器](https://azure.microsoft.com/en-us/features/storage-explorer/)` 或 [portal.azure.com](https://portal.azure.com)，然后在这里插入链接。

## 其他信息或已知依赖

---

> - 对本 PR 重要的其他信息或已知依赖。
> - 本 PR 之后待完成的 TODO。
```

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/code-reviews/pull-request-template.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

# 密钥轮换

密钥轮换是刷新应用所用密钥的过程。 向 Azure 服务认证的最佳方式是使用托管身份（managed identity），但在某些场景下无法这么做。那些情况下会使用访问密钥或密钥。你应当定期轮换访问密钥或密钥。

## 为什么要轮换密钥

密钥是一种资产，因此有泄露或被窃取的可能。通过轮换密钥，我们就能撤销任何可能已被泄露的密钥。因此密钥应当频繁轮换。

## 托管身份

Azure 托管身份由 Azure 自动颁发，用于标识各个资源，可以代替密钥和密码用于认证。使用托管身份的吸引力在于免除了对密钥和凭据的管理。它们不需要出现在开发者机器上，也不需要检入版本控制，而且不需要轮换。托管身份被认为比替代方案更安全，是推荐选择。

## 如何实施密钥轮换

如果无法使用 Azure 托管身份，本节与以下章节将说明如何实现密钥轮换：

为了促成密钥的频繁轮换，应定义一个自动化的定期密钥轮换过程。 密钥轮换过程在重启应用以引入新密钥时，可能导致停机。一个常见解决方案是同时准备两个版本的密钥，也称为蓝/绿密钥轮换。手上备有第二个密钥，我们就可以在前一个密钥被撤销之前用新密钥启动应用的第二个实例，从而避免任何停机。

## 密钥轮换架与工具

1. 对只使用一套认证凭据的资源做密钥轮换，[点这里](https://learn.microsoft.com/en-us/azure/key-vault/secrets/tutorial-rotation)
2. 对使用两套认证凭据的资源做密钥轮换，[点这里](https://learn.microsoft.com/en-us/azure/key-vault/secrets/tutorial-rotation-dual?tabs=azure-cli)

## 结论

刷新密钥很重要，它能确保你的密钥仍然是秘密，同时不给应用带来停机。

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/dev-sec-ops/secrets-management/secrets_rotation.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

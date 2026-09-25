# Pod 中密钥的轮换

本文介绍在 Kubernetes pod 中用环境变量和挂载密钥做密钥轮换的几种方式。

## 通过 secretKeyRef 把密钥映射到环境变量

如果我们通过 `secretKeyRef` 把 K8s 原生 secret 映射到环境变量，然后轮换了密钥，即使 K8s 原生 secret 已被更新，环境变量也不会更新。我们需要重启 Pod 才能让改动生效。[Reloader](https://github.com/stakater/Reloader) 用一个 K8S 控制器解决了这个问题。

```yaml
...
    env:
        - name: EVENTHUB_CONNECTION_STRING
          valueFrom:
            secretKeyRef:
              name: poc-creds
              key: EventhubConnectionString
...
```

## 通过 volumeMounts 映射密钥（ESO 方式）

如果我们通过卷挂载把 K8s 原生 secret 映射进去，轮换密钥时文件会被更新。应用需要能在不重启的前提下接收到这些改动（这很可能需要应用中有自定义逻辑来支持）。这样就无需重启应用。

```yaml
...
    volumeMounts:
    - name: mounted-secret
      mountPath: /mnt/secrets-store
      readOnly: true
  volumes:
  - name: mounted-secret
    secret:
      secretName: poc-creds
...
```

## 通过 volumeMounts 映射密钥（AKVP SSCSID 方式）

SSCSID 聚焦于把外部密钥挂载进 CSI。因此轮换密钥时文件会被更新。应用需要能在不重启的前提下接收到这些改动（这很可能需要应用中有自定义逻辑来支持）。这样就无需重启应用。

```yaml
...
    volumeMounts:
    - name: app-secrets-store-inline
      mountPath: "/mnt/app-secrets-store"
      readOnly: true
  volumes:
  - name: app-secrets-store-inline
    csi:
      driver: secrets-store.csi.k8s.io
      readOnly: true
      volumeAttributes:
        secretProviderClass: akvp-app
      nodePublishSecretRef:
        name: secrets-store-sp-creds
...
```

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/CI-CD/gitops/secret-management/secret-rotation-in-pods.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}

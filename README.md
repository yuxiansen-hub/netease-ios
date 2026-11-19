# opencode TWA

把 `https://opencode.xsyu.top/` 包成安卓 app，用 **Trusted Web Activity**（TWA）：
外壳是原生 app，页面仍是网页，所以前端一改、手机上立刻生效，不用重新发版。

## 为什么不在这里构建

本机是 2 GB 内存的 Lightsail，11-17 跑一次 Gradle 构建就把整机拖到断网 8 小时。
所有编译都在 **GitHub Actions**（`.github/workflows/android.yml`）里跑，
本机只负责改代码、推 tag。

## 构建

推到 `main` 或手动 `workflow_dispatch` 就会出 APK，产物在 Actions 的
`opencode-twa-apk` artifact 里；打 `v*` tag 会额外挂到 Release。

签名信息走 GitHub Secrets，仓库里不放私钥：

| Secret | 说明 |
| --- | --- |
| `KEYSTORE_B64` | `release.jks` 的 base64 |
| `KEYSTORE_PASSWORD` | store 密码 |
| `KEY_ALIAS` | `opencode` |
| `KEY_PASSWORD` | key 密码（PKCS12 下与 store 密码相同） |

证书 SHA-256（写死在 `strings.xml` 的 `asset_statements` 里，也是服务器端
`/.well-known/assetlinks.json` 的内容）：

```
F9:8C:94:12:23:AC:53:A5:92:1D:5D:5B:76:98:75:F7:96:A3:E3:29:6E:59:51:C8:67:F9:36:79:FE:8C:8C:C3
```

## TWA 的硬前提

1. 站点必须是 HTTPS（Cloudflare 已经满足）；
2. `https://opencode.xsyu.top/.well-known/assetlinks.json` 必须能公开访问，
   且包含上面的指纹和包名 `top.xsyu.opencode`；
3. APK 必须用对应私钥签名。

三条都满足时，打开 app 是**没有地址栏的全屏网页**；任何一条不满足，
Chrome 会自动降级成带地址栏的 Custom Tab（不会白屏）。

## 平板

`SCREEN_ORIENTATION=any` + `resizeableActivity` 默认开启，横竖屏和分屏都跟随系统；
页面本身的平板断点由 `opencode-chat/index.html` 负责（821–1180 px 收窄聊天列）。

## 注意

- 包名 `top.xsyu.opencode` 不能改，否则 `assetlinks.json` 和签名指纹都要同步。
- 换签名 = 换指纹 = 必须同步改服务器端 assetlinks，否则 app 会退回带地址栏模式。
- 这是自用包，没上架商店；装的时候需要在系统里允许「安装未知来源应用」。

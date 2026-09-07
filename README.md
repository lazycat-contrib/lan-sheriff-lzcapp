# LAN Sheriff for LazyCat

[LAN Sheriff](https://github.com/291-Group/LAN-Sheriff) watches which servers your network talks to. Self-hosted, no account. Observes only; blocks nothing. Built by 291 Group.

- 包名：`community.lazycat.app.lan-sheriff`（默认 community 前缀）
- 初始版本：`1.0.1`，镜像：`ghcr.io/291-group/lan-sheriff:1.0.1`
- 系统：lzcos 1.5.2+，LPK v2，Linux amd64
- 仅发布喵喵商店，图标由用户提供

## 使用

安装时授予声明的网络与 Compose Override 权限，打开应用并设置访问密码，无需外部账号。保持镜像原有启动参数，服务监听宿主 `0.0.0.0:2911`，懒猫入口转发至 `http://host.lzcapp:2911/`。局域网也可直接访问微服的 2911 端口，该入口使用应用自身的密码保护，不经过懒猫入口鉴权。

数据库及配置持久化到 `/lzcapp/var/data`，对应容器 `/data`。不集成免密登录或懒猫文件选择器拦截，不添加与本应用无关的时区、注册或认证向导开关。

## 网络与权限

保留 Compose 的 `network_mode: host`，避免监控对象变成容器桥接网络。通过原生 `netadmin: true` 与 Compose Override 的 `cap_drop: ALL`、`cap_add: [NET_RAW, NET_ADMIN]` 保留抓包所需的两项能力，不启用 privileged。

包声明 `net.host`、`net.lan`、`net.internet`、`net.admin` 和 `compose.override`。Compose Override 是兼容性不受保证的运行时覆盖机制；若未来提交官方商店，需向官方说明并备案。

正常抓包应显示 Patrol Mode。Deputy Mode 表示抓包能力受限，需检查权限及运行日志。宿主网络不等于自动获得全网流量：普通交换机端口通常只能看到本机及广播等可见流量；全网观测需要网关、SPAN/镜像端口等合适位置。详见上游 [VANTAGE-POINTS.md](https://github.com/291-Group/LAN-Sheriff/blob/v1.0.1/docs/VANTAGE-POINTS.md)。

2911 是宿主固定端口，请避免端口冲突或同时运行多个实例。保留上游 `lan-sheriff status --data-dir /data` 健康检查（60s 间隔、10s 超时、3 次重试、20s 启动宽限）。上游该版本的 status 可直接读取数据库，健康检查成功不代表 HTTP API 或抓包正常；安装后还需验证网页和 Patrol Mode。

## 构建与发布

```sh
lzc-cli project release -o dist/community.lazycat.app.lan-sheriff-v1.0.1.lpk
lzc-cli lpk info dist/community.lazycat.app.lan-sheriff-v1.0.1.lpk
```

GitHub Actions 每日发现稳定版，也可手动触发。使用 `ca-x/lazycat-github-action@v1` 可复用工作流，`publish` 策略，镜像代理模式，核对 amd64 镜像与上游 digest 一致。默认代理为 `ghcr.1ms.run`，可用 GitHub Variable `LAZYCAT_GHCR_MIRROR` 覆盖。

必需 Secrets：`APPSTORE_URL`、`APPSTORE_TOKEN`；可选 `PRIVATE_STORE_GROUP_CODES`。共享组织 Secrets 必须授权本仓库，不需要官方商店凭据。Release 保存 `community.lazycat.app.lan-sheriff-v<version>.lpk`，校验下载 URL 和 SHA256 后提交喵喵商店；相同或更高版本自动跳过。

构建及 Release 校验不能代替微服实机验证。安装后应检查权限授权、入口访问、密码设置、Patrol Mode、实际流量可见性和数据持久化。

上游作者：291 Group；上游许可证：AGPL-3.0。

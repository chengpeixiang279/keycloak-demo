# Keycloak 登录集成 Demo

演示一个前端单页应用如何接入 [Keycloak](https://www.keycloak.org/) 实现统一身份认证（OIDC / SSO）。页面加载后自动检查登录状态，未登录可跳转 Keycloak 登录，登录后展示用户信息，并支持刷新 Token、退出登录。

## 功能

- 初始化 Keycloak JS Adapter，检查单点登录（SSO）状态
- 未登录跳转 Keycloak 登录页
- 登录后展示用户名、邮箱、姓名等 ID Token 信息
- 刷新 Token（updateToken）
- 退出登录（logout）

## 技术栈

- HTML / CSS / JavaScript（原生）
- Keycloak JS Adapter（keycloak-js）

## 前置条件

- Docker（用于本地启动 Keycloak 服务）

## 快速开始

### 1. 启动 Keycloak

```bash
docker run -d -p 8080:8080 \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=admin \
  quay.io/keycloak/keycloak:26.0.0 \
  start-dev
```

启动后访问管理控制台 http://localhost:8080/admin ，使用 admin / admin 登录。

### 2. 创建 Realm 与 Client

- 左侧「Create realm」创建名为 `demo` 的 realm（也可直接用默认的 master realm）
- 进入 `Clients` → `Create client`，Client ID 填 `demo-client`
- Client 类型选 `OpenID Connect`，下一步
- 关闭 `Client authentication`（即 Public client）
- 在 `Valid redirect URIs` 中填 `http://localhost:*`（或 `*`）
- 保存

### 3. 创建测试用户

在 `Users` → `Add user` 创建用户，填写用户名、邮箱，并在 `Credentials` 里设置密码（设置时关闭 Temporary 选项）。

### 4. 运行 Demo

1. 按需修改 `index.html` 中 `Keycloak` 构造函数的配置：
   - `url`：Keycloak 地址（默认 http://localhost:8080）
   - `realm`：realm 名（默认 demo）
   - `clientId`：client ID（默认 demo-client）
2. 浏览器打开 `index.html`
3. 点击「使用 Keycloak 登录」，跳转 Keycloak 登录后回到页面，即可看到用户信息

## 核心代码说明

```js
const keycloak = new Keycloak({
    url: 'http://localhost:8080',   // Keycloak 服务地址
    realm: 'demo',                   // Realm
    clientId: 'demo-client'          // Client
});

keycloak.init({ onLoad: 'check-sso' }).then(authenticated => {
    // authenticated 为 true 表示已登录
});
```

- `keycloak.init({ onLoad: 'check-sso' })`：初始化并检查 SSO 登录状态，不强制跳转
- `keycloak.login()`：跳转 Keycloak 登录页
- `keycloak.logout()`：退出登录
- `keycloak.updateToken(minValidity)`：刷新 Token
- `keycloak.tokenParsed`：解析后的 ID Token（含用户名、邮箱等）

## 说明

- `onLoad` 为 `check-sso` 时页面不强制登录，先展示登录按钮；如需访问即强制登录，改为 `login-required`
- keycloak-js 通过 CDN 引入，生产环境建议改为从自己的 Keycloak 服务器加载 `/js/keycloak.js`，以保证前后端版本一致

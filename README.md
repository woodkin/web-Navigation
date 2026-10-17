# CF WebStack Admin

一个可部署到 Cloudflare Workers + D1 的 WebStack 风格网址导航项目，包含前台、后台、管理员登录、网站/分类/站点设置管理。

## 功能

- 前台：分类展示、站点卡片、关键词搜索、响应式布局
- 后台：管理员登录、站点新增/编辑/删除、分类新增/编辑/删除、排序、站点启用/隐藏
- 设置：网站标题、副标题、Logo URL、搜索引擎
- 数据：Cloudflare D1 持久化
- 安全：HttpOnly 会话 Cookie、HMAC 签名、服务端鉴权、SameSite=Strict、写请求 Origin 检查
- 不依赖第三方前端 CDN；页面资源全部自带

## 部署前准备

需要 Node.js 20+、npm，以及 Cloudflare 账号。建议用 GitHub 管理代码。

### 1. 安装依赖

```bash
npm install
npm install -D wrangler
```

### 2. 创建 D1 数据库

```bash
npx wrangler d1 create webstack-nav-db
```

命令会返回数据库 ID。把 `wrangler.jsonc` 中 `database_id` 的 `REPLACE_WITH_YOUR_D1_DATABASE_ID` 替换成实际 ID。

### 3. 初始化数据库

```bash
npx wrangler d1 execute webstack-nav-db --file=./schema.sql
```

### 4. 设置管理员密码和会话密钥

设置管理员密码（请使用强密码）：

```bash
npx wrangler secret put ADMIN_PASSWORD
```

设置随机会话签名密钥：

```bash
npx wrangler secret put SESSION_SECRET
```

输入 `SESSION_SECRET` 时，建议使用至少 32 字节的随机字符串。可在本地生成：
- macOS/Linux：`openssl rand -base64 48`
- PowerShell：`[Convert]::ToBase64String((1..48 | ForEach-Object { Get-Random -Maximum 256 }))`

请勿把这两个值提交到 Git 仓库。不要使用默认密码。

### 5. 本地开发

```bash
npm run dev
```

本地开发时 Wrangler 会提供本地地址。第一次启动前建议先执行数据库初始化命令；本地 D1 与线上 D1 是分开的。

### 6. 部署

```bash
npm run deploy
```

部署后访问根路径 `/`，后台为 `/admin`。

## 重要说明

- `wrangler.jsonc` 的 D1 `database_id` 必须替换，否则无法启动数据库绑定。
- 部署到生产环境后，首次登录使用你通过 `wrangler secret put ADMIN_PASSWORD` 设置的密码。
- `SESSION_SECRET` 用于签名会话 Cookie，不能泄露。
- 本项目是单管理员模式，没有公开注册和找回密码功能。
- 当前图标字段使用图片 URL / favicon URL；可直接填 `https://example.com/favicon.ico`。
- 如果你使用自定义域名，请在 Cloudflare 绑定域名并确保通过 HTTPS 访问。
- 删除分类前请先把其中站点移动到其他分类；数据库会阻止删除仍被使用的分类。
- 这是一个可运行的起点，正式公开使用前建议先在测试环境验证，并定期备份 D1。

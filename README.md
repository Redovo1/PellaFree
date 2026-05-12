# ⭐ Star 一下支持项目 ⭐

> 动动发财手点点 Star ⭐

基于 **Cloudflare Workers** 部署的 **PellaFree 自动续期 + 自动重启脚本**

---

## 📌 功能说明

* ✅ 多账号自动续期
* ✅ 支持定时 Cron 触发
* ✅ 支持手动触发（网页 / API）
* ✅ 支持服务器一键重启
* ✅ Telegram 通知推送

---

## ⚠️ 注意事项

> ❗ 仅支持 **Web 创建的机器（PellaFree）**
> ❗ 不支持 API / 其他来源创建的实例

---

## 📝 注册地址

👉 [https://www.pella.app/](https://www.pella.app/)

---

## 🚀 部署方式

使用 **Cloudflare Workers** 部署

---

## 🔧 环境变量配置

| 变量名            | 说明                 |
| -------------- | ------------------ |
| `PASSWORD`     | 访问密码               |
| `ACCOUNT`      | 账号列表（格式见下）         |
| `TG_BOT_TOKEN` | Telegram Bot Token |
| `TG_CHAT_ID`   | Telegram Chat ID   |

### 📄 ACCOUNT 格式示例

```
user1@gmail.com-----password1
user2@gmail.com-----password2
```

---

## ⏰ 定时任务（Cron）

这个版本已经把 **续期** 和 **重启** 做成两个 Cron：

| Cron 表达式 | 作用 | 北京时间说明 |
| --- | --- | --- |
| `0 */4 * * *` | 自动续期 | 每 4 小时执行一次 |
| `0 8,20 * * *` | 自动重启 / redeploy | 每天 16:00 和 04:00 执行 |

在 Cloudflare Workers 里把这两条都加进去即可。

> 注意：Cloudflare Cron 使用的是 UTC 时间，不是北京时间。上面的 `0 8,20 * * *` 已经按北京时间 16:00 / 04:00 换算好了。

---

## 🌐 使用方式

### 1️⃣ 浏览器手动续期

```
https://xxx.workers.dev/
```

---

### 2️⃣ API 触发续期（所有账号）

```bash
curl "https://xxx.workers.dev/?pwd=你的密码"
```

---

## 🔄 重启功能

### 1️⃣ 浏览器手动重启

```
https://xxx.workers.dev/
```

---

### 2️⃣ API 重启所有账号服务器

```bash
curl "https://xxx.workers.dev/restart?pwd=你的密码"
```

---

### 3️⃣ API 重启指定账号服务器

```bash
curl "https://xxx.workers.dev/restart?pwd=你的密码&account=user@gmail.com"
```

---


## 🧭 保姆级部署教程（Cloudflare Workers 网页版）

> 目标：把这个项目重新部署到另一个 Cloudflare 账号上，得到一个新的 `xxx.workers.dev`，让它自己定时续期和重启。
>
> 这个仓库只用来保存 Worker 代码和教程，**不需要配置 GitHub Actions**。

### 1️⃣ 创建 Worker

1. 登录 Cloudflare Dashboard
2. 左侧进入 **Workers & Pages**
3. 点击 **Create application**
4. 选择 **Worker**
5. 随便起一个名字，例如：`keeppellaalive`
6. 创建完成后，进入这个 Worker 的编辑页面

### 2️⃣ 粘贴代码

1. 打开本仓库里的 `_worker.js`
2. 复制全部内容
3. 回到 Cloudflare Worker 编辑器
4. 删除默认示例代码
5. 粘贴 `_worker.js` 的全部内容
6. 点击 **Deploy** 保存

### 3️⃣ 配置环境变量

进入 Worker 的 **Settings → Variables**，添加下面变量：

| 变量名 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `PASSWORD` | Variable 或 Secret | 是 | 访问管理面板和 API 时用的密码 |
| `ACCOUNT` | Secret | 是 | PellaFree 账号列表 |
| `TG_BOT_TOKEN` | Secret | 否 | Telegram Bot Token，用来推送结果 |
| `TG_CHAT_ID` | Secret | 否 | Telegram 接收消息的 chat id |

推荐把 `ACCOUNT`、`TG_BOT_TOKEN`、`TG_CHAT_ID` 都设成 **Secret**，不要公开显示。

### 4️⃣ 填写 ACCOUNT

`ACCOUNT` 一行一个账号，格式必须是：

```txt
邮箱-----密码
```

单账号示例：

```txt
user1@gmail.com-----password1
```

多账号示例：

```txt
user1@gmail.com-----password1
user2@gmail.com-----password2
```

中间是 **5 个短横线**：`-----`，不要写成空格、逗号或冒号。

### 5️⃣ 配置 Cron 定时任务

进入 Worker 的 **Settings → Triggers → Cron Triggers**，添加两条：

```txt
0 */4 * * *
0 8,20 * * *
```

含义：

- `0 */4 * * *`：每 4 小时自动续期
- `0 8,20 * * *`：每天北京时间 16:00 和 04:00 自动重启 / redeploy

这个 Worker 会自动判断是哪一条 Cron 触发的：

- 续期 Cron → 执行 `renew`
- 重启 Cron → 执行 `restart`

注意：代码里是按 `0 8,20 * * *` 这个表达式识别“重启任务”的。除非你同步改 `_worker.js`，否则这条重启 Cron 不要随便换时间。

### 6️⃣ 手动测试

假设你的 Worker 地址是：

```txt
https://keeppellaalive.xxx.workers.dev
```

手动打开管理面板：

```txt
https://keeppellaalive.xxx.workers.dev/
```

手动触发续期：

```bash
curl "https://keeppellaalive.xxx.workers.dev/?pwd=你的PASSWORD"
```

手动触发重启：

```bash
curl "https://keeppellaalive.xxx.workers.dev/restart?pwd=你的PASSWORD"
```

只重启某一个账号：

```bash
curl "https://keeppellaalive.xxx.workers.dev/restart?pwd=你的PASSWORD&account=user@gmail.com"
```

如果配置了 Telegram，执行后应该会收到续期 / 重启结果通知。

### 7️⃣ 迁移旧 Worker 时的顺序

如果你是从旧 Cloudflare 账号迁移过来，建议按这个顺序：

1. 先在新账号部署 Worker
2. 配好 `PASSWORD` / `ACCOUNT` / Telegram 变量
3. 手动测试续期接口
4. 手动测试重启接口
5. 确认 Telegram 有结果通知
6. 再添加两条 Cron
7. 观察一次自动执行正常后，再去旧 Worker 删除 Cron Trigger

不要一上来就关旧的；先让新的跑通，避免中间断档。

---
## 📸 效果展示

### 🔔 通知效果

![通知效果](img/通知效果.png)

### ⏰ Cron 设置

![Cron 定时](img/Cron定时.png)

### ⚙️ 环境变量

![环境变量](img/环境变量.png)

### 📢 注意事项 
- ❗ 仅支持 Web 创建的机器

![注意事项](img/注意：只支持web创建的机器.png)

---

## 💬 Telegram 通知说明

配置 `TG_BOT_TOKEN` 和 `TG_CHAT_ID` 后：

* ✅ 续期结果推送
* ✅ 重启结果推送
* ❌ 失败告警

---

## ❤️ 支持项目

如果这个项目对你有帮助：

👉 点个 **Star ⭐** 支持一下吧！

---
## ⚠️ 免责声明

本项目仅供学习研究使用。使用本脚本产生的任何后果由使用者自行承担。请遵守 www.pella.app 的服务条款。

<div align="center">

<sub><strong>L0NE-6 · DAILY SIGNAL</strong></sub>

# 🧠 ChatGLM Daily

**智谱清言每日登录积分、活动、每日任务与新手任务自动化，包含客户端登录态读取、令牌续期、积分回读和多账号汇总。**

> 当前版本：**v1.0.2**（2026-11-04） · 脚本逻辑：**v1.0.1** · 核心能力：签到、活动、任务结算、余额回读

[简体中文](README.md) · [English](docs/README.en.md) · [繁體中文](docs/README.zh-Hant.md) · [日本語](docs/README.ja.md) · [한국어](docs/README.ko.md) · [Português (BR)](docs/README.pt-BR.md)

<a href="https://github.com/L0NE-6/ChatGLM-Daily/releases"><img src="https://img.shields.io/github/v/release/L0NE-6/ChatGLM-Daily?style=for-the-badge&label=Release&color=2ea44f&logo=github" alt="Release" /></a>
<img src="https://img.shields.io/github/stars/L0NE-6/ChatGLM-Daily?style=for-the-badge&label=Stars&color=FFC75F&logo=github" alt="Stars" />
<img src="https://img.shields.io/badge/Python-3.11-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python" />
<img src="https://img.shields.io/badge/Platform-Windows%20%7C%20Linux%20%7C%20Qinglong-4EAA25?style=for-the-badge&logo=linux&logoColor=white" alt="Platform" />
<img src="https://img.shields.io/github/license/L0NE-6/ChatGLM-Daily?style=for-the-badge&label=License&color=F472B6" alt="License" />
<a href="https://github.com/L0NE-6/ChatGLM-Daily/releases"><img src="https://img.shields.io/badge/Download-Releases-2ea44f?style=for-the-badge&logo=github" alt="Download" /></a>

[🚫 使用边界](#usage-policy) · [✨ 30 秒速览](#overview) · [🧭 这是什么](#what-is-this) · [🔁 工作原理](#protocol) · [🔑 凭据](#credentials) · [🚀 本地](#local) · [🐧 青龙](#qinglong) · [☁️ Actions](#actions) · [⌨️ CLI](#cli-output) · [🧾 结果码](#result-codes) · [✅ 成功判据](#success) · [🧪 测试](#tests) · [🧱 设计原则](#design) · [🔒 隐私](#privacy) · [📁 目录](#layout) · [📦 版本](#release) · [⚠️ 免责声明](#disclaimer) · [☕ 赞赏](#donate) · [💬 反馈与 Star](#feedback)

</div>

---

<a id="usage-policy"></a>
## 🚫 使用边界与转载声明

> [!IMPORTANT]
> **官方版本永久免费。** 未经作者书面许可，禁止付费包装、搬运引流、冒充作者，或使用作者身份与二维码制造官方授权假象。

本项目仓库、脚本、部署说明和 Release 下载始终免费公开。未经作者明确书面许可，禁止将其包装为付费资源、内部版、独家版、会员权益、收费课程、付费部署或包更新服务；禁止搬运到其他账号、群组、频道、网盘、电商或付费平台进行涨粉、截流、倒卖和引流。

如需商业合作、品牌授权、联合分发或使用作者名义提供付费服务，请先取得作者明确书面授权。完整规则见 [NOTICE.md](NOTICE.md)。

---

<a id="overview"></a>
## ✨ 30 秒速览

| 维度 | 说明 |
| :--- | :--- |
| **做什么** | 完成每日登录积分、会员活动、季节活动、每日任务和新手任务。 |
| **输入** | Windows 客户端登录态，或 `CHATGLM_ACCOUNTS` / 单账号 access、refresh、device_id。 |
| **输出** | 每账号签到状态、任务完成数、奖励积分、余额和中性状态说明。 |
| **部署** | Windows 登录工具、青龙面板、GitHub Actions、Linux 定时任务。 |
| **依赖** | `requests`；登录工具额外使用 `cryptography`。 |
| **成功判据** | 签到链路得到明确到账或已领结果，并读取余额或任务状态；活动空窗不伪造成功。 |

---

<a id="what-is-this"></a>
## 🧭 这是什么

仓库包含主签到脚本和 Windows 登录工具：

- `chatglm_daily.py`：登录积分、会员/季节活动、任务中心、新手任务、余额回读。
- `chatglm_login.py`：读取桌面客户端 Cookies、使用 Windows DPAPI 解密登录态，并输出账号变量。

访客账号没有积分体系，会被明确跳过。多账号的 access token、refresh token、device_id、新手任务状态和运行结果互相隔离。

---

<a id="protocol"></a>
## 🔁 协议、工作原理与回读

默认基址为 `https://chatglm.cn`，可通过 `CHATGLM_BASE_URL` 覆盖。主要路径：

```text
GET/POST /chatglm/member-api/member/score_activity_open
POST     /chatglm/member-api/member/daily_login_score
POST     /chatglm/member-api/member/score_activity_draw
POST     /chatglm/member-api/member/summer_day_give_score
GET      /chatglm/agent-api/task/center
POST     /chatglm/agent-api/task/click
POST     /chatglm/agent-api/task/complete
```

请求头按客户端形态生成：

- `X-Timestamp`
- `X-Nonce`
- `X-Sign`
- `X-Device-Id`
- `X-App-Version: 2.0.7`

签到流程先查活动是否开放，再领取登录积分。任务中心按 `click → complete` 结算；新手任务只在本地状态未完成时执行，完成后持久化。access token 失效时使用 refresh token 续期并保存轮换结果。

---

<a id="credentials"></a>
## 🔑 凭据与环境变量

### 账号与运行

| 变量 | 必填 | 说明 |
| :--- | :---: | :--- |
| `CHATGLM_ACCOUNTS` | 二选一 | 多账号，每行 `备注\|access\|refresh\|device_id` |
| `CHATGLM_TOKEN` | 二选一 | 单账号 access token |
| `CHATGLM_REFRESH_TOKEN` | 否 | 单账号 refresh token |
| `CHATGLM_DEVICE_ID` | 否 | UUID；缺省时尝试从 JWT 读取 |
| `CHATGLM_ACCOUNT_JSON` | 否 | 账号 JSON 字符串 |
| `CHATGLM_ACCOUNT_FILE` | 否 | 账号 JSON 文件 |
| `CHATGLM_STATE_FILE` | 否 | 状态文件，默认 `chatglm_state.json` |
| `CHATGLM_PROXY` | 否 | HTTP(S) 或 SOCKS 代理 |
| `CHATGLM_NO_NOTIFY` | 否 | `1` 表示关闭推送 |
| `CHATGLM_BASE_URL` | 高级 | 默认 `https://chatglm.cn` |

### 推送

| 通道 | 变量 |
| :--- | :--- |
| PushPlus | `PLUSPLUS_TOKEN` |
| Bark | `BARK_URL` |
| 企业微信 | `WECOM_WEBHOOK` 或 `QYWX_TOKEN` |
| 钉钉 | `DINGTALK_WEBHOOK` |

---

<a id="local"></a>
## 🚀 本地快速开始

```bash
python -m pip install -r requirements.txt
```

读取 Windows 客户端登录态：

```bash
python chatglm_login.py
python chatglm_login.py --paste
python chatglm_login.py --json-file accounts.json
python chatglm_login.py --write-env .env
```

手动配置单账号：

```text
CHATGLM_TOKEN=ACCESS_TOKEN
CHATGLM_REFRESH_TOKEN=REFRESH_TOKEN
CHATGLM_DEVICE_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

多账号：

```text
CHATGLM_ACCOUNTS=
主账号|ACCESS_TOKEN|REFRESH_TOKEN|DEVICE_UUID
小号|ACCESS_TOKEN|REFRESH_TOKEN|DEVICE_UUID
```

运行：

```bash
python chatglm_daily.py
python chatglm_daily.py --query
python chatglm_daily.py --only 1
```

---

<a id="qinglong"></a>
## 🐧 青龙面板

```bash
pip3 install requests cryptography
```

在青龙环境变量中填写 `CHATGLM_ACCOUNTS`，再创建任务：

```text
10 8 * * * python3 chatglm_daily.py
```

续期后的令牌、设备号和新手任务完成状态写入 `chatglm_state.json`。该文件已被 `.gitignore` 屏蔽；青龙脚本目录应可写且可持久化。

---

<a id="actions"></a>
## ☁️ GitHub Actions

工作流位于 [`.github/workflows/daily.yml`](.github/workflows/daily.yml)。当前只启用 `workflow_dispatch`；定时段保持注释，示例 cron `10 0 * * *` 对应北京时间 08:10。

| Secret | 必填 | 说明 |
| :--- | :---: | :--- |
| `CHATGLM_ACCOUNTS` | 二选一 | 多账号令牌 |
| `CHATGLM_TOKEN` | 二选一 | 单账号 access token |
| `CHATGLM_REFRESH_TOKEN` | 否 | 单账号 refresh token |
| `CHATGLM_DEVICE_ID` | 否 | 单账号设备 UUID |
| `CHATGLM_PROXY` | 否 | 代理 |
| `PLUSPLUS_TOKEN` / `BARK_URL` / `WECOM_WEBHOOK` / `DINGTALK_WEBHOOK` | 否 | 推送 |

Actions 不保留 `chatglm_state.json`。refresh token 轮换后需要重新导出并更新 Secret；长期运行优先使用可持久化目录。

---

<a id="cli-output"></a>
## ⌨️ CLI 与输出

| 命令 | 作用 |
| :--- | :--- |
| `python chatglm_daily.py` | 完整签到、活动、每日任务和新手任务 |
| `python chatglm_daily.py --query` | 只查询余额和任务进度 |
| `python chatglm_daily.py --only 1` | 只处理第一个账号 |
| `python chatglm_daily.py --skip-newbie` | 本轮跳过新手任务 |
| `python chatglm_daily.py --no-notify` | 本轮关闭推送 |

输出按账号分块，包含签到动作、活动状态、任务完成数、奖励积分、余额和最终摘要。活动未开始、已结束或已领取会作为中性状态显示。

---

<a id="result-codes"></a>
## 🧾 结果码

| 结果 | 含义 |
| :--- | :--- |
| 退出码 `0` | 至少一个非访客账号执行成功 |
| 退出码 `1` | 账号缺失、全部失败，或只匹配到访客账号 |
| 到账 | 登录积分返回可解析的正积分，或任务结算得到明确奖励 |
| 已领 | 服务端明确表示今天已领取 |
| 未开启 | 活动不在开放期，属于中性结果 |
| 未确认 | 响应缺少积分或任务读数，不能写成成功 |

---

<a id="success"></a>
## ✅ 成功判据

每日签到成功需要满足以下任一条件：

- 登录积分接口返回可解析的正积分。
- 服务端明确表示今天已领取。

任务结算需要看到任务状态变化或奖励数值；余额回读需要能解析真实积分字段。活动未开启、已结束或已领取不会伪造奖励，也不会单独推翻已确认的签到结果。

---

<a id="tests"></a>
## 🧪 测试

仓库当前没有独立 `tests/` 目录，发布前验证命令：

```bash
python -m py_compile chatglm_daily.py chatglm_login.py
python chatglm_daily.py --help
python chatglm_login.py --help
git diff --check
```

本节明确记录测试边界：以上检查验证语法、CLI 和文档改动，不替代真实账号的签到、活动、任务与余额回读。

---

<a id="design"></a>
## 🧱 设计原则

- **账号隔离**：令牌、设备号、状态和错误按账号处理。
- **状态优先**：活动开放状态、任务状态和余额均以服务端读数为主。
- **中性状态不冒充成功**：未开启、已领、未确认分开报告。
- **令牌轮换可恢复**：续期后立即保存新值。
- **最小依赖**：主签到流程仅需 `requests`，DPAPI 登录工具才使用 `cryptography`。

---

<a id="privacy"></a>
## 🔒 隐私与数据边界

- 仓库不包含账号、令牌、设备号或 Cookie。
- 日志不输出完整 access token 和 refresh token。
- `chatglm_state.json`、`.env` 和客户端 Cookies 不应提交。
- Windows DPAPI 读取必须在当前 Windows 用户环境完成。
- 提交 Issue 前请移除令牌、设备 ID、账号标识和 Cookie。

---

<a id="layout"></a>
## 📁 目录结构

```text
ChatGLM-Daily/
├── .github/workflows/daily.yml
├── assets/
├── docs/
├── chatglm_daily.py
├── chatglm_login.py
├── CHANGELOG.md
├── LICENSE
├── NOTICE.md
├── requirements.txt
└── README.md
```

---

<a id="release"></a>
## 📦 版本与发布

- 当前发布版：`v1.0.2`，文档发行版；签到脚本逻辑为 `v1.0.1`。
- 历史版本保留在 [Releases](https://github.com/L0NE-6/ChatGLM-Daily/releases)。
- 变更记录见 [CHANGELOG.md](CHANGELOG.md)。

---

<a id="disclaimer"></a>
## ⚠️ 免责声明

本项目用于个人学习与合法自动化实践。使用者应自行确认账号、服务条款和部署环境；因误配置、凭据泄露、服务策略变化或账号异常产生的风险由使用者自行承担。

---

<a id="donate"></a>
## ☕ 支持与投喂：微信与支付宝赞赏码

脚本完全免费，赞赏纯自愿，不影响任何功能。

<p align="center">
  <img src="assets/donate-wechat.png" width="260" alt="微信赞赏码" />
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  <img src="assets/donate-alipay.jpg" width="260" alt="支付宝收款码" />
</p>

<p align="center"><sub>微信支付（左） · 支付宝（右）</sub></p>

> [!WARNING]
> 收款码仅用于向作者自愿赞赏。任何第三方以代下载、代部署、内部资源、会员群或官方授权为由收费，均与作者无关。

---

<a id="feedback"></a>
## 💬 反馈与贡献：Star History

- [Issues](https://github.com/L0NE-6/ChatGLM-Daily/issues)
- [Pull Requests](https://github.com/L0NE-6/ChatGLM-Daily/pulls)
- [Releases](https://github.com/L0NE-6/ChatGLM-Daily/releases)

<a href="https://star-history.com/#L0NE-6/ChatGLM-Daily&Date">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=L0NE-6/ChatGLM-Daily&type=Date&theme=dark" />
    <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=L0NE-6/ChatGLM-Daily&type=Date&theme=light" />
    <img alt="ChatGLM Daily Star History" src="https://api.star-history.com/svg?repos=L0NE-6/ChatGLM-Daily&type=Date&theme=light" />
  </picture>
</a>

<div align="center">

[⭐ Star](https://github.com/L0NE-6/ChatGLM-Daily) · [🍴 Fork](https://github.com/L0NE-6/ChatGLM-Daily/fork) · L0NE-6 · DAILY SIGNAL

</div>

# dsh-launcher

> DeepSeek Harness 托盘启动器 · A system-tray launcher for [DeepSeek Harness](https://www.npmjs.com/package/@deepseek-ai/dsh) (DSH)

把 DSH 的日常使用变成一个托盘小工具：双击启动，自动拉起服务、自动打开界面、插件变动自动重启、常驻系统托盘。

A tiny Windows tray app that turns DSH into a one-click tool: starts the service hidden, opens the UI, auto-restarts when plugins change, and lives in the system tray.

![Windows](https://img.shields.io/badge/Windows-10%2F11-blue) ![C#](https://img.shields.io/badge/.NET-Framework%204.x-512BD4) ![build](https://img.shields.io/github/actions/workflow/status/BoHe-dev/dsh-launcher/build.yml?label=build)

---

## 功能 Features

- **一键启动**：双击 exe → 隐藏启动 `npx @deepseek-ai/dsh web --no-open` → 等就绪 → 打开 DSH 界面 → 常驻托盘
  One-click start: hidden server launch, wait until ready, open the UI, stay in tray.
- **自动登录 DSH 网页（0.1.5+）**：读取 `dsh web` 打印的带 token 网址，换取会话 Cookie；界面直接打开在已登录状态，`/api` 轮询也能用
  Handles DSH 0.1.5+'s web authentication: reads the token URL `dsh web` prints, exchanges it for a session cookie, opens the UI already signed in, and uses the same session for its `/api` polls.
- **插件变动自动重启**：监控 DSH 插件目录（`~/.dsh/profiles/web`），检测到插件安装/卸载/启停后，**等聊天工作全部结束**再自动重启服务（重启会关闭并重开界面，插件才生效）
  Auto-restart on plugin changes: watches the plugin profile, waits until all agent work finishes, then restarts the service (web UIs are closed and reopened so plugins load).
- **防写一半重启**：连续 3 秒没有新的插件变动才确认（每次新变动都会刷新计时）
  3-second quiet window before confirming a change (new events refresh the timer).
- **托盘菜单**：打开界面 / 重启服务 / 关于 / 退出；菜单跟随系统深浅色模式
  Tray menu: open UI / restart / about / exit; follows Windows light/dark mode.
- **单实例**：重复双击只会重新打开界面
  Single instance: launching again just re-opens the UI.
- **中/英文界面**：`config.json` 里 `language` 切换
  Chinese/English UI, switch via `language` in `config.json`.

## 下载 Download

从 [Releases](https://github.com/BoHe-dev/dsh-launcher/releases) 下载 `dsh-launcher.exe`（Windows 10/11，无需安装 .NET——系统自带 .NET Framework 4.x）。

Download `dsh-launcher.exe` from Releases. No .NET SDK needed; Windows 10/11 ships the required .NET Framework 4.x.

## 使用 Usage

1. 把 `dsh-launcher.exe` 放到任意文件夹（例如 `D:\tools\dsh-launcher\`）或直接放桌面
   Put `dsh-launcher.exe` anywhere (e.g. `D:\tools\dsh-launcher\`) or on the Desktop.
2. 双击 `dsh-launcher.exe`。**不会在 exe 旁边生成任何文件**——配置自动写到 `%APPDATA%\dsh-launcher\config.json`（可随时编辑）
   Double-click the exe. **No files are created next to it** - config is auto-created at `%APPDATA%\dsh-launcher\config.json`.
3. 右键托盘图标可：打开界面、手动重启服务、退出（退出会停止服务）
   Right-click the tray icon: open UI, restart service, or exit (exit stops the service).
4. 日志在 `%TEMP%\dsh-launcher.log`，出问题先看它
   Logs go to `%TEMP%\dsh-launcher.log` - check it first when something is off.

> **便携模式 Portable mode**：把 `config.json` 放在 exe **同目录**，程序会优先读它（适合整个文件夹拷走用）；否则用 `%APPDATA%\dsh-launcher\config.json`（首次运行自动生成）。
> If a `config.json` sits next to the exe it wins (portable setup); otherwise the AppData one is used (auto-created on first run).

> **登录缓存 Session cache**：程序会把自己的 DSH 网页会话 Cookie 存在 `config.json` 旁边的 `browser-session.json`（同样是 `%APPDATA%\dsh-launcher\`）。它让"服务已在运行、由别的窗口启动"时也能用——那种情况下拿不到进程 token。删掉它只会多一次 token 交换。
> The launcher caches its own DSH web session cookie in `browser-session.json` beside `config.json` (i.e. also in `%APPDATA%\dsh-launcher\`). It is what keeps the `/api` polls working when the launcher attaches to a server it did not start, where the per-process token is unreachable. Deleting the file costs one token exchange.

> 前提：已安装 [Node.js](https://nodejs.org)（用于 `npx`），且能通过 `npx @deepseek-ai/dsh web` 启动 DSH。
> Prerequisite: [Node.js](https://nodejs.org) installed (for `npx`), DSH startable via `npx @deepseek-ai/dsh web`.

## 配置 config.json

| 字段 | 默认值 | 说明 |
|---|---|---|
| `port` | `3080` | DSH 服务端口 |
| `serverCommand` | `npx @deepseek-ai/dsh web --no-open` | 启动服务的命令（`--no-open` 让服务不要自己开浏览器，界面由本程序负责） |
| `workspace` | `""`（exe 所在目录） | 服务工作目录 |
| `profileDir` | `""`（`~/.dsh/profiles/web`） | 插件监控目录 |
| `chromeAppId` | `""`（自动发现） | DSH Chrome PWA 的 app-id；留空自动扫描 |
| `language` | `"zh"` | `"zh"` 或 `"en"` |
| `appName` | `"DeepSeek Harness"` | 托盘提示/气泡标题 |
| `openOnStart` | `true` | 启动后自动打开界面 |
| `autoRestart` | `true` | 插件变动自动重启总开关 |
| `closeWebUisOnRestart` | `true` | 重启时关闭并重开界面窗口 |
| `watchdog` | `true` | 服务意外退出时自动重启 |
| `changeQuietSeconds` | `3` | 变动确认前的静默秒数 |
| `idlePollSeconds` | `3` | 会话空闲轮询间隔（秒） |
| `idleConfirmCount` | `2` | 连续几次空闲才重启 |
| `stopTimeoutSeconds` | `15` | 等待旧服务停止上限 |
| `startTimeoutSeconds` | `180` | 等待新服务启动上限 |
| `logFile` | `""`（`%TEMP%\dsh-launcher.log`） | 日志路径 |

## 从源码构建 Build from source

```powershell
git clone https://github.com/BoHe-dev/dsh-launcher.git
cd dsh-launcher
.\build.ps1          # 输出 dist\dsh-launcher.exe
```

无需任何 SDK：构建脚本直接调用系统自带的 .NET Framework 编译器（csc.exe）。
No SDK needed: the script uses the csc.exe that ships with Windows.

GitHub Actions 会在打 `v*` 标签时自动构建并发布 Release。

## 测试环境变量 Test-only environment variables

| 变量 | 作用 |
|---|---|
| `DSH_MUTEX_SUFFIX` | 单实例标识后缀（用于并行跑多个实例测试） |
| `DSH_WATCH_DIR` | 覆盖插件监控目录 |
| `DSH_NO_RESTART=1` | 只记录日志、不真正重启（安全测试） |

## 工作原理 How it works

1. **启动服务**：隐藏窗口运行 `serverCommand`，同时读它的 stdout 抓取登录网址，轮询端口直到就绪
   Starts the server hidden, reads the URL line off its stdout, and polls the port until ready.
2. **登录**：`dsh web` 打印的 `?token=` 只属于那一个进程，所以程序把它换成 `dsh-auth-*` 会话 Cookie 供自己的 `/api` 调用；界面则直接开到那个带 token 的网址上，由浏览器自己完成同样的交换再跳回干净网址
   Authentication: the `?token=` printed by `dsh web` belongs to that one process, so the launcher exchanges it once for a `dsh-auth-*` session cookie for its own `/api` calls, and opens the UI at the token URL so the browser performs the same exchange and lands on the clean URL.
3. **空闲检测**：带会话 Cookie `POST /api/session/list`，看每个会话的 `running`
   Idle detection via `POST /api/session/list` with that session cookie, checking every session's `running` flag.
4. **插件监控**：只认清单文件（`package.json` / `pnpm-lock.yaml` / `pnpm-workspace.yaml` / `cordis.patch.yml`）和 `node_modules` 下新增/删除的包目录；插件包内部写入（缓存、运行时数据）一律忽略
   Plugin watch: only manifest files and new/removed packages under `node_modules` count; runtime writes inside packages are ignored.
5. **界面打开**：用 Chrome 应用窗口打开带 token 的网址（自带登录）；Chrome 不在时回退 PWA 快捷方式 / 默认浏览器
   UI open: a Chrome app window at the authenticated URL (which authenticates itself); the PWA shortcut and the default browser are fallbacks.

## 兼容性 Compatibility

1.0.3 起适配 **DSH 0.1.5+**。该版本给 Web 界面加了鉴权，同时对 RPC 做了重命名，1.0.2 的三处假设全部失效：

Since 1.0.3 the launcher targets **DSH 0.1.5+**. That release put the web surface behind authentication and renamed the RPC transport, which broke three assumptions 1.0.2 made:

| 变化 Change | 1.0.2 的做法 | 1.0.3 的做法 |
|---|---|---|
| `/` 与 `/api/*` 无 Cookie 时返回 **401** | 把 401 当成"服务没起来" | 任何 HTTP 应答都算"服务已就绪" |
| 端点改为规范式 `<命名空间>/<方法>` | `POST /api/session.list` | `POST /api/session/list` |
| Remote 负载只收一个 `args` 字段 | `"payload":{}` | `"payload":{"args":{"_request":{}}}` |
| `dsh web` 默认自己打开浏览器 | —— | 默认命令加 `--no-open`，界面由本程序负责 |

1.0.2 的症状：服务明明在跑，托盘却认为没起来（于是再拉起一个服务、界面永远打不开、看门狗不生效、重启必定"启动超时"）；同时空闲检测把 401 读成"没有会话在跑"，会在聊天进行中重启。

Symptoms under 1.0.2: a running server read as down (so the launcher started a second one, never opened the UI, never armed the watchdog, and every restart ended in "startup timeout"), while idle detection read the 401 as "no session running" and could restart in the middle of a chat.

## 注意 / Notes

- 重启时会强制结束占用 `port` 的进程（`taskkill /T /F`），请勿在服务运行关键任务时手动重启
  Restart force-kills the process listening on `port`; avoid manual restarts while critical work is running.
- 重启会把 DSH 的网页窗口关掉并重新打开（插件需要全新加载）
  Restart closes and reopens the DSH web windows (plugins need a fresh page load).
- 界面用 Chrome 应用窗口（`--app=<带 token 的网址>`）打开，因为已安装的 PWA 快捷方式（`--app-id`）没法带上 token；想要 PWA 图标/窗口的话，把 `chromeAppId` 配上，它仍用于托盘图标和无 token 时的回退
  The UI opens in a Chrome app window (`--app=<authenticated URL>`), because an installed PWA (`--app-id`) cannot carry the token. `chromeAppId` still drives the tray icon and the no-token fallback.
- 拿不到 token（例如服务是先由别的窗口启动的）时，`/api` 判不出空闲，自动重启会保守地暂停，不会再冒险打断正在进行的聊天
  With no token available (a server started elsewhere), `/api` cannot report an idle state; auto-restart then stays conservative instead of risking an interruption.

## License

[MIT](LICENSE)

---
name: antshell
description: AntShell 本地 MCP 接口使用手册 — 帮 AI 客户端通过本地 HTTP 驱动 AntShell 的 SSH / FTP / 本地终端 / 文件操作 / 软件设置。在以下场景必须调用本 skill:(1) 用户提到 AntShell、SSH/FTP 终端自动化、本机跑命令、本地操作远端服务器;(2) 用户希望 AI 替自己操作 AntShell 已保存的连接、打开新终端会话(SSH/FTP/本地)、读终端输出、发命令、读写远端文件、上传下载、修改软件设置(界面/侧边AI/本地工具链);(3) 用户希望与 AntShell 软件界面同步看到 AI 在做什么(打开哪个 session、执行什么命令、文件怎么改、设置怎么改);(4) 用户提供 AntShell 的 HTTP 地址(默认 http://127.0.0.1:4180)希望 AI 通过它做事。即使用户没明说"用 MCP",只要涉及 AntShell 的能力调用,就先用本 skill 查协议。
---

# AntShell Skill (v5)

AntShell 是一个跨平台(macOS / Windows / Linux)的 SSH/FTP/本地终端桌面工作台(Electron + Vue 3),内置一个**本地 HTTP MCP 接口**供外部 AI 工具驱动。本 skill 是这个接口的完整使用手册——AI 客户端通过 curl 调对应端点即可。

**核心事实**:
- AntShell 当前必须**已经在用户本机运行**且 **MCP 开关为已开启**。如果没开,引导用户去 AntShell → 设置 → MCP → 开启。
- 接口**仅监听 127.0.0.1**,不会对外暴露,不需要鉴权。
- 端口用户可配置(默认 4180),从 `GET /v1/health` 的 `data.port` 读实际值,不要硬编码。
- 端点全部使用统一响应壳 `{ ok: true, data: ... }` / `{ ok: false, error: { code, message } }`,HTTP 状态码 200/400/404/409/413/500/402。
- 操作 AntShell 的 session 后,AntShell 软件界面会同步激活对应会话。SSH 命令及输出显示在 xterm；SFTP/FTP 文件操作显示在文件区和传输中心。不要伪造终端命令来表现文件操作。

---

## 第一步:健康检查 + 端口探测

操作前先调一次:

```bash
curl -s http://127.0.0.1:4180/v1/health
```

期望:
```json
{"ok":true,"data":{"running":true,"port":4180,"pid":...}}
```

- `ok=false` 或 `running=false` → 提示用户去 AntShell → 设置 → MCP → 开启
- `data.port` 是**实际监听端口**,后续所有 URL 用这个端口拼(用户可能改过默认 4180)
- 如果用户提供的 URL 不是 4180,使用用户给的 URL 端口

---

## 第二步:36 个端点(单一来源)

接口分 5 组,以下 URL 都用 `${base}` 代替 `http://127.0.0.1:${port}`。

### Health (1)
| Method | Path | 用途 |
|---|---|---|
| GET | `${base}/v1/health` | 健康检查 |

### Connections (4)
| Method | Path | 用途 | Body |
|---|---|---|---|
| GET    | `${base}/v1/connections` | 列出所有连接 | — |
| POST   | `${base}/v1/connections` | 新增连接 | `{name, type:'ssh'\|'ftp', host, port, username, password, ...}` |
| PUT    | `${base}/v1/connections/{id}` | 修改连接 | 同上(覆盖写) |
| DELETE | `${base}/v1/connections/{id}` | 删除连接 | — |

### Sessions (8)
| Method | Path | 用途 | Body |
|---|---|---|---|
| POST | `${base}/v1/sessions` | 打开会话 | `{connectionId}` (SSH/FTP) **或** `{options:{command,args,cwd}}` (本地) |
| GET  | `${base}/v1/sessions` | 列出活跃会话 | — |
| GET  | `${base}/v1/sessions/{id}` | 会话详情 | — |
| GET  | `${base}/v1/sessions/{id}/output?since=N` | 增量读输出(256KB ring buffer) | — |
| POST | `${base}/v1/sessions/{id}/input` | 写输入字节 | `{data: "<base64>"}` |
| POST | `${base}/v1/sessions/{id}/exec` | 执行命令(SSH 非零退出码报错 / FTP raw) | `{command, timeout}` |
| POST | `${base}/v1/sessions/{id}/reconnect` | 原位重连 SSH/FTP，会话 ID 保持不变 | — |
| POST | `${base}/v1/sessions/{id}/close` | 关闭会话 | — |

### Files (17)
| Method | Path | 用途 | Body |
|---|---|---|---|
| GET  | `${base}/v1/sessions/{id}/files?path=/etc` | 列目录 | — |
| POST | `${base}/v1/sessions/{id}/files/read`    | 读文本 | `{path}` |
| POST | `${base}/v1/sessions/{id}/files/write`   | 写文本 | `{path, content}` |
| POST | `${base}/v1/sessions/{id}/files/mkdir`   | 建目录 | `{path}` |
| POST | `${base}/v1/sessions/{id}/files/remove`  | 删除   | `{path}` |
| POST | `${base}/v1/sessions/{id}/files/rename`  | 重命名 | `{from, to}` |
| POST | `${base}/v1/sessions/{id}/files/chmod`   | 改权限 | `{path, mode}` |
| POST | `${base}/v1/sessions/{id}/files/upload`  | 上传小文件并进入传输中心 | `{path, name, contentBase64, options?}` |
| POST | `${base}/v1/sessions/{id}/files/download` | 下载到本机临时目录并进入传输中心 | `{path}` |
| POST | `${base}/v1/sessions/{id}/files/upload-paths` | 从本机路径批量/递归上传 | `{remoteDir, localPaths, options?}` |
| POST | `${base}/v1/sessions/{id}/files/download-many` | 批量/递归下载到指定本机目录 | `{paths, localDir, options?}` |
| POST | `${base}/v1/sessions/{id}/files/remove-many` | 批量递归删除 | `{paths}` |
| POST | `${base}/v1/sessions/{id}/files/move` | 批量移动 | `{paths, targetDir}` |
| POST | `${base}/v1/sessions/{id}/files/chmod-many` | 批量改权限 | `{paths, mode}` |
| POST | `${base}/v1/sessions/{id}/files/quick-remove` | SSH rm -rf 快速批量删除 | `{paths}` |
| POST | `${base}/v1/sessions/{id}/files/compress` | SSH 远程批量压缩 | `{paths, options}` |
| POST | `${base}/v1/sessions/{id}/files/extract` | SSH 远程解压 | `{path, options}` |

FTP 路径上传的 `options.conflictPolicy` 支持 `ask`、`cancel`、`coexist`、`overwrite`，默认 `ask`；`options.maxRetries` 为 0-10，默认 3。base64 请求体上限 32MB，大文件和文件夹必须使用 `upload-paths`。

### Transfers (4)
| Method | Path | 用途 |
|---|---|---|
| GET | `${base}/v1/transfers` | 列出传输、自动重试和递归子项状态 |
| POST | `${base}/v1/transfers/{id}/cancel` | 取消单项传输 |
| POST | `${base}/v1/transfers/cancel-all` | 终止全部传输及待执行批次 |
| POST | `${base}/v1/transfers/clear` | 清理已完成、失败和取消记录 |

### Settings (2)
| Method | Path | 用途 | Body |
|---|---|---|---|
| GET | `${base}/v1/settings` | 读取当前设置(界面/侧边AI/本地工具链)。`apiKey` 响应中脱敏为 `sk-***xxxx` 形式 | — |
| PUT | `${base}/v1/settings` | **部分更新**设置,写盘、设置页、实际运行效果三方实时同步 | 见下方 |

**PUT /v1/settings body**(三段均为可选,均未传字段保持当前值;`tools` 数组为整段替换):

```jsonc
{
  "ui": {
    "topNavAlign": "left" | "center" | "right",  // 顶部选项位置
    "darkMode": "system" | "off" | "on",          // 暗夜模式(改后主题即时生效)
    "preventSleep": true | false                   // 阻止屏幕息屏(不影响系统休眠)
  },
  "ai": {
    "responseFormat": "openai-chat" | "openai-responses" | "anthropic",
    "baseUrl": "https://api.example.com/v1",
    "apiKey": "sk-xxx",
    "model": "gpt-4o",
    "toolCalling": true | false,                   // Agent 模式开关
    "streaming": true | false,
    "mockMode": true | false,                      // 离线假数据测试
    "systemPrompt": ""                             // 追加到内置系统提示之后
  },
  "tools": [
    { "id": "可选,有则更新无则新增", "name": "Claude", "commandPath": "claude", "args": "" }
  ]
}
```

**注意**:
- `mcpEnabled` / `mcpPort` **不允许**通过此接口修改,请使用 AntShell 的 MCP 开关(设置 → MCP)。误传会返回 `400 bad_request`。
- 未激活用户若误传 `mcpEnabled:true`,返回 `402 license_required`,提示用户先激活。
- `PUT` 是**部分合并**:只需传要改的段和字段,未传字段保留当前值(但 `tools` 数组整段替换,不传则保留)。
- 修改后 AntShell 设置页、配置文件 (`settings.json`)、实际行为(主题/息屏/AI/工具链下拉)**同步即时生效**,无需重启。

**示例**:
```bash
# 1) 读取当前配置
curl -s ${base}/v1/settings
# 2) 切换暗夜模式
curl -s -X PUT ${base}/v1/settings -H 'Content-Type: application/json' -d '{"ui":{"darkMode":"on"}}'
# 3) 更换 AI 模型
curl -s -X PUT ${base}/v1/settings -H 'Content-Type: application/json' -d '{"ai":{"model":"gpt-4o"}}'
# 4) 新增一条本地工具链
curl -s -X PUT ${base}/v1/settings -H 'Content-Type: application/json' -d '{"tools":[{"name":"Claude","commandPath":"claude","args":""}]}'
```

---

## 第三步:典型工作流

执行会话操作前先 `GET /v1/sessions` 获取真实 sessionId，不要根据连接名或历史响应猜测。遇到网络波动、`shell closed` 或暂时断线时，优先调用原位重连；只有重连明确失败且用户仍需继续时才关闭并新建会话。

### 1. SSH/FTP 工作流
```bash
# 1) 列连接
curl -s http://127.0.0.1:4180/v1/connections
# 2) 打开会话(拿 sessionId)
curl -s -X POST http://127.0.0.1:4180/v1/sessions -H 'Content-Type: application/json' -d '{"connectionId":"<id>"}'
# 3) 读输出(buffer 累计,since 用上次 totalLen)
curl -s 'http://127.0.0.1:4180/v1/sessions/<sid>/output?since=0'
# 4) 一次性命令(SSH 非零退出码返回错误,FTP 走 raw)
curl -s -X POST http://127.0.0.1:4180/v1/sessions/<sid>/exec -H 'Content-Type: application/json' -d '{"command":"whoami","timeout":5000}'
# 5) 临时断线时原位重连，成功后继续使用同一个 sid
curl -s -X POST http://127.0.0.1:4180/v1/sessions/<sid>/reconnect
# 6) 关闭
curl -s -X POST http://127.0.0.1:4180/v1/sessions/<sid>/close
```

### 2. 本地终端工作流
```bash
# 1) 打开本地终端(不传 connectionId,传 options)
curl -s -X POST http://127.0.0.1:4180/v1/sessions -H 'Content-Type: application/json' -d '{"options":{"command":"/bin/zsh","args":["-l"]}}'
# 2) 写输入(SSH/本地都支持 /input)
DATA=$(printf 'pwd\nls /\n' | base64)
curl -s -X POST http://127.0.0.1:4180/v1/sessions/<sid>/input -H 'Content-Type: application/json' -d "{\"data\":\"$DATA\"}"
# 3) 读输出
curl -s 'http://127.0.0.1:4180/v1/sessions/<sid>/output?since=0'
# 4) 关闭
curl -s -X POST http://127.0.0.1:4180/v1/sessions/<sid>/close
```

### 3. 文件操作工作流(基于已打开的 session)
```bash
# 所有文件操作都基于一个 open sessionId
# 写操作前先 GET 当前状态以避免覆盖
curl -s -X POST http://127.0.0.1:4180/v1/sessions/<sid>/files/write -H 'Content-Type: application/json' -d '{"path":"/tmp/a.txt","content":"hello"}'
curl -s -X POST http://127.0.0.1:4180/v1/sessions/<sid>/files/read  -H 'Content-Type: application/json' -d '{"path":"/tmp/a.txt"}'
curl -s -X POST http://127.0.0.1:4180/v1/sessions/<sid>/files/remove -H 'Content-Type: application/json' -d '{"path":"/tmp/a.txt"}'
# 大文件或文件夹从本机路径递归上传
curl -s -X POST http://127.0.0.1:4180/v1/sessions/<sid>/files/upload-paths -H 'Content-Type: application/json' -d '{"remoteDir":"/tmp","localPaths":["/Users/name/folder"],"options":{"conflictPolicy":"ask","maxRetries":3}}'
# 批量下载到本机目录
curl -s -X POST http://127.0.0.1:4180/v1/sessions/<sid>/files/download-many -H 'Content-Type: application/json' -d '{"paths":["/etc/hosts","/var/log"],"localDir":"/Users/name/Downloads"}'
```

---

## 与 AntShell 软件界面的同步

每次 `POST /v1/sessions` 成功,AntShell **自动**:
- SSH/FTP: 跳到「工作台」视图,激活该 session,xterm 接管显示
- local: 跳到「本地终端」视图,激活该终端

每次 `POST /v1/sessions/{id}/exec`(SSH) 成功:
- 真实命令以终端提示行形式显示，stdout/stderr 随后显示；不会添加多余的 `[MCP]` 前缀

每次 `POST /v1/sessions/{id}/reconnect` 成功:
- 原位重连 SSH/FTP 标签页并保留原 sessionId，后续请求继续使用该 ID
- 重连失败时旧 SSH 会话不会被成功重连结果错误替换；应向用户报告接口返回的具体原因

每次文件和传输操作:
- 目录变化刷新文件区；上传、下载、预览拉取和文本保存同步显示在传输中心
- 文件操作不伪装成 shell 命令，也不向 xterm 写入 `[MCP]`、`[FTP]` 等噪声标签

每次 `POST /v1/sessions/{id}/close` 触发:
- AntShell 自动从 UI 移除该 session 标签、关闭 xterm、清空文件列表、回到「连接」视图

**所以 AI 一边操作,用户一边看着 xterm 实时同步——不要做"悄悄"操作**。

---

## 错误处理

- `400 bad_request` — 参数缺失或格式错误(消息含「不能为空」「必须是 1-65535」「缺少 path 字段」「mcpEnabled/mcpPort 不允许通过 settings 接口修改」)
- `402 license_required` — 未激活用户试图开启 MCP(`mcpEnabled:true`),提示先激活
- `404 not_found` — 资源不存在(连接/会话/路径/路由)
- `409 conflict` — 冲突(目前少用)
- `413 payload_too_large` — base64 请求体超过 32MB,改用本地路径上传
- `500 internal` — 服务器内部错误(网络/远端服务器返回错误/连接超时)

非 200 → 把 `error.message` 转给用户(不要只说"出错了")。

---

## 安装 Skill

如果用户**刚装好 AntShell**,AI 客户端还没装这个 skill:
1. 让用户打开 AntShell → 顶部 MCP tab(或设置 → MCP) → 点「安装 Skill」
2. AntShell 会从 `assets/antshell.zip` 抽取，并按用户选择安装到 Claude Code、Codex CLI 或项目 Skill 目录
3. 重启对应 AI 客户端后本 skill 自动可用
4. 也可让用户复制 MCP tab 里的「复制提示词」按钮,把完整协议交给 AI
5. 内置安装与兜底均失败时，从 https://github.com/Mutantcat-Working-Group/AntShell-Skill 下载

如果接口停止,提示用户重新启用(MCP 开关)。

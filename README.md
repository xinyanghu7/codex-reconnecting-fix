# Codex 一直 Reconnecting 1/5？复制一段提示词，让 Agent 帮你修好

**简体中文** | [English](README.en.md)

> 在线一键复制版：**[suzhe.ai/gptfix](https://suzhe.ai/gptfix?utm_source=github&utm_medium=readme&utm_campaign=gptfix)**（中英双语，点一下就复制好）
>
> 由 [苏哲AI · suzhe.ai](https://suzhe.ai/?utm_source=github&utm_medium=readme&utm_campaign=gptfix) 整理

Codex 桌面版（Codex Desktop）每次发消息都卡在 `Reconnecting… 1/5` 到 `5/5`？通常是 Codex 没走你本机的代理。

下面这段提示词会让你的 Agent 自己找到电脑上**真正在用的代理端口**，写进 Codex 的配置，不用你懂任何技术。

## 傻瓜式 3 步

1. **复制**：点下面代码块右上角的复制按钮（或打开 [suzhe.ai/gptfix](https://suzhe.ai/gptfix?utm_source=github&utm_medium=readme&utm_campaign=gptfix) 点蓝色按钮）。
2. **发给 Agent**：粘贴给能在你电脑上执行命令的 Agent（Codex、Claude Code、Cursor 都行），发送。保持代理软件开着，等它做完。
3. **重启 Codex**：按它最后的提示彻底退出 Codex 再打开。

> 提示 `Selected model is at capacity. Please try a different model.` 是模型繁忙，不是这个问题，换个模型或稍后再试即可。

## 提示词

```text
请直接在我这台电脑上操作，修复 Codex 桌面版（Codex Desktop）每次发消息都卡在「Reconnecting… 1/5」到「5/5」的问题。做完后用中文告诉我结果。

## 0. 先确认是不是这个问题
- 本方法只解决 Reconnecting… 1/5～5/5（连接反复断开重连）。
- 如果我看到的提示是「Selected model is at capacity. Please try a different model.」，那是模型容量问题，与本机网络无关：停下来告诉我换个模型或稍后再试，不要改任何代理配置。
- 如果本机没有运行任何代理软件，也停下来告诉我，不要写配置。

## 1. 只读检查本机代理（这一步不改任何东西）
目标：找出正在运行的代理软件（如 Clash Verge、Clash for Windows、mihomo、v2rayN、NekoRay、sing-box、ClashX、Surge 等），以及它在本机真正监听的 HTTP 或 Mixed 端口。

规则：
1. 不要猜 7890、10808、1080 这类常见端口，一律以本机实际监听为准。
2. SOCKS 端口不能写进 HTTP_PROXY / HTTPS_PROXY。软件同时提供 HTTP 和 SOCKS 端口时只用 HTTP；标为 Mixed（混合）的端口可以当 HTTP 用。
3. 交叉核对端口：系统代理设置、代理进程实际监听的 127.0.0.1 端口、代理软件配置里的 port / mixed-port / http 入站，三者对得上再用。
   - Windows 参考：reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings" 查看 ProxyEnable / ProxyServer；netstat -ano -p tcp | findstr LISTENING 配合 tasklist /FI "PID eq <PID>" 对上进程。
   - macOS 参考：scutil --proxy；lsof -nP -iTCP -sTCP:LISTEN。
4. 确认是 HTTP 端口：curl -s -m 5 -o /dev/null -w "%{http_code}" http://127.0.0.1:<端口>/
   几秒内返回任意 HTTP 状态码（包括 400、404）才算；超时、空回复、连接被拒都不算。Windows 上用 curl.exe。
5. 确认能出网：curl -s -m 10 -o /dev/null -w "%{http_code}" -x http://127.0.0.1:<端口> https://api.openai.com/v1/models
   返回 401 之类的状态码就说明代理通了；返回 000 说明这个端口不行，换下一个候选端口，不要硬写。
6. 读代理配置时，回复里不要出现服务器地址、密码、UUID、令牌、订阅链接或任何密钥，只报软件名和本机端口。

## 2. 写入 Codex 的 .env
路径：
- Windows：%USERPROFILE%\.codex\.env（即 C:\Users\<当前用户名>\.codex\.env）
- macOS / Linux：~/.codex/.env

要求：
- 目录或文件不存在就创建。
- 文件已存在：先在同目录备份一份（如 .env.bak-<日期>），保留原有全部内容，只新增或更新下面三个变量；同名变量已存在就改值，不要重复写，不要删除其他配置。
- 写入内容（两处端口必须是第 1 步验证通过的同一个端口）：

HTTP_PROXY="http://127.0.0.1:<端口>"
HTTPS_PROXY="http://127.0.0.1:<端口>"
NO_PROXY="localhost,127.0.0.1,::1"

保存后检查：
- 完整路径正确；文件名就是 .env，不能是 .env.txt（Windows 用 Get-ChildItem -Force "$env:USERPROFILE\.codex" 看真实文件名）。
- 编码是 UTF-8 无 BOM。Windows PowerShell 5 的 Set-Content / Out-File 可能写成带 BOM 或 UTF-16，建议用 [IO.File]::WriteAllText(路径, 内容, (New-Object Text.UTF8Encoding $false)) 写入。
- 读回文件，确认三个变量各只有一行，两处端口一致。

## 3. 不要结束正在运行的 Codex
你可能就运行在 Codex 里，不要自己结束任何 Codex 进程。完成后告诉我这样重启：
- Windows：在右下角托盘找到 Codex 图标，右键选「退出 / Quit」。托盘里没有的话，打开任务管理器，结束 codex.exe、codex-code-mode-host.exe、codex-computer-use-swift.exe、codex-windows-sandbox-service.exe（有哪个结束哪个），然后重新打开 Codex。
- macOS：在 Codex 窗口按 Command + Q 完全退出，再重新打开。

## 4. 如果 .env 不生效
如果你能确认当前 Codex 版本不读取 .codex/.env，或者我重启后告诉你仍然 Reconnecting：不要反复覆盖 .env，改为给当前用户设置同样的三个环境变量（端口同上）。
- Windows（用户级，不需要管理员）：[Environment]::SetEnvironmentVariable("HTTP_PROXY","http://127.0.0.1:<端口>","User")，HTTPS_PROXY、NO_PROXY 同样设置。
- macOS：launchctl setenv HTTP_PROXY "http://127.0.0.1:<端口>"，HTTPS_PROXY、NO_PROXY 同样设置（重启电脑后会失效，需要再执行一次）。
设置后提醒我：先注销或重启电脑，再打开 Codex。也提醒我：以后关掉代理软件或换了端口，要同步修改这些变量，否则其他联网程序也可能连不上。

## 5. 最后按这个格式汇报
- 现象判断：是否属于 Reconnecting 问题
- 代理软件：名称（不写节点、服务器等信息）
- 使用端口：端口号（HTTP 还是 Mixed）和两次 curl 的验证结果
- 已修改：文件完整路径；新建还是更新；备份文件名
- 下一步：我要怎样彻底退出并重开 Codex
```

## 常见问题

**会不会泄露我的代理节点？**
提示词要求 Agent 只报代理软件名和本机端口，不输出服务器地址、密码、订阅链接。所有操作都在你自己的电脑上完成。

**会不会删掉我原来的配置？**
不会。已有的 `.env` 会先备份，只新增或更新三个代理变量，其他内容原样保留。

**重启后还是 Reconnecting？**
对同一个 Agent 说「重启后仍然 Reconnecting」，它会按提示词第 4 步改用系统环境变量。

**没开代理软件能用吗？**
不能。这个方法是让 Codex 走你本机已经在运行的代理，先打开代理软件。

## 原理

Codex 桌面版启动时会读取 `~/.codex/.env`（Windows：`%USERPROFILE%\.codex\.env`）。把 `HTTP_PROXY` / `HTTPS_PROXY` 指向本机代理的 HTTP 或 Mixed 端口后，Codex 的连接就会走代理，不再反复断线重连。最常见的坑是：写了猜的端口（7890、10808）、把 SOCKS 端口当 HTTP 用、或文件被存成了 `.env.txt` / 带 BOM。提示词逐条规避了这些问题。

---

觉得有用请点个 ⭐ Star，方便更多人找到。

更多 ChatGPT / Codex 使用技巧和 GPT 会员服务：**[苏哲AI · suzhe.ai](https://suzhe.ai/?utm_source=github&utm_medium=readme&utm_campaign=gptfix)**

[MIT License](LICENSE)

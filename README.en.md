# Codex stuck on "Reconnecting 1/5"? Copy one prompt and let your agent fix it

[简体中文](README.md) | **English**

> One-click copy version: **[suzhe.ai/gptfix](https://suzhe.ai/gptfix?utm_source=github&utm_medium=readme&utm_campaign=gptfix&lang=en)**
>
> Maintained by [Suzhe AI · suzhe.ai](https://suzhe.ai/?utm_source=github&utm_medium=readme&utm_campaign=gptfix)

Does Codex Desktop get stuck on `Reconnecting… 1/5` to `5/5` every time you send a message? Usually Codex is not going through your local proxy.

The prompt below makes your agent find the proxy port **actually in use** on your computer and write it into Codex's config. No technical knowledge needed.

## 3 simple steps

1. **Copy**: click the copy button at the top right of the code block below (or open [suzhe.ai/gptfix](https://suzhe.ai/gptfix?utm_source=github&utm_medium=readme&utm_campaign=gptfix&lang=en) and click the blue button).
2. **Send it to your agent**: paste it into an agent that can run commands on your computer (Codex, Claude Code, Cursor, etc.) and send. Keep your proxy app running and let it finish.
3. **Restart Codex**: fully quit Codex and reopen it, as the agent tells you.

> Seeing `Selected model is at capacity. Please try a different model.`? That means the model is busy, not this issue. Switch models or try again later.

## The prompt

```text
Please work directly on this computer and fix Codex Desktop getting stuck on "Reconnecting… 1/5" through "5/5" every time I send a message. When you are done, tell me the result in English.

## 0. Confirm this is the right problem
- This fix only covers Reconnecting… 1/5 to 5/5 (the connection keeps dropping and retrying).
- If the message I see is "Selected model is at capacity. Please try a different model.", that is a model capacity issue and has nothing to do with my network: stop, tell me to switch models or try later, and do not change any proxy settings.
- If no proxy app is running on this computer, also stop and tell me. Do not write any config.

## 1. Inspect the local proxy (read-only, change nothing in this step)
Goal: identify the running proxy app (e.g. Clash Verge, Clash for Windows, mihomo, v2rayN, NekoRay, sing-box, ClashX, Surge) and the HTTP or Mixed port it is actually listening on locally.

Rules:
1. Do not guess common ports like 7890, 10808 or 1080. Only use a port that is really listening on this machine.
2. Never put a SOCKS port into HTTP_PROXY / HTTPS_PROXY. If the app offers both HTTP and SOCKS ports, use HTTP only; a port labeled Mixed can be used as HTTP.
3. Cross-check the port: the system proxy setting, the 127.0.0.1 port the proxy process is listening on, and the port / mixed-port / http inbound in the proxy app's config should agree.
   - Windows hints: reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings" for ProxyEnable / ProxyServer; netstat -ano -p tcp | findstr LISTENING together with tasklist /FI "PID eq <PID>" to match the process.
   - macOS hints: scutil --proxy; lsof -nP -iTCP -sTCP:LISTEN.
4. Confirm it is an HTTP port: curl -s -m 5 -o /dev/null -w "%{http_code}" http://127.0.0.1:<port>/
   It counts only if any HTTP status code (including 400 or 404) comes back within a few seconds; a timeout, empty reply or refused connection does not count. Use curl.exe on Windows.
5. Confirm it reaches the internet: curl -s -m 10 -o /dev/null -w "%{http_code}" -x http://127.0.0.1:<port> https://api.openai.com/v1/models
   A status like 401 means the proxy works; 000 means this port does not work, so try the next candidate instead of forcing it.
6. When reading proxy configs, never show server addresses, passwords, UUIDs, tokens, subscription links or any secrets in your replies. Only report the app name and the local port.

## 2. Write Codex's .env
Path:
- Windows: %USERPROFILE%\.codex\.env (i.e. C:\Users\<current user>\.codex\.env)
- macOS / Linux: ~/.codex/.env

Requirements:
- Create the folder or file if it does not exist.
- If the file exists: first back it up in the same folder (e.g. .env.bak-<date>), keep all existing content, and only add or update the three variables below. If a variable already exists, change its value instead of adding a duplicate. Do not delete anything else.
- Content to write (both ports must be the same port verified in step 1):

HTTP_PROXY="http://127.0.0.1:<port>"
HTTPS_PROXY="http://127.0.0.1:<port>"
NO_PROXY="localhost,127.0.0.1,::1"

After saving, check:
- The full path is correct and the file name is exactly .env, not .env.txt (on Windows, run Get-ChildItem -Force "$env:USERPROFILE\.codex" to see the real name).
- The encoding is UTF-8 without BOM. Windows PowerShell 5 Set-Content / Out-File may write a BOM or UTF-16; prefer [IO.File]::WriteAllText(path, content, (New-Object Text.UTF8Encoding $false)).
- Read the file back and confirm each variable appears exactly once and both ports match.

## 3. Do not kill the running Codex
You may be running inside Codex yourself, so do not end any Codex process. When finished, tell me how to restart it:
- Windows: find the Codex icon in the system tray (bottom right), right-click, choose Quit. If it is not in the tray, open Task Manager and end codex.exe, codex-code-mode-host.exe, codex-computer-use-swift.exe and codex-windows-sandbox-service.exe (whichever exist), then open Codex again.
- macOS: press Command + Q in the Codex window to quit completely, then open it again.

## 4. If .env has no effect
If you can confirm this Codex version does not read .codex/.env, or I tell you it still says Reconnecting after a restart: do not keep overwriting .env. Instead, set the same three environment variables for the current user (same port).
- Windows (user level, no admin needed): [Environment]::SetEnvironmentVariable("HTTP_PROXY","http://127.0.0.1:<port>","User"), and the same for HTTPS_PROXY and NO_PROXY.
- macOS: launchctl setenv HTTP_PROXY "http://127.0.0.1:<port>", and the same for HTTPS_PROXY and NO_PROXY (this resets after a reboot and must be run again).
Then remind me to sign out or restart the computer before opening Codex. Also remind me: if I later turn off the proxy app or change its port, I must update these variables, or other apps may lose their connection too.

## 5. Final report in this format
- Diagnosis: is this the Reconnecting problem
- Proxy app: name only (no nodes, servers or other details)
- Port used: port number (HTTP or Mixed) and the results of both curl checks
- Changes made: full file path; created or updated; backup file name
- Next step: how I fully quit and reopen Codex
```

## FAQ

**Will it leak my proxy nodes?**
The prompt tells the agent to report only the proxy app name and local port, never server addresses, passwords or subscription links. Everything happens on your own computer.

**Will it delete my existing config?**
No. An existing `.env` is backed up first; only the three proxy variables are added or updated, everything else stays.

**Still Reconnecting after restart?**
Tell the same agent "still Reconnecting after restart". It will switch to user environment variables (step 4 of the prompt).

**Does it work without a proxy app?**
No. This makes Codex use the proxy already running on your computer. Start your proxy app first.

## How it works

Codex Desktop reads `~/.codex/.env` (Windows: `%USERPROFILE%\.codex\.env`) on startup. Pointing `HTTP_PROXY` / `HTTPS_PROXY` at your local proxy's HTTP or Mixed port routes Codex through the proxy, so it stops dropping and reconnecting. The usual mistakes are guessed ports (7890, 10808), using a SOCKS port as HTTP, or a file saved as `.env.txt` / with a BOM. The prompt avoids each of these.

---

If this helped, please give it a ⭐ Star so more people can find it.

More ChatGPT / Codex tips and GPT subscription services: **[Suzhe AI · suzhe.ai](https://suzhe.ai/?utm_source=github&utm_medium=readme&utm_campaign=gptfix)**

[MIT License](LICENSE)

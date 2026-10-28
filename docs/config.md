# 設定檔

位置：`~/.xing-janus/config.json`。完整的格式在 [`config.schema.json`](config.schema.json)，`xj doctor` 會照它檢查。沒寫的欄位用預設值。
`xj setup` 會在設定檔開頭寫上 `"$schema"`：用 VS Code 這類編輯器打開，會自動補完、顯示每一欄的說明、打錯當場標出來。
詢問時選「永久允許」會寫進 `permissions.allow`、`sandbox.allowedDomains`、MCP server 的 `autoApprove`（只改那一欄，寫之前整份檢查過；設定檔本身有錯就不寫）；用 `/permissions` 看、拿掉。

| 欄位 | 預設 | 說明 |
|---|---|---|
| `defaultModel` | `anthropic:claude-opus-5-5` | 預設的廠商和模型。寫系列名稱（`anthropic:opus`）就自動用最新的 |
| `effort` | `{ "default": "high" }` | 每個模型的預設推理強度，鍵是模型 ID、系列名稱或 `default` |
| `defaultMode` | `normal` | `normal`、`auto-edit`、`full-auto`（全自動，等於每次都加 `--auto`，見 usage.md；`xj -p` 不吃這個，要明確加 `--auto`）、`plan`、`read-only`。全部不問不能設成預設：每次都要加 `--dangerously-skip-permissions` |
| `uiLanguage` | 系統的語言 | 介面的語言：`zh-TW` 或 `en`。第一次啟動時問，`/language ui` 會改它 |
| `language` | 跟介面一樣（`繁體中文` 或 `English`） | AI 回覆的語言 |
| `notify` | `false` | 一輪跑完、要問你的時候響一聲 |
| `screen` | `fullscreen` | 畫面：`fullscreen` 是全螢幕模式（寬的視窗右邊放待辦清單和目前的狀態，PageUp／PageDown 和滾輪捲動）；`classic` 是一般模式（終端機自己捲動、搜尋、選取複製）。全螢幕開不了的時候（輸出不是終端機、舊的 Windows 主控台）自動用一般模式。`/tui` 切換時會寫進來 |
| `enterToApprove` | `true` | 詢問時按 Enter 就是「是」（預設開，跟 Claude Code 一樣；寫 `false` 就一律要按 y）（網域的詢問是「允許一次」）。危險的指令、還沒看完的內容、永久允許的確認照樣要按 `y`；詢問剛出現的 0.5 秒內的 Enter 不算（可能是你剛好在按）。信任資料夾的問題不能用 Enter |
| `statusBar.fields` | `model`、`effort`、`mode`、`sandbox`、`context`、`todo`、`day-cost`、`month-cost`、`progress` | 狀態列顯示哪些欄位，照重要程度排：視窗太窄時從最後面開始省略。另外有 `session-cost`（這次對話的花費）。`todo` 是待辦清單（沒有清單、或全部完成時不佔位置；2026-10-22 加的，之前就自己寫了這一項的，要自己加上 `todo` 才看得到） |
| `budget` | 每輪 $5、每天 $30、每月 $100、帳單週期從 1 號開始 | 到 80% 提醒、到 100% 停下來問。上限是拿**預估**的花費來比的（本機價格表 × 用量），不是服務商的帳單；填 0 等於關掉那一項。`xj setup` 不會問你這個 |
| `limits` | 每輪 50 次工具呼叫、輸出 64000 token | 碰到上限就停下來問 |
| `permissions` | 空的 | 白名單（`allow`）、危險清單（`danger`）、機密檔（`secretFiles`）的增減；額外可讀的路徑（`extraReadable`：家目錄本身、放憑證的資料夾像 `~/.ssh`、`~/.aws`，和包含它們的上層，都不採用，開場會講） |
| `sandbox` | 見 schema | 可讀的工具鏈、可寫的快取、允許的網域、本機連線；`gitconfig`：沙盒裡的 git 讀哪一份全域設定 —— `"filtered"`（預設）只留 `user.name`、`user.email` 這類無害的設定（你的 alias、`insteadOf` 在沙盒裡不生效），`"full"` 是整份 `~/.gitconfig`（裡面的網址、token 沙盒裡的指令都讀得到）；`excludedCommands`：在沙盒外跑的指令（預設是空的），見下面「Docker」；`allowUnsandboxedCommands`：沙盒裡跑不了的工具（flutter 這類）能不能要求在沙盒外跑（預設 `true`，每次都問），見下面「沙盒裡跑不了的工具」。`autoFixWritable`（Windows）：開沙盒時發現沙盒的帳號寫得進 PATH 上的地方，不問就用系統管理員權限擋（照樣跳 UAC；預設 `false`：問你一次，選「永久允許」就是寫成 `true`；`/permissions` 拿得掉），見 usage.md 的「全部不問模式」。`writablePrograms`（Windows）：沙盒外會跑的程式（xj 的 git、Git Bash、`excludedCommands`、flutter 這類、MCP server、搜尋用的 rg）放在沙盒寫得進去的地方時怎麼辦 —— `"ask"`（預設）開沙盒時問一次、這次對話有效，`"deny"` 不問、不在沙盒外跑（搜尋改在沙盒裡跑），`"allow"` 不問、照樣跑（每次用到照樣標紅）；選「永久允許」就是寫成 `"allow"`，`/permissions` 拿得掉，見 cerberus.md。Windows（2026-10-21）：開放一個資料夾要逐檔改權限，所以 `toolchainReadable`、`writable` 用預設值的時候照專案的種類開（有 `package.json` 才開 npm 全域、有 `build.gradle` 這類 Gradle 檔才開 Android SDK 和 `~/.gradle`…；Flutter 專案（`pubspec.yaml`）不開 Flutter SDK、Pub cache，它底下的 `android/` 也不算 —— flutter、dart 一律在沙盒外跑，開了只會讓開沙盒多等），大的工具鏈只開用得到的部分；自己寫了這兩項的話照你寫的開，xj 不再自己加 |
| `windows` | `{}` | Windows 才用（2026-10-20）：`gitBash` 是 Git Bash 的 `bash.exe` 完整路徑、`powershell` 是 `pwsh.exe` 或 `powershell.exe` 的完整路徑。沒寫就自己找（Git Bash 照 PATH 上的 git.exe 和預設的安裝位置；PowerShell 先找 7 再用內建的）。不能是專案裡的檔案，也不能是 `C:\Windows\System32\bash.exe`（那是 WSL） |
| `env.passthrough` | `JAVA_HOME`、`ANDROID_HOME` | 指令的乾淨環境裡額外留下的變數 |
| `keyFiles` | 空的 | 沒有鑰匙圈的環境：廠商 → 金鑰檔。路徑要是絕對路徑或 `~` 開頭（Windows 要有磁碟代號，例如 `C:\keys\anthropic.txt`）：相對路徑會從工作目錄讀，設定檔不收。這些路徑自動算成機密檔 |
| `providers` | 內建 `openai`、`ollama` | 改 `baseURL`、用 `enabled: false` 關掉，或加自己的服務（`kind: "chat-completions"`、`baseURL`、`keyName`） |
| `capabilities` | 空的 | 覆寫模型的能力（上下文、最大輸出、工具組…），鍵是「廠商:模型」。`todoTool`：給不給這個模型待辦清單的工具（`todo_write`）；`webFetch`：給不給抓網頁的工具（`web_fetch`）。預設每個模型都給，不想讓某個模型用就寫 `false`。`subagents`：給不給子代理的工具（`agent`），預設也給，不想讓某個模型用就寫 `false` |
| `subagents` | 同時 4 個、30 個工具呼叫、10 分鐘 | 子代理（見 usage.md 的「子代理」）：`maxConcurrent` 同時跑幾個（只有唯讀的 explore 會同時跑）、`maxToolCalls`、`timeoutMinutes` 每個的上限（到了就叫它交報告）、`model` 預設用的模型（沒寫跟主對話同一個） |
| `mcp.servers` | 空的 | MCP server：`command`、`args`、`env`、`description`、`sandbox`、`autoApprove`、`readOnly`。`env` 裡名稱看起來就是金鑰的（結尾是 `_KEY`、`_TOKEN`、`_SECRET`、`_PASSWORD` 這些），值會被當成金鑰：server 的輸出、錯誤訊息、對話紀錄裡出現就遮掉。Windows 的沙盒現在跑不了 MCP server（stdin 接不進去）：沙盒開著的話要設 `sandbox: false`（在沙盒外、以你的權限跑） |
| `anthropic` | `fallback: default`、`prefixMismatch: drop_block` | Anthropic 專用的設定 |
| `evals.budgetUsd` | `2` | 每次評測最多花多少 |
| `sessions.retentionDays` | `30` | 對話紀錄保留幾天 |

## 沙盒裡跑不了的工具（Flutter）

有些工具在沙盒裡根本跑不起來：`flutter`、`dart`（和 `fvm`）每次執行都要寫 Flutter SDK 自己的資料夾（`bin/cache`），沙盒裡寫不進去，它會一直卡住。
開放那個資料夾也不行：裡面的 `dart.exe` 這些之後會在沙盒外以你的權限執行，等於讓沙盒裡的程式改得到它們（2026-10-27 Windows 實測）。

所以（照 Claude Code 的做法）：沙盒開著時模型叫到這些指令，xj 不讓它卡住，直接告訴模型原因；模型會要求**在沙盒外跑那一個指令**，
xj 就問你一次（標紅寫「在沙盒外用你的權限跑」和實際跑的是哪一支），按「是」才跑。不用改設定、不用關沙盒，其他指令照樣在沙盒裡。

- 每次都問，全自動模式也問，不會有「永久允許」；跟 `excludedCommands` 一樣只放行單純的寫法（要單獨跑，不能接在別的指令後面）。
- 只有這張表上的工具能要求在沙盒外跑（現在是 flutter、dart、fvm，和下面的安裝）；其他指令要求的話一律拒絕。以後有類似的工具會加進同一張表。

**安裝工具**：這台沒裝某個工具時，模型會提議安裝，同一條路：每次都問你，按「是」才在沙盒外跑。
- 只有 Windows 的 `winget install --scope user …`、`scoop install …`（加上它們的 search 這類查詢），macOS 的 `brew install …`。其他的安裝程式（choco、`npm -g`、pip…）不行。
- `winget install` 一定要 `--scope user`（裝在你的帳號底下，不用系統管理員權限）。
- **原則上不用系統管理員權限**：要的話，模型把完整的指令給你，你在系統管理員的終端機自己跑。你在對話裡明確說「可以直接跑」時，模型才會要求用系統管理員權限跑
  （只有 `winget install`）：xj 照樣問你（標紅），Windows 還會跳 UAC 讓你按；全部不問模式也一樣問。
- 裝好之後要**重開 xj**：PATH 和沙盒讀得到的範圍都是啟動時算的，新裝的指令要重開才找得到。
- winget 裝在你帳號底下的（`--scope user`）沙盒讀得到：xj 一律把 `%LOCALAPPDATA%\Microsoft\WinGet\Packages`、`Links` 開放給沙盒讀（跟 Scoop 一樣）。
  **用系統管理員權限裝到整台電腦的，沙盒裡用不了**：winget 給那些檔案的權限沒有一般使用者，沙盒用另一個帳號跑（2026-10-28 實測）。沙盒外照常可以用；要在沙盒裡用就裝成 `--scope user`。
- 不想要這個選項：設定檔寫 `"sandbox": { "allowUnsandboxedCommands": false }`，這些指令就一律拒絕；要用的話用 `xj --no-sandbox` 重開，或列進 `excludedCommands`。
- 自己列進 `excludedCommands` 的照原本的做法（問、在沙盒外跑），不用模型要求。

## Docker：在沙盒外跑的指令

沙盒擋住了 Docker 的連線（`docker.sock`：能叫 Docker 做事，就能把整台電腦的檔案掛進容器），所以 `docker` 在沙盒裡一定失敗。
要讓它用 Docker，又不想整個關掉沙盒，就把指令列進 `sandbox.excludedCommands`：

```json
{ "sandbox": { "excludedCommands": ["docker"] } }
```

- 寫 `"docker"` 是整個 docker；寫 `"docker compose"` 只有那個子指令。要寫指令的名字，不能寫路徑。
- 列出來的指令**每次都問**，標紅寫「在沙盒外用你的權限跑」和實際跑的是哪一支；按「是」才跑。全自動模式也照樣問，不會有「都允許」。
- 只放行單純的寫法：可以用 `&&` 串幾個列出來的指令、接 `2>&1`、`>/dev/null`；不能接管線給別的指令、不能寫進檔案、
  不能有 `$( )`、不能在前面加環境變數（`PATH=`、`LD_PRELOAD=` 會換掉實際跑的程式）。寫法不行的時候直接拒絕，並告訴它怎麼改。
- 程式照沙盒外的 PATH 找：專案、暫存區裡放的同名檔案（沙盒裡的指令放得進去）不算。
- 不信任的資料夾、唯讀和計劃模式不在沙盒外跑。機密檔照樣一律擋。
- Docker 會照專案裡的 `Dockerfile`、`docker-compose.yml` 做事，而這些檔案沙盒裡的指令改得到：按「是」之前先看過 `git diff`。
- 要連遠端的 Docker（`DOCKER_HOST`），把變數加進 `env.passthrough`。

全部不問模式（`--dangerously-skip-permissions`）用 root 跑的時候預設開不了；在可以整個丟掉的容器裡、又一定要用 root，
設環境變數 `XING_JANUS_ALLOW_ROOT_BYPASS=1`（只認 `1`）。這個變數跟其他 `XING_JANUS_` 開頭的一樣，不會傳給它跑的指令。

金鑰不寫在設定檔裡：放鑰匙圈（`xing-janus-<廠商>`）、環境變數 `XING_JANUS_<廠商>_API_KEY`，或 `keyFiles` 指的檔案。不要用 `ANTHROPIC_API_KEY`、`OPENAI_API_KEY` 這類通用的名稱：其他工具也會讀它們。

## 接別的模型服務

只要是 OpenAI 相容的 API，加一個 `providers` 就能用，不用改程式。金鑰放環境變數
`XING_JANUS_<名稱大寫>_API_KEY`，或用 `xj setup` 存進鑰匙圈（項目名是 `xing-janus-<名稱>`）。

```json
{
  "providers": {
    "openrouter": { "kind": "chat-completions", "baseURL": "https://openrouter.ai/api/v1" },
    "gemini": { "kind": "chat-completions", "baseURL": "https://generativelanguage.googleapis.com/v1beta/openai" },
    "local": { "kind": "chat-completions", "baseURL": "http://localhost:8080/v1" }
  },
  "capabilities": {
    "openrouter:anthropic/claude-sonnet-4.5": { "contextTokens": 200000, "maxOutputTokens": 32000 },
    "local:你的模型id": { "contextTokens": 65536, "maxOutputTokens": 8192, "tools": "unreliable", "toolset": "reduced", "parallelToolCalls": false }
  },
  "defaultModel": "openrouter:anthropic/claude-sonnet-4.5"
}
```

三件要注意的：

1. **能力表要自己填。** 查不到模型能力時用的是保守預設（**32k 上下文**、8k 輸出）。所以 1M 上下文的模型
   不寫 `capabilities` 的話只會被當成 32k。模型 id 帶斜線沒問題。
2. **花費會顯示「未知」。** 花費是查 `~/.xing-janus/prices.json`，自己的服務要自己補一筆
   （本機模型就全部填 0，不然每輪會問你「有幾個請求查不到價格，要繼續嗎」）。
3. **本機模型建議降級**：`"tools": "unreliable"`、`"toolset": "reduced"`、`"parallelToolCalls": false`。
   Ollama 會自動這樣（小模型常常不會正確呼叫工具），但「自己的服務」會被當成雲端 API。

本機的服務（llama.cpp 的 `llama-server`、vLLM、LM Studio）用 `http://localhost:<埠>/v1` 就好，
不用金鑰；`http://` 只對 localhost 不會被警告。

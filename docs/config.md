# 設定檔

位置：`~/.xing-janus/config.json`。完整的格式在 [`config.schema.json`](config.schema.json)，`xj doctor` 會照它檢查。
**每一項都有預設值，什麼都不寫也能用**：設定檔只要寫你想改的那幾項（`xj setup` 會寫上預設的模型、介面和回覆的語言、提示音）。下表的「預設」就是沒寫的時候用的值，或 xj 自己怎麼決定。
`xj setup` 會在設定檔裡寫上 `"$schema"`：用 VS Code 這類編輯器打開，會自動補完、顯示每一欄的說明、打錯當場標出來。
詢問時選「永久允許」會寫進 `permissions.allow`、`sandbox.allowedDomains`、MCP server 的 `autoApprove`，Windows 開沙盒時問的那兩題寫進 `sandbox.autoFixWritable`、`sandbox.writablePrograms`（只改那一欄，寫之前整份檢查過；設定檔本身有錯就不寫）；用 `/permissions` 看、拿掉。

| 欄位 | 預設 | 說明 |
|---|---|---|
| `version` | `1` | 設定檔格式的版本：xj 自己寫，格式改了會自動轉換，不用自己改 |
| `defaultModel` | `anthropic:claude-opus-5-5` | 預設的廠商和模型。寫系列名稱（`anthropic:opus`）就自動用最新的。模型的名字照服務給的（例如 llama.cpp 的檔案路徑），不能有控制碼和看不見的字。`xj setup` 會照你選的那一家改寫這一項，`/model` 換了也會改寫 |
| `effort` | `{ "default": "high" }` | 每個模型的預設推理強度，鍵是模型 ID、系列名稱或 `default`。模型不支援那個等級的話用模型自己的預設；不支援推理強度的模型不送 |
| `defaultMode` | `normal` | `normal`、`auto-edit`、`full-auto`（全自動，等於每次都加 `--auto`，見 usage.md）、`plan`、`read-only`。`xj -p` 不吃 `auto-edit`、`full-auto`（用一般模式），全自動要明確加 `--auto`。全部不問不能設成預設：每次都要加 `--dangerously-skip-permissions` |
| `uiLanguage` | 照系統的語言 | 介面的語言：`zh-TW` 或 `en`。沒寫的話看環境變數 `LC_ALL`、`LC_MESSAGES`、`LANG` 第一個有設的（`C`、`POSIX` 不算），都沒設就照作業系統的地區設定：`zh` 開頭的用繁體中文，其他用英文。第一次啟動時問（照這個預先選好），`/language ui` 會改它 |
| `language` | 跟介面一樣（中文介面是 `繁體中文`、英文介面是 `English`） | AI 回覆的語言，照你平常的寫法寫（例如 `English`、`Deutsch`） |
| `notify` | `false` | 一輪跑完、要問你的時候響一聲 |
| `screen` | `fullscreen` | 畫面：`fullscreen` 是全螢幕模式（寬的視窗右邊放待辦清單和目前的狀態，PageUp／PageDown 和滾輪捲動）；`classic` 是一般模式（終端機自己捲動、搜尋、選取複製）。全螢幕開不了的時候（輸出不是終端機、舊的 Windows 主控台）自動用一般模式。`/tui` 切換時會寫進來 |
| `enterToApprove` | `true` | 詢問時按 Enter 就是「是」（預設開，跟 Claude Code 一樣；寫 `false` 就一律要按 y）（網域的詢問是「允許一次」）。危險的指令、還沒看完的內容、永久允許的確認照樣要按 `y`；詢問剛出現的 0.5 秒內的 Enter 不算（可能是你剛好在按）。信任資料夾的問題不能用 Enter |
| `statusBar.fields` | `model`、`key`、`effort`、`mode`、`sandbox`、`context`、`todo`、`day-cost`、`month-cost`、`progress` | 狀態列顯示哪些欄位，照重要程度排：視窗太窄時從最後面開始省略。另外有 `session-cost`（這次對話的花費）。`todo` 是待辦清單（沒有清單、或全部完成時不佔位置；2026-10-22 加的，之前就自己寫了這一項的，要自己加上 `todo` 才看得到）。`key` 是同一家好幾把 key 的話現在用哪一把（只有一把的話不佔位置；2026-10-29 加的，自己寫了這一項的一樣要自己加） |
| `budget` | 每輪 $5、每天 $30、每月 $100、帳單週期從 1 號開始 | `perTurnUsd`、`perDayUsd`、`perMonthUsd`（美元），`cycleStartDay`（帳單週期從每月幾號開始，1～28）、`unknownRequestsPerTurn`（一輪裡幾個查不到價格的請求就停下來問，預設 10：算不出花了多少錢，金額的上限管不到它們；用本機的服務不想被問的話填 0，或在價格表替那個模型填 0）。到 80% 提醒、到 100% 停下來問。上限是拿**預估**的花費來比的（本機價格表 × 用量），不是服務商的帳單；填 0 等於關掉那一項。`xj setup` 不會問你這個 |
| `limits` | 每輪 100 次工具呼叫、每次請求最多輸出 64000 token | `toolCallsPerTurn`：每輪最多幾次工具呼叫，碰到就停下來問你。`maxOutputTokens`（至少 1024）：每次請求的輸出上限，不會超過模型自己的上限；碰到它不會停下來問（回覆被截斷時提醒一聲，被截斷的工具呼叫不執行） |
| `permissions` | 每一項都是空的（內建的白名單、危險清單、機密檔照樣生效） | **內建的一直都在**（有哪些見下面「內建的白名單、危險清單、機密檔」），這裡只寫你要增減的：`allow`／`allowRemove`（白名單）、`danger`／`dangerRemove`（危險清單，拿掉的寫名稱）、`secretFiles`／`secretFilesRemove`（機密檔）。`extraReadable`：工作目錄以外也讓它讀的資料夾（家目錄本身、放憑證的資料夾像 `~/.ssh`、`~/.aws`，和包含它們的上層，都不採用，開場會講）。沒有要放寬或收緊的話不用寫 |
| `sandbox.toolchainReadable` | `~/.gitconfig`、`~/.config/git`、`~/.cargo`、`~/.rustup`、`~/.pyenv`、`~/.nvm`、`~/.volta`、`~/development/flutter`、`~/fvm`、`~/.pub-cache`、`~/.gradle`、`~/.android`、`~/Library/Android/sdk`（不存在的略過） | 沙盒裡的指令讀得到的工具鏈（工作目錄以外；家目錄的其他地方讀不到）。`~/.gitconfig`、`~/.config/git` 只有 `sandbox.gitconfig` 是 `"full"` 時才開（預設的 `"filtered"` 給沙盒的是過濾過的那一份）。Windows（2026-10-21）：開放一個資料夾要逐檔改權限，所以用預設值的時候照專案的種類開（有 `package.json` 才開 npm 全域、有 `build.gradle` 這類 Gradle 檔才開 Android SDK 和 `~/.gradle`…；還看不出是哪種專案時（全新的空資料夾），Node、Python 的照開；Flutter 專案（`pubspec.yaml`）不開 Flutter SDK、Pub cache，它底下的 `android/` 也不算 —— flutter、dart 一律在沙盒外跑，開了只會讓開沙盒多等），大的工具鏈只開用得到的部分；自己寫了這一項或 `writable` 的話照你寫的開，xj 不再自己加 |
| `sandbox.writable` | `~/.pub-cache`、`~/.gradle` | 沙盒裡的指令寫得到的地方（工作目錄和暫存資料夾一定寫得到，不用寫） |
| `sandbox.allowedDomains` | 空的 | 沙盒裡的指令不用問就能連的網域。空的就是每連一個新的網域都先問你（允許一次、這次對話、永久允許；選永久允許就寫進這裡） |
| `sandbox.allowLocalBinding` | `false` | 讓沙盒裡的指令聽本機的連接埠（Flutter、Gradle 這類要的話設成 `true`）。**macOS 上同時放行連到這台電腦的每一個本機連接埠**（Ollama、資料庫、Docker 這類只聽本機的服務），不經過網域的詢問；需要的時候才開 |
| `sandbox.gitconfig` | `filtered` | 沙盒裡的 git 讀哪一份全域設定：`"filtered"` 只留 `user.name`、`user.email` 這類無害的設定（你的 alias、`insteadOf` 在沙盒裡不生效），`"full"` 是整份 `~/.gitconfig`（裡面的網址、token 沙盒裡的指令都讀得到） |
| `sandbox.excludedCommands` | 空的 | 在沙盒外跑的指令（每次都問），見下面「Docker」 |
| `sandbox.allowUnsandboxedCommands` | `true` | 沙盒裡跑不了的工具（flutter 這類）能不能要求在沙盒外跑那一個指令（每次都問），見下面「沙盒裡跑不了的工具」 |
| `sandbox.autoFixWritable` | `false` | Windows：開沙盒時發現沙盒的帳號寫得進 PATH 上的地方，不問就用系統管理員權限擋（照樣跳 UAC）。`false` 是問你一次，選「永久允許」就是寫成 `true`；`/permissions` 拿得掉，見 usage.md 的「全部不問模式」 |
| `sandbox.writablePrograms` | `ask` | Windows：沙盒外會跑的程式（xj 的 git、Git Bash、`excludedCommands`、flutter 這類、MCP server、搜尋用的 rg）放在沙盒寫得進去的地方時怎麼辦 —— `"ask"` 開沙盒時問一次、這次對話有效，`"deny"` 不問、不在沙盒外跑（搜尋改在沙盒裡跑），`"allow"` 不問、照樣跑（每次用到照樣標紅）；選「永久允許」就是寫成 `"allow"`，`/permissions` 拿得掉，見 cerberus.md |
| `windows` | `{}` | Windows 才用（2026-10-20）：`gitBash` 是 Git Bash 的 `bash.exe` 完整路徑、`powershell` 是 `pwsh.exe` 或 `powershell.exe` 的完整路徑。沒寫就自己找（Git Bash 照 PATH 上的 git.exe 和預設的安裝位置；PowerShell 先找 7 再用內建的；Microsoft Store 裝的 7 也算，沙盒開著時不用它）。不能是專案裡的檔案，也不能是 `C:\Windows\System32\bash.exe`（那是 WSL） |
| `env.passthrough` | `JAVA_HOME`、`ANDROID_HOME` | 指令跑在乾淨的環境裡：只留 `PATH`、`HOME`、`LANG`、`TERM`、`TMPDIR`（Windows 另外留系統要的那幾個，例如 `SystemRoot`），加上這裡列的；金鑰、token 這類不給 |
| `keyFiles` | 空的（照環境變數 → 鑰匙圈或認證管理員找） | 沒有鑰匙圈的環境：廠商 → 金鑰檔。路徑要是絕對路徑或 `~` 開頭（Windows 要有磁碟代號，例如 `C:\keys\anthropic.txt`）：相對路徑會從工作目錄讀，設定檔不收。這些路徑自動算成機密檔。同一家好幾把的話鍵寫成「廠商:名字」（例如 `anthropic:work`） |
| `keys` | 空的（每一家一把） | 同一家好幾把 key：這個資料夾用哪一把、被拒時怎麼辦、某一把另外的上限。見下面「同一家好幾把 key」 |
| `providers` | 內建 `openai`（`https://api.openai.com/v1`）、`ollama`（`http://localhost:11434/v1`）、`gemini`（Google 的 Gemini API，見下面「Gemini」） | 改 `baseURL`、用 `enabled: false` 關掉，或加自己的服務（`kind: "chat-completions"`、`baseURL`、`keyName`、`allowPlainHttpKey`，見下面「接別的模型服務」）。OpenAI、Gemini 找不到金鑰就不接；自己的服務沒有金鑰也接（本機的服務常常不用）；Ollama 不用金鑰。`gemini` 寫了 `baseURL` 的話改走那個網址的 OpenAI 相容端點 |
| `capabilities` | 空的（照各家的模型清單查到的；查不到的用保守的預設：上下文 32k、輸出 8k，狀態列標 `?`） | 覆寫模型的能力（上下文、最大輸出、工具組…），鍵是「廠商:模型」：`contextTokens`、`maxOutputTokens`、`compactAtTokens`（上下文到多少 token 就自動摘要；沒寫的話是 15 萬，但不超過上下文的 80%）、`imageInput`（收不收圖片）、`tools`（`"none"` 是不送工具；`"reliable"`、`"unreliable"` 目前沒有差別；Anthropic 的模型不看這一項）、`parallelToolCalls`（目前沒有作用）、`toolset`（`"full"`，或 `"reduced"`：只給 read、search、edit）。`todoTool`：給不給這個模型待辦清單的工具（`todo_write`）；`webFetch`：給不給抓網頁的工具（`web_fetch`）。預設每個模型都給，不想讓某個模型用就寫 `false`。`subagents`：給不給子代理的工具（`agent`），預設也給，不想讓某個模型用就寫 `false` |
| `subagents` | 同時 4 個、30 個工具呼叫、10 分鐘 | 子代理（見 usage.md 的「子代理」）：`maxConcurrent` 同時跑幾個（只有唯讀的 explore 會同時跑）、`maxToolCalls`、`timeoutMinutes` 每個的上限（到了就叫它交報告）、`model` 預設用的模型（沒寫跟主對話同一個） |
| `mcp.servers` | 空的（不接任何 MCP server） | MCP server：`command`、`args`、`env`、`description`、`sandbox`、`autoApprove`、`readOnly`、`timeoutSec`（一次呼叫最多等幾秒，預設 600）。`env` 裡名稱看起來就是金鑰的（結尾是 `_KEY`、`_TOKEN`、`_SECRET`、`_PASSWORD` 這些），值會被當成金鑰：server 的輸出、錯誤訊息、對話紀錄裡出現就遮掉。Windows 的沙盒現在跑不了 MCP server（stdin 接不進去）：沙盒開著的話要設 `sandbox: false`（在沙盒外、以你的權限跑） |
| `mcpServer` | 唯讀、沙盒外一律不行、只有啟動 `xj mcp` 的那個資料夾、花費每輪 $2／每天 $10 | 別的程式透過 MCP 叫 xj（`xj mcp`）時能做什麼：`mode`、`outsideSandbox`、`projects`、`anyProject`、`budget`。放寬之前先看 [mcp-server.md](mcp-server.md) 的風險 |
| `anthropic` | `fallback: default`、`prefixMismatch: drop_block` | Anthropic 專用的設定。`fallback`：`"default"` 是請求被安全分類器擋下時，讓 Anthropic 的伺服器換一個模型再試、改由它回答（狀態列標「→ 實際回答的模型」，花費照它的價格算），`"off"` 是不換、不再試。`prefixMismatch`：前面的對話改過（例如摘要之後）、跟模型的推理內容對不上時，`"drop_block"` 是丟掉那一段推理照常送，`"error"` 是直接報錯（測試用） |
| `evals.budgetUsd` | `2` | 開發用：每次評測最多花多少（美元）。評測（`node evals/run.ts`）只在原始碼的 repo 裡，發佈的版本沒有 |
| `sessions.retentionDays` | `30` | 對話紀錄保留幾天：每次啟動自動清掉更舊的（`xj sessions prune` 也能手動清） |

## 內建的白名單、危險清單、機密檔

這些不用設定就有；`permissions` 只寫你要增減的。

- **白名單**（沙盒開著時不用問就跑；沒有沙盒的時候（`--no-sandbox`、Windows 沒裝沙盒）不生效，每個指令都問）：唯讀的 `git status`、`git diff`、`git log`、`git show`、`git branch`、`git rev-parse`、`git ls-files`、`git blame`，`ls`、`pwd`、`cat`、`head`、`tail`、`wc`、`file`、`stat`、`which`、`echo`、`date`、`tree`、`du`、`df`、`rg`、`grep`、`find`；
  測試、檢查用的 `npm test`、`npm run test`、`npm run lint`、`npm run typecheck`、`npm run build`、`npx tsc`、`npx vitest`、`npx eslint`、`npx prettier`、`pytest`、`cargo test`、`go test`、`flutter test`、`flutter analyze`、`dart analyze`、`dart test`、`dart format`。
  有些另外看參數（例如 `find -delete`、`git branch -D` 不算）。要加：`permissions.allow`（例如 `"make test"`）；要拿掉：`permissions.allowRemove`（例如 `"npm run build"`）。
- **危險清單**（每次都問，全自動模式也問）。要拿掉某一項，把名稱寫進 `permissions.dangerRemove`；要加，寫指令的開頭進 `permissions.danger`（例如 `"terraform apply"`）：

  | 名稱 | 擋的是 |
  |---|---|
  | `rm-recursive-force` | 刪除檔案（`rm -r`／`-f`／`-d`） |
  | `find-delete` | 刪除檔案（`find -delete`） |
  | `shred` | 徹底刪除檔案 |
  | `git-reset-hard`、`git-checkout-discard`、`git-restore` | 丟掉改動（`git reset --hard`、`git checkout -- …`、`git restore`） |
  | `git-clean` | 刪除沒被追蹤的檔案（`git clean -f`） |
  | `git-stash-drop` | 丟掉暫存的改動（`git stash drop`／`clear`） |
  | `git-branch-delete` | 刪除分支（`git branch -d`／`-D`） |
  | `git-push` | 推送到遠端 |
  | `publish` | 發佈套件 |
  | `remote-shell` | 連到別的機器（`ssh`、`scp`、`rsync`、`sftp`） |
  | `upload` | 往外送資料（`curl`／`wget` 上傳） |
  | `sudo` | 切換成 root 執行（`sudo`、`su`、`doas`、`sudoedit`、`pkexec`、`run0`） |
  | `chmod-chown-recursive` | 大範圍改權限（`chmod`／`chown -R`） |
  | `disk` | 直接寫磁碟（`mkfs`、`dd`） |
  | `power` | 關機或重開機 |
  | `kill` | 強制結束程式（`kill -9`、`killall`） |
  | `system-launch` | 請系統代為執行，可能跑在沙盒外（`open`、`osascript`、`launchctl`、`crontab`） |
  | `windows-launch` | 請 Windows 代為執行，可能跑在沙盒外（`explorer`、`rundll32`、`mshta`、`wscript`） |
  | `windows-elevate` | 切換成系統管理員執行（`gsudo`、`runas`、`-Verb RunAs`，或看不出內容的 PowerShell 指令） |
  | `windows-shell` | 交給 cmd 或 PowerShell 執行（裡面的指令沒有逐一檢查） |
  | `windows-system` | 改 Windows 的系統設定（登錄、排程、服務、權限、環境變數、防火牆） |
  | `keychain` | 存取鑰匙圈、認證管理員（`security`、`cmdkey`、`vaultcmd`） |
  | `system-dirs` | 寫系統目錄 |

- **機密檔**（讀、寫、搜尋都擋，但沙盒裡的指令間接改得到，見 cerberus.md；在對話裡按「是」也不行，只能改設定檔）：`.env`、`.env.*`、`*.pem`、`*.key`、`*.p12`、`*.pfx`、`*.keystore`、`*.jks`、`*.p8`、`*.ppk`（PuTTY），
  SSH 金鑰（`id_rsa`、`id_ed25519`、`id_ecdsa`、`id_dsa`、`id_ed25519_sk`、`id_ecdsa_sk` 和它們的 `.pub`），`credentials`、`credentials.json`、`credentials.toml`、`.npmrc`、`.netrc`、`.pypirc`、`.git-credentials`、`.envrc`，
  Android／Flutter 的 `local.properties`、`key.properties`、`keystore.properties`、`secrets.properties`、`adbkey`，家目錄的 `~/.gradle/gradle.properties`、`~/.docker/config.json`、`~/.kube/config`、`~/.config/gh/hosts.yml`、`~/.m2/settings.xml`、nvm 和 npm 裡的 `npmrc`，
  加上 `keyFiles` 指的金鑰檔。不算的：`.env.example`、`.env.sample`、`.env.template`、`~/.android/debug.keystore`。要加：`permissions.secretFiles`（例如 `"*.secret"`）；要拿掉：`permissions.secretFilesRemove`（例如 `".npmrc"`）。

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
- winget 裝在你帳號底下的（`--scope user`）沙盒讀得到：`sandbox.toolchainReadable` 用預設值的時候，xj 把 `%LOCALAPPDATA%\Microsoft\WinGet\Packages`、`Links` 開放給沙盒讀（跟 Scoop 一樣；自己寫了這一項的話，要自己列進去）。
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

全部不問模式（`--dangerously-skip-permissions`）用 root 跑的時候預設開不了（Windows 上「以系統管理員身分執行」的終端機也一樣）；在可以整個丟掉的容器、虛擬機裡，又一定要用 root（系統管理員）的話，
設環境變數 `XING_JANUS_ALLOW_ROOT_BYPASS=1`（只認 `1`）。這個變數跟其他 `XING_JANUS_` 開頭的一樣，不會傳給它跑的指令。

金鑰不寫在設定檔裡：放鑰匙圈（`xing-janus-<廠商>`）、環境變數 `XING_JANUS_<廠商>_API_KEY`，或 `keyFiles` 指的檔案。不要用 `ANTHROPIC_API_KEY`、`OPENAI_API_KEY` 這類通用的名稱：其他工具也會讀它們。

## 同一家好幾把 key

公司的和自己的、不同客戶各一把：每一把取個名字（小寫英數字和 `-`，例如 `work`）。沒有名字的那一把叫 `default`，就是原本那一把：**只有一把的話什麼都不用改**，畫面也跟以前一樣。

有名字的放法跟原本一樣，名稱後面接名字：

| 放在哪裡 | 沒有名字的那一把 | 名字是 work 的那一把 |
|---|---|---|
| 環境變數 | `XING_JANUS_ANTHROPIC_API_KEY` | `XING_JANUS_ANTHROPIC_API_KEY_WORK`（名字裡的 `-` 換成 `_`） |
| macOS 鑰匙圈、Windows 認證管理員 | `xing-janus-anthropic` | `xing-janus-anthropic--work` |
| 金鑰檔 | `"keyFiles": { "anthropic": "…" }` | `"keyFiles": { "anthropic:work": "…" }` |

`xj setup` 會問要不要再加一把。名字要寫在設定檔裡（`keys` 的 `byFolder`、`failover`、`names`，或 `keyFiles` 的「廠商:名字」），xj 才知道有那一把：環境變數、鑰匙圈不會另外去找。`xj setup` 加的會自己寫進去。

```json
{
  "keys": {
    "anthropic": {
      "default": "personal",
      "byFolder": { "~/work": "work", "~/clients/acme": "acme" },
      "onReject": "ask",
      "failover": ["backup"],
      "budget": { "work": { "perDayUsd": 10 } }
    }
  }
}
```

- **用哪一把**（照這個順序）：指令列的 `--key work`（別家的寫 `--key openai:work`）→ 這次對話用 `/key` 換過的 → `byFolder` 對到的（那個資料夾和底下的子資料夾；`~` 開頭或絕對路徑，也可以用 glob；對到好幾條時最長的那一條優先；比的是實際路徑，經過 symlink 的資料夾（例如 macOS 的 `~/Dropbox`）也對得到）→ `default` → 沒有名字的那一把。**只看你自己的設定檔**：專案裡的檔案不能決定花誰的錢。選到的那一把找不到的話不會改用別把（帳單可能不一樣），會講要放在哪裡。
- **`/key`**：列出這一家有哪幾把（名字、放在哪裡，不顯示內容），選一把換過去（只換這次對話）。換 key 常常等於換帳號：快取會斷（下一個請求比較貴），能用的模型可能不一樣（模型清單照 key 分開快取）。
- **畫面**：開場那一行、狀態列（`key` 欄位）、`/cost` 標出現在用的是哪一把。對話紀錄只記名字，不記內容。`/resume` 照這個資料夾現在的設定用，跟原本那次不一樣時講一聲。
- **花費**：照「廠商＋名字」分開記，`/cost` 分開列。`budget` 照舊算全部；`keys.<廠商>.budget` 可以替某一把另外設每天（`perDayUsd`）、每月（`perMonthUsd`）的上限，到了一樣先提醒、再停下來問。
- **被拒時**（額度用完、key 失效、重試之後還是被限流）：`onReject` 是 `"ask"`（預設：問你要不要換成下一把、接著做這一輪）、`"auto"`（直接換、講一聲）、`"off"`（停下來，你自己 `/key` 換）。只會換到**這個資料夾也能用的**那幾把：`byFolder` 對到這個資料夾的，加上 `failover` 列的（照順序）——不會為了繼續做就換到別的帳單。只換這次對話；`xj -p` 沒有人能回答，`"ask"` 一律不換。
- 設成 `"auto"` 之前先想兩件事：被限流時自動輪流用好幾把 key 繞過上限，**可能違反服務商的條款**（請先看各家的條款）；換過去之後，花費記在那一把的帳上。`xj doctor` 也會提醒。
- 子代理用跟主對話同一把。`xj -p` 和評測（開發用，`node evals/run.ts --key work`）也能加 `--key`。`xj doctor` 每一把各一列（在不在、放在哪裡、連不連得上）。

## Gemini

內建，連 Google 的 Gemini API（原生的轉接層：thought signature 原樣帶回，連續好幾輪的工具呼叫也不會壞）。

- **金鑰**：在 Google AI Studio（或 Google Cloud 主控台）建一把只限 Gemini API 的金鑰。`xj setup` 第一題選 3（Gemini）存起來，或設環境變數 `XING_JANUS_GEMINI_API_KEY`、
  鑰匙圈（認證管理員）的 `xing-janus-gemini`。不要用 `GEMINI_API_KEY`、`GOOGLE_API_KEY`：其他工具也會讀它們（xj 不讀）。`AQ.` 開頭的新金鑰、`AIza` 開頭的舊金鑰都可以。
- **模型**：`gemini:gemini-flash-latest`（Google 的別名，永遠指到最新的 Flash）、`gemini:gemini-3.5-flash-lite`、`gemini:gemini-3.1-pro-preview`…；
  `/model` 列出這把金鑰能用的（語音、圖片、音樂這類不是拿來對話的不列）。
- **推理強度**：Gemini 3 起是 `low`、`high`（`/effort`，預設 `high`）；2.5 照伺服器的預設。模型想的時候 Google 有時會回一段思考的摘要，xj 拿它的開頭當進度顯示。
- **送出去的內容**：金鑰所屬的 Cloud 專案綁了帳單（付費）的話，Google 不拿你送的內容改進產品；**免費額度的金鑰會**，也可能有人工審閱（Gemini API 的附加條款）。
  不想被拿去改進產品、被人工審閱的程式碼用付費的金鑰；完全不能送出去的程式碼不要用雲端的模型（改用本機的）。
- **價格**：預設價格表有主要的幾個 Gemini 模型（抄自 Google 的價格頁）。限期的優惠價標了用到哪一天（`until`）和之後的正式價（`after`），xj 照日期自己換（見下面「限期的優惠價」）。
  你的 `~/.xing-janus/prices.json` 是舊版的 xj 建的話沒有 Gemini，花費會顯示「未知」：`xj doctor` 會講怎麼換成新的。
- **以前接相容端點的**（`providers.gemini` 寫了 `baseURL`，這份文件以前的寫法）：照舊走相容端點。要改用原生的，把 `providers.gemini` 拿掉。
  Google 的論壇上有人回報：AI Studio 現在發的 `AQ.` 開頭的金鑰在相容端點會失敗（`Multiple authentication credentials received`）。

## 接別的模型服務

只要是 OpenAI 相容的 API，加一個 `providers` 就能用，不用改程式。金鑰放環境變數
`XING_JANUS_<名稱大寫>_API_KEY`、鑰匙圈或認證管理員（項目名是 `xing-janus-<名稱>`），或 `keyFiles` 指的檔案。
`xj setup` 問 Anthropic、OpenAI、Gemini，或本機和其他 OpenAI 相容的服務（第 4 項：先列 Ollama、這台找到的本機伺服器，最後一項自己填網址；再問名字、要不要金鑰）；之後再加的服務，金鑰在 xj 裡當場設（`/model` 選它、或新增它的時候貼上），
或自己存（macOS：`security add-generic-password -s xing-janus-<名稱> -a "$USER" -w`；Windows：`cmdkey /generic:xing-janus-<名稱> /user:xj /pass`）。
也可以在 xj 裡加：`/model` 選單的最後一項「新增一個 OpenAI 相容的服務」，或選「這台找到的」本機伺服器（usage.md 的「換平台、換模型」）。
查得到模型清單才寫進這裡的 `providers`（只寫 `kind` 和 `baseURL`）；金鑰當場設的話，存法跟 `xj setup` 一樣。
名字只能用小寫英文、數字和 `-`（小寫英文開頭、最多 32 個字），不能有 `--`（有名字的金鑰用它分開，會跟別家的金鑰撞在一起）；
`prototype`、`constructor`、`con`、`nul` 這類保留的名字不收（設定檔、Windows 的檔名用不了）。

```json
{
  "providers": {
    "openrouter": { "kind": "chat-completions", "baseURL": "https://openrouter.ai/api/v1" },
    "local": { "kind": "chat-completions", "baseURL": "http://localhost:8080/v1" }
  },
  "capabilities": {
    "openrouter:anthropic/claude-sonnet-4.5": { "contextTokens": 200000, "maxOutputTokens": 32000 },
    "local:你的模型id": { "contextTokens": 65536, "maxOutputTokens": 8192, "toolset": "reduced" }
  },
  "defaultModel": "openrouter:anthropic/claude-sonnet-4.5"
}
```

三件要注意的：

1. **能力表要自己填。** 查不到模型能力時用的是保守預設（**32k 上下文**、8k 輸出）。所以 1M 上下文的模型
   不寫 `capabilities` 的話只會被當成 32k。模型 id 帶斜線沒問題。
2. **花費會顯示「未知」。** 花費是查 `~/.xing-janus/prices.json`（格式見下面「限期的優惠價」那一段的例子），自己的服務要自己補一筆
   （本機模型就全部填 0，不然一輪裡查不到價格的請求累積到 10 個，就會停下來問你要不要繼續）。
3. **本機模型建議降級**：`"toolset": "reduced"`（只給 read、search、edit）。
   Ollama 會自動這樣（小模型常常不會正確呼叫工具），但「自己的服務」（LM Studio、llama.cpp、vLLM…）會被當成雲端 API，要自己寫。

本機的服務（llama.cpp 的 `llama-server`、vLLM、LM Studio）用 `http://localhost:<埠>/v1` 就好，
不用金鑰；`http://` 只對 localhost 不會被警告。
沒加密的 `http://`、又不在這台的（區網裡別台電腦的 vLLM 這種）：金鑰會以明文送出，xj 不送（開 xj、在 xj 裡換過去、當場設金鑰、`xj doctor` 都一樣），
不用金鑰的服務照常用。真的要送的話，在那一家加上 `"allowPlainHttpKey": true`（同一個網路上的人看得到金鑰）。
llama.cpp 的 `llama-server` 沒加 `--alias` 的話，模型的名字就是 `-m` 給的檔案路徑（例如 `C:\models\qwen2.5-7b-instruct-q4_k_m.gguf`）：
照樣選得到、記得住，嫌長的話開的時候加 `--alias qwen`，之後用 `/model llamacpp:qwen` 選它（`llamacpp` 是 `/model` 找到它時取的名字，自己加的照你取的）。

## 限期的優惠價

價格表（`~/.xing-janus/prices.json`）裡，一個模型的那一筆可以標「優惠價用到哪一天」：這一筆的單價（`inputPerMTok` 這些）用到 `until` 那一天為止，
之後照 `after` 裡的單價。日期照 UTC 算（官方寫的多半是美國時間：UTC 會早幾個小時換成之後的單價，寧可估多）。
兩個要一起寫；寫錯的話那一筆不用（等於沒有價格），開場和 `xj doctor` 會講。例如預設價格表裡的這一筆（在 `providers.gemini` 底下）：

```json
"gemini-4-argon": {
  "inputPerMTok": 2, "outputPerMTok": 10, "cacheReadPerMTok": 0.1,
  "until": "2026-11-30",
  "after": { "inputPerMTok": 4, "outputPerMTok": 20, "cacheReadPerMTok": 0.2 },
  "source": "https://aistudio.google.com/docs/pricing", "asOf": "2026-11-04"
}
```

`after` 也可以有自己的 `longContext`（超過門檻的那一組單價）；沒寫的話之後就沒有長上下文的另一組價格。

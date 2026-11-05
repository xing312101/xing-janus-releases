# 讓別的程式透過 MCP 叫 xj（xj mcp）

`xj mcp` 讓別的程式把 Xing Janus 當成一個工具來叫：Claude Desktop、IDE、別的 agent，或你自己寫的通訊軟體橋接（Telegram、Discord 這類）。
它是一個 MCP server，走 stdio：由 MCP client 啟動，不開對外的連接埠。連網路的跟終端機裡的 xj 一樣：模型的 API、網頁工具，和沙盒給裡面的指令用的本機代理。

xj 本身不接通訊軟體。想從手機叫 xj 做事的話，寫（或找）一個「橋接」程式：它保管 bot 的 token、收你的訊息，當 MCP client 叫 xj；
xj 要你核准的時候用 MCP 的 elicitation 問，橋接把問題變成按鈕、把你的答案傳回來。這一頁後半是給寫橋接的人看的介面。

## 開始之前

- 先在終端機開一次 `xj`，同意使用提醒、設好金鑰。
- 要讓 xj 做事的資料夾，要先在電腦前面用 `xj` 開一次、選信任。**透過 MCP 不能信任新的資料夾**。
- 沙盒一定要開著，而且要整個擋得住（`xj doctor` 的「OS 沙盒」那一列；Windows 上「擋不全」的時候也不行）。沒有沙盒的話 `xj_task` 一律拒絕；`--no-sandbox`、全部不問、`--auto` 在這裡都不能用。
- 改了設定檔的 `mcpServer` 要重開 MCP client：`xj mcp` 只在啟動時讀設定；設定改過之後它不接新的事，叫你重開（放寬、收緊都一樣）。

## 接到 client

Claude Desktop（`claude_desktop_config.json`）、Claude Code（專案裡的 `.mcp.json`）這類都是同一種寫法：

```json
{
  "mcpServers": {
    "xing-janus": { "command": "xj", "args": ["mcp"] }
  }
}
```

Windows 上 `command` 寫 `xj.cmd`（或完整路徑 `C:\Users\<你>\AppData\Local\Programs\xj\xj.cmd`）。
client 在哪個資料夾啟動 `xj mcp`，那個資料夾就是預設的專案；要用別的專案，看下面的 `mcpServer.projects`。
**Claude Desktop 不照設定檔的 `cwd`，在 `C:\Windows\System32` 這類系統資料夾啟動 `xj mcp`**（2026-11 實測）：那裡沒信任過，`xj_task` 會拒絕。
用 Desktop 的話一定要在 `mcpServer.projects` 列專案，請它叫 `xj_task` 時帶 `project`（例如「用 xj_task、project 是 web，…」）。

## 設定（設定檔的 `mcpServer`）

透過 MCP 能做什麼由你決定。沒寫的話取安全的那一邊：

| 欄位 | 預設 | 說明 |
|---|---|---|
| `mode` | `read-only` | 透過 MCP 用哪個模式：`read-only`、`plan`、`normal`、`auto-edit`、`full-auto`（跟對話裡的模式一樣）。`bypass`（全部不問）一律不行 |
| `outsideSandbox` | `never` | 沙盒外的動作（`excludedCommands`、沙盒裡跑不了的工具、裝工具、系統管理員權限、設定檔裡 `sandbox: false` 的 MCP server）：`never` 一律拒絕（要在電腦前面做），`ask` 用 elicitation 問（那種 MCP server 設了 `autoApprove` 的工具也問） |
| `projects` | `{}` | 名字 → 資料夾（完整路徑或 `~` 開頭），client 用名字選。沒列的話只有啟動 `xj mcp` 的那個資料夾 |
| `anyProject` | `false` | client 可以直接傳資料夾的路徑（照樣只能是信任過的：信任過的資料夾底下的子資料夾也算，之後才建的也是） |
| `budget` | 每輪 $2、每天 $10 | 透過 MCP 時的花費上限：`perTurnUsd`、`perDayUsd`（美元，要大於 0）。跟 `budget` 一起算、取比較低的；每天的是全部的花費（終端機用的也算）。到了上限就停，不問要不要繼續 |

例如讓你自己的橋接能在兩個專案裡寫程式：

```json
{
  "mcpServer": {
    "mode": "normal",
    "projects": { "web": "~/code/web", "api": "~/code/api" },
    "budget": { "perDayUsd": 10 }
  }
}
```

**放寬之前要知道的**：

- `mode` 設成 `normal` 以上：叫 xj 的程式能改你的專案、在沙盒裡跑指令。叫它的不只是「那個程式」，還有它背後的人和它收到的訊息
  （例如橋接收到別人傳的訊息、另一個 agent 讀到的網頁）。`normal` 改檔、白名單以外的指令照樣問，但按按鈕的人不在電腦前面；
  `auto-edit` 改檔不問；`full-auto` 改檔和沙盒裡的指令都不問（危險的、連新的網域、沙盒外的才問）。
- `outsideSandbox` 設成 `ask`：沙盒外就是你的權限。按的人的帳號、手機被別人拿到時，按鈕擋不住。
- `anyProject`：client 可以挑任何一個你信任過的資料夾，包括它底下的子資料夾。
- 放寬了的地方，每次啟動 `xj mcp` 會印在 stderr（client 的紀錄裡看得到），`xj doctor` 也會講。

## 工具（給寫插件的人：之後改了會照顧相容）

xj 的工具說明、結果、錯誤一律英文（給 client 的模型看）；elicitation 的問題照你設的介面語言（給人看）。

### `xj_task`

交一件事、等這一輪做完。

| 參數 | 型別 | 說明 |
|---|---|---|
| `prompt` | string（必填） | 要做什麼（最多 100,000 個字）。裡面的 `@路徑`、檔案路徑不會附加檔案（那是在電腦前面打字時才有的）：要看檔案就請它讀 |
| `project` | string | `mcpServer.projects` 的名字；`anyProject` 開著時也可以是路徑。沒給是啟動 `xj mcp` 的那個資料夾 |
| `conversation` | `"continue"`（預設）或 `"new"` | 同一個專案接著上一次的對話，或開新的 |

回傳：`content[0].text` 是給人看的整理（回覆、做了的事、沒做的、提醒、花費），`structuredContent` 是：

```json
{
  "project": "web",
  "reply": "最後的回覆（做事之前講的話和最後的回答，中間空一行）",
  "actions": ["讀取 package.json", "執行 npm test"],
  "blocked": ["建立 x.txt (user)"],
  "notices": ["…"],
  "ended": "done",
  "costUsd": 0.0123
}
```

`ended` 是 `done`、`interrupted`、`stopped`、`error`、`refused`；`error`、`refused` 時 `isError` 是 true。`costUsd` 查不到價格時是 `"unknown"`。
`blocked` 每一項括號裡是誰擋的：`user`（你拒絕了）、`rules`（模式或權限規則）、`sandbox`、`trust`、`interrupt`、`input`。
不行的情況（沒信任過的資料夾、沒有沙盒、正在做別的事…）回 `isError: true` 和原因。`project` 給的是路徑時，不存在、沒信任過回同一句話（client 探測不出路徑存不存在），回的 `project` 是你給的那個寫法。

同一時間只做一件；做的時候再叫會回「正在忙」。花費到了上限（`mcpServer.budget`、`budget`）就停（`ended` 是 `stopped`，`notices` 講原因），不問要不要繼續。請求帶了 `_meta.progressToken` 的話，每個工具呼叫、待辦清單的變化都送 `notifications/progress`
（`message` 是那一步的說明）：client 可以照這個重設逾時。

### `xj_status`

參數：`project`（同上）。回模型、模式、沙盒、今天和本月的花費、待辦清單、是不是正在做事（`structuredContent` 有同樣的欄位）。

### `xj_stop`

中斷正在做的事。client 送 `notifications/cancelled` 取消 `xj_task` 的請求也一樣。

## 要核准的時候（elicitation）

client 在 `initialize` 時宣告 `capabilities.elicitation`（協定版本 `2025-06-18`），xj 要問的時候送 `elicitation/create`：

- `message`：要做什麼、為什麼要問、要注意的、完整的指令或 diff。送出去之前先遮金鑰（問題會經過橋接、離開這台電腦），控制碼標成看得到的樣子，
  只能一行的（要做什麼、為什麼、網址）換行收成空格。
- `requestedSchema`：一個欄位 `answer`，字串、`enum` 是可以選的值，`enumNames` 是給人看的字。
  - 改檔、跑指令：`yes`（允許一次）、`no`（拒絕）。沒有「永久允許」「這次對話都允許」：這兩個只能在電腦前面給。
  - 連一個網域：`once`、`no`。「這次對話都允許」會把網域加進沙盒的白名單，跟永久允許一樣要在電腦前面給（常用的網域寫進設定檔的 `sandbox.allowedDomains`）。
  - 停下來問要不要繼續（工具呼叫的上限這類）：`yes`、`no`。預算到了不問，直接停。
- 回 `{ "action": "accept", "content": { "answer": "yes" } }`。`decline`、`cancel`、或不是給過的值，都當成拒絕。
- 一次只問一個：前一個答了才送下一個。10 分鐘沒回答當成拒絕（xj 送 `notifications/cancelled` 收回那個問題）；這一件中斷了，還沒答的也收回。
  沒有在做事的時候（例如沙盒裡留著的程式要連網域）不問，直接拒絕。

client 不支援 elicitation 的話，要核准的一律拒絕（跟 `xj -p` 一樣）。2026-11 實測：Claude Code 支援（問題顯示在它的畫面上、選項照 `enumNames`）；
Claude Desktop 還不支援（`initialize` 沒有宣告 elicitation），透過它只做得了不用核准的事（例如預設的唯讀）。Claude Code 的 `-p`（不互動）宣告了 elicitation，
但沒有人回答，要核准的一律被拒絕。橋接程式要把 `message` 整段給人看：太長的話附成檔案，不要只給摘要。

## 協定的細節

stdio 上一行一則 JSON-RPC 2.0。協定版本收 `2025-06-18`、`2025-03-26`、`2024-11-05`（elicitation 只有 `2025-06-18` 有）。batch（陣列）照 JSON-RPC 處理、回一個陣列；
同一個 id 的請求還在跑時，再送一次回錯誤（`-32600`）；送了 `notifications/cancelled` 的請求不再回結果。一行最長 8 MB。

## 紀錄

透過 MCP 的對話照樣存在 `~/.xing-janus/sessions`：記 client 的名字和版本、用的專案，每一次問了什麼、怎麼答（含逾時、沒有回答）、被拒絕的原因都記。

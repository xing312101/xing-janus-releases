# 資料在哪、送到哪、怎麼刪

## 送到哪

- 你打的字、它讀到的檔案內容、指令的輸出，會送到你選的模型服務（Anthropic、OpenAI、Gemini、本機的 Ollama，或你接的其他 OpenAI 相容的服務），適用該服務的條款。
  Gemini 用免費額度的金鑰的話，Google 會拿送去的內容改進它的產品（config.md 的「Gemini」）。
- Xing Janus 本身不收集任何資料：沒有遙測、沒有錯誤回報、不自動檢查更新。對外的連線只有模型的 API、你允許的網域，和你打 `xj update` 時連的 GitHub（只查版本、下載發佈檔，不送任何資料）。
- 抓網頁（`web_fetch`）：那個網站看得到整個網址、你的 IP，和 `xing-janus/版本` 這個 User-Agent；不送 cookie、不送登入資訊。
  抓到的內容跟讀到的檔案一樣，會送給模型、存進對話紀錄。網址不會另外送去別的地方檢查。
- 用 `/model` 換到別的平台：之前的對話（包括它讀過的檔案內容、指令的輸出）轉成文字送到那一家，之後的也都送到那一家。對話有內容的話換之前會問你、講會送到哪一台
  （換到這台的平台不問：對話不離開這台）。
- 子代理：用的模型跟主對話一樣送到同一家；你指定別家的模型的話，交代的內容和它讀到的東西送到那一家。
- 用 `xj mcp` 讓別的程式叫 xj 時：回覆、做了的事，和要你核准的問題（完整的指令、diff）交給叫它的程式（例如 Claude Desktop、你寫的通訊軟體橋接），
  之後送到哪裡由那個程式決定（橋接會送到通訊軟體）。交出去之前已知格式的金鑰會被遮掉（[mcp-server.md](mcp-server.md)）。
- 模型服務回了轉址（3xx）的話不跟：金鑰和對話不會跟著送到別的地方。金鑰也不以沒加密的 http 送到別台電腦（設了 `allowPlainHttpKey` 的除外，config.md 的「接別的模型服務」）。
- 送出前，已知格式的金鑰（`sk-ant-`、`sk-proj-`、`AIza`、`AQ.`、`ghp_`、`AKIA`…）會被遮掉。這是最後一道防線，機密檔本來就讀不到。

## 存在哪

都在 `~/.xing-janus/`（設了環境變數 `XING_JANUS_HOME` 的話，在它指的資料夾；資料夾權限 700、檔案權限 600）：

| 檔案 | 內容 | 保留 |
|---|---|---|
| `config.json`、`prices.json`、`checks.json` | 設定、價格表、事後檢查 | 一直 |
| `trusted.json`、`accepted.json` | 信任的資料夾、同意過的使用提醒 | 一直 |
| `spend.jsonl` | 每次請求的花費（不含內容） | 一直 |
| `keys/` | 金鑰檔：`xj setup`、`/model` 存不進鑰匙圈、認證管理員時（Linux、WSL 沒有鑰匙圈）存的金鑰 | 一直 |
| `skills/` | 你自己的技能 | 一直 |
| `models-*.json` | 各家（同一家好幾把 key 的話是各把）的模型清單快取 | 一直（超過一天就重新查） |
| `acl-fix.json` | Windows：沙盒的帳號寫得進 PATH 上的哪些地方、要擋哪裡（見 usage.md 的「全部不問模式」） | 每次開沙盒時更新 |
| `sessions/` | 對話紀錄（金鑰已遮掉）、子代理的完整過程（`subagents/`，金鑰已遮掉）、改檔前的快照（原封不動，給 `/undo` 用）、完整輸出、你附加的圖片、`--debug` 的紀錄 | 30 天（`sessions.retentionDays`） |

對話紀錄裡有它讀過的檔案內容，所以不要把 `~/.xing-janus/` 放進會同步到雲端的資料夾（`xj doctor` 會檢查）。

旁邊的 `~/.xing-janus-cache/`（設了 `XING_JANUS_HOME` 的話，是那個路徑後面加 `-cache`）是沙盒裡的指令用的：npm 的快取、沙盒裡的 git 用的設定。沙盒寫得到，所以金鑰和紀錄不放這裡。
Windows 另外有 `%LOCALAPPDATA%\xing-janus\`：開沙盒時替上層資料夾加了哪些權限的記錄（拿掉的時候用，見 troubleshooting.md 的「Windows」）。

## 怎麼刪

- 舊的對話紀錄：`xj sessions prune`
- 全部：`xj reset`（會先列出來、問過再刪；`~/.xing-janus-cache/` 也一起刪）
- 鑰匙圈裡的金鑰：`security delete-generic-password -s xing-janus-<廠商>`
- Windows 認證管理員裡的金鑰：`cmdkey /delete:xing-janus-<廠商>`
- 同一家好幾把 key 的話，有名字的那幾把是另外的項目（`xing-janus-<廠商>--<名字>`）：上面兩行刪不到，要照名字各刪一次

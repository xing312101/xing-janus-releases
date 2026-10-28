# 資料在哪、送到哪、怎麼刪

## 送到哪

- 你打的字、它讀到的檔案內容、指令的輸出，會送到你選的模型服務（Anthropic、OpenAI、Gemini，或本機的 Ollama），適用該服務的條款。
- Xing Janus 本身不收集任何資料：沒有遙測、沒有錯誤回報、不自動檢查更新。對外的連線只有模型的 API，和你允許的網域。
- 抓網頁（`web_fetch`）：那個網站看得到整個網址、你的 IP，和 `xing-janus/版本` 這個 User-Agent；不送 cookie、不送登入資訊。
  抓到的內容跟讀到的檔案一樣，會送給模型、存進對話紀錄。網址不會另外送去別的地方檢查。
- 子代理：用的模型跟主對話一樣送到同一家；你指定別家的模型的話，交代的內容和它讀到的東西送到那一家。
- 送出前，已知格式的金鑰（`sk-ant-`、`sk-proj-`、`AIza`、`ghp_`、`AKIA`…）會被遮掉。這是最後一道防線，機密檔本來就讀不到。

## 存在哪

都在 `~/.xing-janus/`（資料夾權限 700、檔案權限 600）：

| 檔案 | 內容 | 保留 |
|---|---|---|
| `config.json`、`prices.json`、`checks.json` | 設定、價格表、事後檢查 | 一直 |
| `trusted.json`、`accepted.json` | 信任的資料夾、同意過的使用提醒 | 一直 |
| `spend.jsonl` | 每次請求的花費（不含內容） | 一直 |
| `sessions/` | 對話紀錄（金鑰已遮掉）、子代理的完整過程（`subagents/`，金鑰已遮掉）、改檔前的快照（原封不動，給 `/undo` 用）、完整輸出、你附加的圖片、`--debug` 的紀錄 | 30 天（`sessions.retentionDays`） |

對話紀錄裡有它讀過的檔案內容，所以不要把 `~/.xing-janus/` 放進會同步到雲端的資料夾（`xj doctor` 會檢查）。

## 怎麼刪

- 舊的對話紀錄：`xj sessions prune`
- 全部：`xj reset`（會先列出來、問過再刪）
- 鑰匙圈裡的金鑰：`security delete-generic-password -s xing-janus-<廠商>`

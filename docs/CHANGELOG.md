# 更新說明

版本號照 semver。

## 0.1.5

**需要**：跟 0.1.4 一樣 —— macOS、Linux 或 Windows，Node 24 以上（Windows 建議 24.20.0 以上）、`ripgrep`；Linux 另外要 bubblewrap、socat 和 sandbox-runtime 0.0.79，
Windows 要 Git for Windows（見 `docs/install.md`）。

**從 0.1.4 更新**：打 `xj update`（Homebrew 裝的用 `brew upgrade xj`）。設定檔、金鑰、對話紀錄都不用動。有幾件跟以前不一樣：
- `--dangerously-skip-permissions` 只換成全部不問，不再順便關掉沙盒。沙盒起不來的地方（例如容器裡）要再加 `--no-sandbox`，或在那台的設定檔寫 `"sandbox": { "enabled": false }`。
- 沙盒關掉時，讀檔、改檔、搜尋的工具也碰得到工作目錄以外的地方（以前一律擋）：讀之前先問、改之前先秀 diff 再問；機密檔、放憑證的地方、xj 自己的資料夾照樣一律擋（下面的「範圍跟著沙盒走」）。
- 狀態列整列標紅，改成只有「全部不問而且沒有沙盒」的時候（以前沒有沙盒、或全部不問就標紅）。
- 設定檔自己寫了 `statusBar.fields` 的：新的「資料夾」（`folders`）、「背景」（`jobs`）、這一輪的花費（`turn-cost`）要自己加進去。
- 每一輪送給模型的工具多了幾個（`skill`、`settings_propose`，和背景跑的 `job_output`、`job_stop`），說明變長，花費會多一點（評測裡同樣的題目多了大約 4～7%）。

**下載之後先驗**（兩個檔案跟壓縮檔一起放）：

```bash
shasum -a 256 -c xj-0.1.5-darwin.tar.gz.sha256
minisign -Vm xj-0.1.5-darwin.tar.gz -P <你留的那份公鑰>
```

Linux 是 `xj-0.1.5-linux-x64.tar.gz` 或 `xj-0.1.5-linux-arm64.tar.gz`（SHA-256 用 `sha256sum -c` 驗），Windows 是 `xj-0.1.5-windows.zip`
（照 `docs/install.md`「Windows 的安裝」第 1、2 步驗，用你留的那份公鑰）。驗完看 minisign 印的 `Trusted comment:`，
要是 `xing-janus v0.1.5 xj-0.1.5-<平台>…`，不是的話不要裝（舊版連同它的簽章換個檔名，前面兩行照樣驗得過）。
第一次用的話，照 `docs/install.md` 裝好、跑 `xj setup`。

新功能：

- **沙盒和模式分開**：沙盒開不開看設定檔的 `sandbox.enabled`（預設開）和指令列的 `--sandbox`、`--no-sandbox`，模式只管哪些事要先問你。
  全自動沒有沙盒也進得去（改檔不問、指令每個都問），`--auto --no-sandbox` 可以一起用。
  全部不問、沙盒開著時：指令照樣關在沙盒裡，沒允許過的網域（指令和 `web_fetch`）不問就擋，`excludedCommands` 列的直接在沙盒外跑，沙盒裡跑不了的工具不問就擋、講怎麼辦，MCP server 照樣在沙盒裡。
  README 多一段「沙盒和模式」，使用說明的模式表換成沙盒開著、關掉的四張表，新的一份「該怎麼設」（[setup-guide.md](setup-guide.md)：照情況寫設定檔的那幾行）
- 設定檔的 `allowFullAuto`、`allowBypass`：在 xj 裡用 Shift-Tab、`/mode` 也切得到全自動、全部不問（`--auto`、`--dangerously-skip-permissions`、已經寫了的 `defaultMode: "full-auto"` 照舊）；
  `defaultMode` 可以寫 `"bypass"`（要 `allowBypass`）。全部不問每次開 xj 第一次進去要打字確認；Shift-Tab 跳過沒開放的模式時提示一次，`/mode` 列出切不到的和原因
- 全部不問時，刪掉重要的地方（根目錄、根目錄下第一層、家目錄、工作目錄和它的上層、加進來的資料夾）和你自己加進 `permissions.danger` 的照樣問
  （設定檔的 `permissions.bypassAsksCriticalDelete`，預設 `true`）
- **在 xj 裡關掉、打開沙盒**：`/sandbox off`（要按 `y`，Enter 不算；詢問講關了會怎樣）、`/sandbox on`（開得起來才換，起不來講原因、照舊）、`/sandbox` 看狀態；全螢幕點右邊那一欄的「沙盒」也一樣。
  只管這次對話，模型在跑的時候不換；換了之後對話裡記一行、模型也知道；`/add-dir` 加的資料夾、允許過的網域帶到新的沙盒。
  Windows：開場沒有沙盒時用的 PowerShell，沙盒的帳號跑不了的話，打開之後改用跑得了的那一個
- 狀態列、右邊那一欄寫沙盒是哪裡關的（「關（設定檔）」「關（指令列）」「關（/sandbox）」）。右邊那一欄的模式、沙盒底下多一行灰字（在現在的沙盒下實際怎麼做、碰得到哪裡）；
  順序改成模式、沙盒、信任資料夾、改檔都允許、永久允許、事後檢查、Enter＝是、提示音；常用指令加了 `/sandbox`、`/add-dir`（`/tui` 拿掉了，`/help` 裡照樣有）；矮的視窗先拿掉每一項底下那一行，待辦清單照樣看得到
- **加進來的資料夾也能改**：`/add-dir <路徑>` 會問「也要能改嗎？」（要按 `y`）。能改的跟工作目錄同一套規矩（改檔照模式問、`/undo` 還原得了、沙盒裡的指令也寫得到、機密檔和受保護的檔案照樣擋）；
  要在電腦前面信任過，不能是家目錄、它的上層、根目錄、放憑證的資料夾，xj 在沙盒外跑的程式、PATH 上的資料夾、xj 自己的程式在裡面的也不行；有自己的 `AGENTS.md`、`CLAUDE.md` 的會講。只算這次對話。
  `/add-dir` 不帶參數列出這次加的，`/add-dir remove <編號>` 拿掉；`/resume` 接續時，那時加過的照現在的規矩再加一次
- 開 xj 時就加：指令列的 `--add-dir <路徑>`（也能改，可以好幾個）、設定檔的 `permissions.addDirs`（照開 xj 的資料夾列，每一個標只能讀或也能改），換對話也在。
  `xj -p` 也加，但不問信任（沒信任過的只能讀）；`xj mcp` 不收。狀態列多一欄「資料夾」，右邊那一欄多一項「資料夾」（點了列出；沙盒關掉時寫「不限」）
- 事後檢查改到哪個資料夾就跑那裡的：只改了加進來的資料夾的話，跑那個資料夾自己的檢查（第一次跑先問），工作目錄的不跑
- **範圍跟著沙盒走**：沙盒開著時，讀檔、改檔、搜尋的工具跟沙盒同一個範圍（跟以前一樣）；沙盒關掉時沒有範圍：工作目錄以外的讀先問、改檔先秀 diff 再問、標紅
  （自動改檔、全自動也問；全部不問不問），`/undo` 還原得了。照樣一律擋機密檔、放憑證的地方（`~/.ssh`、`~/.aws`、`~/.gitconfig`…）、xj 自己的資料夾；
  改的話另外擋受保護的檔案（任何一層的 `.git`、`.vscode`…）和之後會自動執行的地方（shell 的啟動檔、開機和登入時自動跑的、排程、`/etc` 這類；
  Windows 上 PowerShell 的設定檔 `$PROFILE`，「文件」被轉到 OneDrive 或公司的資料夾重新導向的話照實際的位置）
- **`/permissions` 看得懂**：先講現在的狀態和怎麼換，再分組列出你永久允許的（有編號、拿得掉）、設定檔改了的、內建的（白名單整份照用途分組、危險清單、機密檔的樣式）；
  全螢幕模式列完接著開選單（↑↓ 選、Enter 看細節、按 `y` 才拿掉），右邊那一欄點「永久允許」直接開選單
- **預算看得到、改得了**：`/budget` 列出三個上限、花了多少、怎麼改；`/budget day 50` 這樣改（Enter 只改這次對話，寫進設定檔要再按 `y`）。
  狀態列的今天寫上限，模型在跑的時候多一欄這一輪的；右邊那一欄多一項「今天的預算」（到 80% 黃、到上限紅、點了就是 `/budget`）；到 80%、到上限的那一句講怎麼調高；`/cost` 也列上限
- **指令能跑更久、能放到背景跑**：設定檔的 `commands`（前景的預設 `timeoutSec`、最長 `maxTimeoutSec`，跟以前一樣 2 分鐘、10 分鐘；背景的最長 `backgroundMaxMinutes`，預設 60 分鐘、0 是不限），
  逾時的那一行講是哪一欄、現在是多少。模型可以把開發伺服器這類放到背景跑，看輸出、等它跑完、停掉；照樣經過同一套詢問（問你的時候會講這是放到背景跑的、最多跑多久）、在沙盒裡跑，同時最多 8 個；一個最多印 5 GB（超過就停掉、講原因，輸出檔只留開頭和最後一段）。
  `/jobs` 列出、看輸出、停掉；狀態列的「背景 N」，全螢幕右邊那一欄列出在跑的；跑完時對話裡記一行（有提示音）；換沙盒、`/exit` 時還有在跑的先問。`xj -p`、`xj mcp`、子代理不能用。
  Windows 的沙盒要重新擋檔案的時候（機密檔被換成另一個檔案、拿掉了資料夾、加了也能改的資料夾、也能改的資料夾本身多了新的機密檔），背景還有指令在跑的話下一個指令不會跑、講要先停掉哪幾個
  （重新擋的那一下，還在跑的指令讀得到機密檔、改得到受保護的檔案）
- **`/config`**：常用的設定列出現在的值，選一項就改、講改了會怎樣（開 xj 時開沙盒、預設模式、允許切到全自動、預算、指令的時間上限、介面語言、提示音、Enter＝是、畫面、一起加的資料夾；
  允許切到全部不問只能看）。放寬的要按 `y`；只改那一欄，寫之前整份檢查過。預算、介面語言、提示音、Enter＝是、畫面這次對話也馬上換
- **問 xj 怎麼設定**：直接問（「這個專案要用 docker，該怎麼設？」），它照你現在的設定回答（內建的技能 `xj-settings`，讀的時候附上現在的設定，金鑰這類遮掉）。
  它也可以提議改設定：每一項標放寬、收緊或不影響權限，先問你（放寬的標紅、要按 `y`），你答應了 xj 才寫進設定檔；模式、允許切到全自動、預算、白名單、網域這次對話也馬上生效。
  全部不問、拿掉內建的保護、MCP server、`xj mcp` 的設定、金鑰和服務、環境變數它不能提；不信任的資料夾不收放寬的；`xj -p`、`xj mcp`、子代理不能用
- `xj doctor` 多一列「設定檔放寬的」：關掉沙盒、`allowFullAuto`、`allowBypass`、寬鬆的 `defaultMode`、`bypassAsksCriticalDelete: false`、`allowedDomains` 有 `"*"`、
  拿掉的內建危險規則和機密檔、`gitconfig: "full"`、`allowLocalBinding`、`writablePrograms: "allow"`
- `xj setup`：macOS 存金鑰時講清楚 `security` 問的「password data for new item:」是 macOS 自己的字（金鑰照舊由 `security` 直接收、不經過 xj）；
  第 3 步列出的模型一行一個、寫成「廠商:模型」，最多 15 個，再多的講還有幾個、在 xj 裡打 `/model` 看全部；最後講一句預設的預算和 `/budget`
- 使用者文件：`config.md` 的危險清單照程式補齊（每一項認得的指令、PowerShell 那一份），寫清楚 `sandbox.allowedDomains` 的寫法（連接埠、`*.github.com`、`"*"` 是任何網域，本機、內網、metadata 照樣擋）；
  `install.md` 的 Homebrew 那一段講 brew 會連帶裝 node 的依賴、升級舊版的套件

修正：

- **Windows 的沙盒裡可以把整個 `.git`（或其他受保護的檔案所在的資料夾）改名、換成新的**，`.git/config`、`.git/hooks` 的擋就繞過了
  （之後你自己跑 git 的時候會照新的設定執行程式；xj 自己跑的 git 不受影響）。受保護的檔案所在的資料夾開著沙盒時加「沙盒不能刪除」，
  沙盒的帳號和群組各一條、開放資料夾之前就加（srt-win 改權限時會拿掉帳號那一條，另一個 xj 做的也算；群組那一條它不碰）；
  加不上、或沙盒的群組不在的話白名單不生效、每個指令都問。macOS、Linux 沒有這個問題
- **全部不問時，`bash -c '…'`、`sh -c '…'`、`eval '…'` 裡面寫到機密檔的指令照樣跑**（`bash -c 'cat .env'`），內容交給模型：
  危險清單、「刪掉重要的地方」都往這種字串裡面看，擋機密檔的那一段沒有（0.1.4 以前全部不問一律沒有沙盒；這一版起沙盒開著的話作業系統那一層照樣擋）。
  改成同一份拆法。現在檔名原樣寫在指令裡的都擋：不管哪一種引號（`cat $'.env'`）、大括號（`cat {.env,README.md}`、`{cat,.env}`）、
  每一層的重新導向（`echo "$(< .env)"`）、`dd if=.env`、包在 `bash -c`／`sh -c`／`eval` 裡（一層包一層、`find -exec sh -c '…'`、沒列到的包裝指令後面的）。
  要算出來的（變數、萬用字元、別的程式的程式碼）照樣擋不到，要擋住靠沙盒（見 cerberus.md）
- **指令的幾種寫法，危險清單、「刪掉重要的地方」、機密檔的檢查都看不到**：
  雙引號裡以 `$` 結尾的字串後面、到下一個雙引號為止的那一段（`grep "x$" f; rm -rf ~; echo "ok"`：中間的被當成字串的一部分）；
  `-c` 後面還有選項或 `--` 的（`bash -c -- 'rm -rf ~'`、`bash -ceo pipefail '…'`）、`bash +c '…'`；`$( )` 裡有 `case` 的。都修了。
  一層包一層包得太深、大括號展開太多而看不完的指令，以前當成沒事，現在會問（全部不問、全自動也問）。
  一層包一層的長指令以前要看很久（幾十 KB 的十幾秒，xj 那段時間沒有反應），現在一秒內
- **Windows：指令裡夾著 `\r`（CR）時，危險清單、「刪掉重要的地方」、機密檔的檢查看到的跟 Git Bash 實際跑的不一樣**：
  Git Bash 執行前把指令裡的每一個 `\r` 拿掉（引號裡的、`eval`、`bash -c` 裡的也是），xj 以前把它當成空白（`cat .e\rnv` 實際讀的是 `.env`；
  `true\r#x; cat .env` 的後半段被當成註解）。現在跟 Git Bash 一樣先拿掉再看；有 `\r` 的指令照樣不算白名單
- **Windows：全部不問、沒有沙盒時，從 Git Bash 叫 `cmd`、PowerShell 讀機密檔的指令照樣跑**（`cmd //c "type .env"`、`powershell -c "Get-Content .env"`），內容交給模型：
  引號裡那一串 bash 只當成一個字。現在照 PowerShell 工具的規則看裡面：機密檔、受保護的檔案擋，刪掉重要的地方要問；cmd 的 `^` 跳脫拿掉再看。
  PowerShell 工具（和這裡）也認得夾在字中間的引號：PowerShell 會把 `type .e'n'v` 接成 `.env`，以前照引號切開、看不出來
- **夾在一般檔案裡的多行私鑰只遮到第一行**：`read`（加了行號）、`search`（一行一行的結果）把私鑰的本體原樣交給模型。私鑰放在機密檔（`*.pem`、`id_rsa`…）的話本來就整個擋；
  出事的是夾在設定檔、文件、測試資料裡的。現在整段遮：`read` 每一行各自換掉、行號照舊（從私鑰中間開始讀也認得）；`search` 的結果裡單獨一行、整行像私鑰本體的也換掉。
  指令的輸出另外補上幾種以前遮不到的：有密碼的舊格式（`Proc-Type`、`DEK-Info` 之後空一行才是本體）、PGP 的私鑰、寫成一行的（換行寫成 `\n`）、本體沒折行的、
  每一行前面多了字的（diff 的 `> `、`grep -n` 的路徑和行號、log 的前綴）、只印出後半段和 END 的（`tail`、diff 的前後文）。
  BEGIN 後面接著十幾萬行 base64 的輸出以前整個交不出去（工具回錯誤），現在照樣回得來
- 自訂的機密檔樣式裡的 `[!…]`（「不是這幾個字」）被當成一般的字：`key[!s].json` 反而擋到 `keys.json`、沒擋 `keya.json`。現在 `[!…]`、`[^…]` 都是「不是」（內建的清單沒有用到）
- 貼上的文字看起來像斜線指令時，確認的問題只顯示第一行，按「是」之後卻是整段交給那個指令（`/checks add` 會把後面整段存成一個檢查）：現在整段看得到，控制碼和看不見的字標出來。
  詢問裡的危險說明、開場那一行技能資料夾的名字也標出方向控制字元、換行
- Windows：找沙盒外的指令（`sandbox.excludedCommands`）時，Microsoft Store 的執行別名照找 PowerShell 的同一份檢查（`..` 先算掉、指到的要真的是檔案），以前只比目標路徑的開頭
- 換對話（`/clear`、`/resume`）之後，技能的資料夾在沙盒裡讀不到（macOS、Linux）；Windows 有技能時，開沙盒之後第一個指令要整個重新初始化一次（大的工具鏈要好幾分鐘）→ 開的時候就一起給
- Windows：`/add-dir` 之後不再把沙盒整個重新準備一次（大的工具鏈要好幾分鐘）：只加新的那個資料夾（先擋那裡的機密檔、受保護的檔案，再給權限），
  開 xj 時一起加的也一樣；拿掉資料夾時才整個重來。加資料夾時開著的 MCP server 也不再重開
- 給模型的模式說明照現在的沙盒講：以前全部不問一律說「沙盒也關了」（沙盒開著時是錯的），全部不問又沒有沙盒時開場的說明自相矛盾（又說每個指令都要問）。
  Windows 的 PowerShell 工具說明不再照沙盒開不開寫（工具說明整個對話固定）
- 換對話時，同一個資料夾也寫在設定檔的 `extraReadable`、又用 `/add-dir` 加成也能改的話，連設定檔的那一份讀的範圍也一起收掉了
- 每月的上限設成 0（不設上限）時，狀態列、`/cost` 寫成「/$0.00」（今天、這一輪的本來就不寫）
- 跑了多久每個地方同一種寫法（「1 分 33 秒」，超過一小時「1 小時 2 分」）：狀態列上在跑的指令、等模型的時間以前只寫秒（「125 秒」），英文介面的這一輪寫成「1m 20s」
- **Anthropic 伺服器端 fallback 時，上下文算成兩倍**：一次回應有兩筆用量、每一筆都含整個對話，以前加起來算，提早自動摘要（多一次請求、丟掉細節）、子代理提早收尾。
  現在照最後一筆算（花費照樣每一筆都算）。`/resume` 接續時，最後一筆用量是子代理的話，以前拿它當主對話的上下文
- `/effort` 之後換模型（同一家）：送出去的還是之前切換的那個，換到沒有推理強度的模型也照樣送。現在照新的模型選（狀態列寫的那個），不支援的不送
- `/sandbox` 換沙盒時，剛停掉、程序還沒收完的背景指令以前不算：停掉之後馬上換，那些指令結束時的收尾打到已經換掉的沙盒上。現在先等它們真的結束（最多幾秒）
- 切在一個字（emoji、中文）的中間：`job_output` 給模型的最後一段、讀技能時附的設定檔、推理的進度、srt-win 的錯誤訊息可能留下半個 emoji（送給模型的話有的服務整個請求都不收）；
  太長的輸出只留開頭和結尾時，切在中文字中間會留下「�」。背景的指令一直不換行地印的時候，以前每 64 KB 照位元組切一刀：那裡的字變成「�」，Windows 上那一段整個照主控台的字碼頁解成亂碼
- `search` 到了時間上限（60 秒）、結果超過 20 MB 時會停掉，交給模型的以前看起來像全部的結果（什麼都還沒找到的還可能說「找不到」）：現在講清楚只有一部分、怎麼縮小範圍

## 0.1.4

**需要**：跟 0.1.3 一樣 —— macOS、Linux 或 Windows，Node 24 以上（Windows 建議 24.20.0 以上）、`ripgrep`；Linux 另外要 bubblewrap、socat 和 sandbox-runtime 0.0.79，
Windows 要 Git for Windows（見 `docs/install.md`）。

**從 0.1.3 更新**：打 `xj update`（Homebrew 裝的用 `brew upgrade xj`）。設定檔、金鑰、對話紀錄都不用動。
- `xj setup` 第一題的編號改了：3 是 Gemini、4 是本機或其他 OpenAI 相容的服務（0.1.3 的 3 是 Ollama）。
- 用沒加密的 `http://` 接別台電腦上的服務、又要金鑰的：這一版起金鑰不送（會以明文送出）。改用 https；確定要送的話，在設定檔的 `providers.<名稱>`
  加上 `"allowPlainHttpKey": true`（見 `docs/config.md` 的「接別的模型服務」）。
- 你的 `~/.xing-janus/prices.json` 是舊版建的話沒有 Gemini 的價格（花費顯示「未知」）：`xj doctor` 會講怎麼換成新的。

還在 0.1.1、0.1.2 的話，先看下面 0.1.3 那一段（換版是舊版的程式在做，有幾件要避開）。

**下載之後先驗**（兩個檔案跟壓縮檔一起放）：

```bash
shasum -a 256 -c xj-0.1.4-darwin.tar.gz.sha256
minisign -Vm xj-0.1.4-darwin.tar.gz -P <你留的那份公鑰>
```

Linux 是 `xj-0.1.4-linux-x64.tar.gz` 或 `xj-0.1.4-linux-arm64.tar.gz`（SHA-256 用 `sha256sum -c` 驗），Windows 是 `xj-0.1.4-windows.zip`
（照 `docs/install.md`「Windows 的安裝」第 1、2 步驗，用你留的那份公鑰）。驗完看 minisign 印的 `Trusted comment:`，
要是 `xing-janus v0.1.4 xj-0.1.4-<平台>…`，不是的話不要裝（舊版連同它的簽章換個檔名，前面兩行照樣驗得過）。
第一次用的話，照 `docs/install.md` 裝好、跑 `xj setup`。

新功能：

- `xj mcp`：讓別的程式（Claude Desktop、IDE、你自己寫的通訊軟體橋接）透過 MCP 叫 xj 做事（[mcp-server.md](mcp-server.md)）。能做什麼照設定檔的 `mcpServer`：
  預設唯讀、沙盒外的動作一律不行、只有啟動它的那個資料夾、花費每輪 $2／每天 $10（到了就停）；放寬的話要核准的用 MCP 的 elicitation 問人（問題先遮金鑰，只給「允許一次」「拒絕」）。
  一律要沙盒、資料夾要在電腦前面信任過；交來的字裡的 `@路徑` 不附加檔案
- Gemini 有原生的轉接層（以前走 Google 的 OpenAI 相容端點）：`xj setup` 選 3，或設環境變數 `XING_JANUS_GEMINI_API_KEY`；`AQ.` 開頭的新金鑰也能用，
  對話紀錄裡會遮掉。思考的內容原樣帶回，連續好幾輪的工具呼叫不會壞。設定檔的 `providers.gemini` 寫了 `baseURL` 的照舊走相容端點。
  免費額度的金鑰，Google 會拿送出去的內容改進產品（見 `docs/config.md` 的「Gemini」）
- `xj setup` 第一題的第 4 項：本機或其他 OpenAI 相容的服務（Ollama、LM Studio、llama.cpp、vLLM、OpenRouter…）。先列出 Ollama 和這台找到的本機伺服器，
  最後一項自己填網址（也可以直接貼），再問名字、要不要金鑰：只用本機模型的人在設定精靈就能設好（以前要先有 Anthropic、OpenAI 的金鑰才進得了 xj、才用得到 `/model`）。
  按 Enter 是 Ollama；找到的其他伺服器要打編號才選得到（多人共用的電腦上，別人的程式可能佔住那些埠）
- 價格表：限期的優惠價（`until`、`after`），照日期自己換成之後的價格；預設的價格表加了主要的 Gemini 模型
- 每一輪的工具呼叫上限，預設從 50 調到 100（設定檔的 `limits.toolCallsPerTurn`）

修正：

- 安全：金鑰不以沒加密的 http 送到別台電腦。以前只有 `xj setup`、`/model` 這次要存的新金鑰不送，同一個名字原本就有的（鑰匙圈、金鑰檔、環境變數）
  照樣以明文送到新的網址（例如重新設定時網址少打一個 s）。開 xj、在 xj 裡換過去、當場設金鑰、`/key` 換一把、`xj doctor` 都不送（設定見上面「從 0.1.3 更新」）
- 安全：`xj setup`、`/model` 新增服務時，網址照設定檔的寫法檢查：`HTTP://`、`http:/`、`http:\\` 這類寫法以前會先把金鑰以明文送出去，最後才被設定檔擋下來；
  網址裡帶帳號、密碼的不收
- 安全：平台的名字不能有 `--`（`anthropic--work` 會蓋掉 Anthropic 叫 work 的那一把金鑰）、不能是 `prototype`、`con` 這類保留的名字；
  `xj setup` 存金鑰之前先驗整份設定，寫設定檔時先寫到暫存檔、驗過才換上去（以前寫壞了之後每次開 xj 都講設定檔有錯）
- 安全：Gemini 關掉 SDK 的自動接續（服務回了接續的 token 時，SDK 會帶著整段對話自己再送、每次都重新計費，預算擋不住；斷線也自己再試）：回了 token 的當成被截斷。
  工具的參數跟另外兩家一樣先過檢查（`__proto__` 這類鍵拿掉）
- 安全：OpenAI 相容的服務（OpenAI、Ollama、OpenRouter…）也不跟轉址：以前跨網域的轉址時，整段對話照樣送到轉址的地方（金鑰不會）。
  Gemini、OpenAI 相容的服務回了轉址時，訊息跟 Anthropic 一樣講是轉址（以前說「連不到」）
- 安全：`xj doctor` 的每一列、開場第一行，把服務回的字、資料夾名稱裡的換行收成空格（以前可以偽造出整列，例如假的「沙盒：擋得住」）；
  開場第一行和「這個資料夾」那一列標出資料夾名稱裡的方向控制字元（以前原樣印：同一行後面的字會反過來，Windows Terminal 右邊那一欄會被推走）
- 安全：`xj setup` 第 3 步連不到時的原因先遮掉金鑰（服務把收到的 Authorization 放進錯誤訊息的話，以前整把印在畫面上）；列模型最多等 15 秒（以前服務不回就一直停著）
- 安全：`.localhost` 結尾的網址不算「在這台」（Node 照 DNS 解析）：`/model` 換過去之前照樣先問
- Windows：沙盒偶爾起不來（「could not bind a loopback listener outside the WFP permit range」：沙盒程式庫找臨時埠只試 5 次）→ xj 自己重開，最多 3 次
- Windows：Microsoft Store 版的 PowerShell 7 找得到了（沒開沙盒時用它；沙盒開著時照舊用 Program Files 的 7 或內建的 5.1）；跳過 PowerShell 7 時的說明改正。
  它開的程式不在 xj 的 job object 裡（Store 的程式准許子程序脫離 Windows 給它的 job）：Ctrl-C、逾時時照父程序收，指令結束之後留下的程式收不到（cerberus.md）。
  執行別名只認指到 `%ProgramFiles%\WindowsApps` 的（安全：沙盒裡的指令在工作目錄建一個別名的話，之後沒有沙盒時 xj 會選它）
- 價格表：限期的優惠價，一般的有長上下文、`after` 沒寫的那一筆不用（以前過了 `until` 長上下文的加價整個不見，花費算少）
- `xj setup`：建議不出模型時（連不上、清單是空的）不給「ollama:」這種預設值（設定檔不收，按 Enter 就整個失敗）；打的不是「廠商:模型」就講清楚再問，三次都不行就不存。
  連不到時的訊息不再多一個句號（「Ollama 有開嗎？。」）
- 英文的 `xj doctor`：一列接兩句話時，後一句的開頭改小寫（以前印成「v0.1.3; The last…」）
- 一般模式把視窗縮窄時，舊的狀態列被終端機折成兩行，上半截一直留在畫面上（之後印的字也蓋不掉）、放寬時又被接回來推到中間：清掉了。
  只在認得出會這樣重折的終端機上清（Windows Terminal、tmux、Terminal.app、iTerm2、VS Code、GNOME Terminal 這類用 VTE 的、kitty、WezTerm、Ghostty、Alacritty、foot）；
  舊的主控台（conhost）、xterm 這類不會重折的，和 SSH 連進來看不出是哪一種的照舊（在不會重折的終端機上清，會刪到對話的最後幾行）
- `/model` 換了之後連平台一起講（「改用 Claude Sonnet 5.5（anthropic，系列 sonnet）」），`/resume` 的清單寫 `平台:模型`、接續時也講平台：不同平台有同名的模型時看得出是哪一個
- 設定檔的 `budget.unknownRequestsPerTurn`：一輪裡幾個查不到價格的請求就停下來問（預設照舊 10；填 0 是不問，例如用本機的服務時）。停下來問的那一句也講怎麼關
- 在 xj 裡打字時的說明折行時，句尾的標點（「）。」這類）不折到下一列的開頭、開頭的括號不留在上一列的結尾
- 使用者文件：README、快速開始、`config.md`（Gemini、限期的優惠價、`xj setup` 的第 4 項、`allowPlainHttpKey`、平台的名字）、使用說明、隱私、疑難排解；
  `install.md` 註明 arm64 的 Windows 還沒實際測過，Windows 的安裝、移除改 PATH 的那幾行照原本的寫法改（`%USERPROFILE%` 這類寫法和值的種類 `REG_EXPAND_SZ` 不變；以前的寫法會換成 `REG_SZ`）

## 0.1.3

**需要**：跟 0.1.2 一樣 —— macOS、Linux 或 Windows，Node 24 以上（Windows 建議 24.20.0 以上）、`ripgrep`；Linux 另外要 bubblewrap、socat 和 sandbox-runtime 0.0.79，
Windows 要 Git for Windows（見 `docs/install.md`）。

**從 0.1.1、0.1.2 更新**：打 `xj update`（Homebrew 裝的用 `brew upgrade xj`）。換版是舊版的程式在做，下面修的這幾件這一次還是會碰到：
- macOS、Linux 手動裝的：**不要在家目錄打**，換到別的資料夾（例如你的專案裡）再打。0.1.1、0.1.2 的 `xj update` 在家目錄打的話，新版會裝好，
  `xj` 的連結卻不會換過去；已經打了的話，照它印出來的那一行 `ln -sf …` 做就好。
- Windows：**不要在 xj 的安裝資料夾**（`%LOCALAPPDATA%\Programs\xj`）**裡打**：在那裡打的話，新版解好了卻換不上去，也不會講。
  你留的公鑰（`~/.xing-janus/minisign.pub`）是用 PowerShell 的 `>` 存的話，舊版會說它「不是 minisign 的公鑰」、不更新：
  從你原本那一份轉存（不要從發佈頁面重抄）：`$f = "$env:USERPROFILE\.xing-janus\minisign.pub"; (Get-Content $f) | Set-Content $f`。

還在 0.1.0 的話，照 `docs/install.md`「更新」那一節的最後一段手動換一次。
設定檔、金鑰、對話紀錄都不用動。

**下載之後先驗**（兩個檔案跟壓縮檔一起放）：

```bash
shasum -a 256 -c xj-0.1.3-darwin.tar.gz.sha256
minisign -Vm xj-0.1.3-darwin.tar.gz -P <你留的那份公鑰>
```

Linux 是 `xj-0.1.3-linux-x64.tar.gz` 或 `xj-0.1.3-linux-arm64.tar.gz`（SHA-256 用 `sha256sum -c` 驗），Windows 是 `xj-0.1.3-windows.zip`
（照 `docs/install.md`「Windows 的安裝」第 1、2 步驗，用你留的那份公鑰）。驗完看 minisign 印的 `Trusted comment:`，
要是 `xing-janus v0.1.3 xj-0.1.3-<平台>…`，不是的話不要裝（舊版連同它的簽章換個檔名，前面兩行照樣驗得過）。
第一次用的話，照 `docs/install.md` 裝好、跑 `xj setup`。

修正：

- `xj update`（macOS、Linux）：在家目錄（或 `/`）打的時候，PATH 上指到舊版的 `xj` 連結也會換到新版（以前說「PATH 上找不到指到這一版的 xj 連結」，`xj` 還是舊版；
  在 `/` 打的話連解壓都會失敗）。換版是舊版的程式在做：從這一版再換到之後的版本才有效
- 安全：`xj update` 解壓的時候不照你的 PATH 找程式（Linux 的 tar 照 PATH 找 gzip：PATH 上有沙盒寫得到的地方的話，
  例如啟用了 venv 的專案，那裡被放了假的 gzip 就會在沙盒外跑，還能換掉裝上去的內容）
- 安全：`xj setup` 只拿設定檔收得下的模型名字當建議的預設值（以前模型服務回的第一個名字原樣印在提示裡：惡意的服務可以改剪貼簿、清畫面）；
  設定檔先檢查過才寫（以前先寫，寫壞了之後每次開 xj 都講設定檔有錯）
- 安全：PuTTY 的私鑰（`*.ppk`）、存在硬體金鑰上的 SSH 金鑰（`id_ed25519_sk`、`id_ecdsa_sk` 和它們的 `.pub`）也算機密檔
- 安全：開沙盒之前，沙盒外的 PATH 也拿掉沙盒寫得到的地方：沙盒程式庫照 PATH 找 `which`，你在 `sandbox.writable` 加的資料夾剛好也在 PATH 上的話，
  沙盒裡的指令在那裡放的 `which` 會在沙盒外跑（預設開放寫入的 `~/.pub-cache`、`~/.gradle` 裡會被放上 PATH 的位置本來就擋寫入，不受影響）
- 安全：要你回答是／否的問題（信任這個資料夾嗎、貼上的斜線指令、換平台）跟核准畫面一樣，把控制碼和看不見的字（方向控制、零寬）標出來：
  資料夾名稱裡的方向控制字元會把同一行後面的字反過來。畫面上其他單行的名字（模型、檔名、服務回的錯誤）也標出方向控制字元
- `xj update`（Windows）：在 xj 的安裝資料夾裡打的話先停下來、叫你換資料夾（以前說「下次開的就是新版」，背景卻換不上去）；
  背景沒換成的話（例如別的視窗開著裡面的檔案），下次開 xj 和 `xj doctor` 會講原因和怎麼做
- `xj update`：你留的公鑰存成 UTF-16 也認得（Windows PowerShell 5.1 的 `'<公鑰>' > 檔案` 存的；以前說它「不是 minisign 的公鑰」、不更新）
- `xj update`：下載、解壓的時候按 Ctrl-C，下載和解到一半的檔案都收掉、不換（以前留在安裝資料夾的上一層，叫 `.xj-update-<16 進位>`，直接刪掉就好）
- `/model`：名字是檔案路徑的模型（llama.cpp 的 `llama-server` 沒加 `--alias` 時）也記得住，下次開 xj 照樣用它（以前設定檔的格式不收反斜線、空白，沒記進去）；
  看不見的字（方向控制、零寬）照樣不收
- Windows：開場的「沙盒裡的指令在這些磁碟的根目錄建得了資料夾」只講一次（以前講兩次），沙盒擋不全的時候也講（以前不講）；`xj -p`、`xj doctor` 也一樣。
  `xj -p` 也跟對話的開場一樣講沙盒的提醒（macOS 開了 `sandbox.allowLocalBinding` 時）
- 在 xj 裡打金鑰、新增服務時的說明照字詞折：英文字不從中間切開，折下來的列不從空白開始
- 使用者文件：README 改寫得更精簡、安全的說法更精準；install.md 寫了從 0.1.1、0.1.2 更新時怎麼做、Windows 怎麼留公鑰（`%USERPROFILE%`）、
  `xj reset` 會連你留的公鑰一起刪；安全說明補了「機密檔照檔名認」「沙盒裡的指令間接改得到機密檔」；config.md 寫了 `llama-server` 的 `--alias`；
  疑難排解補了這兩種更新的問題

## 0.1.2

**需要**：跟 0.1.1 一樣 —— macOS、Linux 或 Windows，Node 24 以上（Windows 建議 24.20.0 以上）、`ripgrep`；Linux 另外要 bubblewrap、socat 和 sandbox-runtime 0.0.79，
Windows 要 Git for Windows（見 `docs/install.md`）。

**從 0.1.1 更新**：打 `xj update`（Homebrew 裝的用 `brew upgrade xj`）。還在 0.1.0 的話，照 `docs/install.md`「更新」那一節的最後一段手動換一次。
設定檔、金鑰、對話紀錄都不用動。

**下載之後先驗**（兩個檔案跟壓縮檔一起放）：

```bash
shasum -a 256 -c xj-0.1.2-darwin.tar.gz.sha256
minisign -Vm xj-0.1.2-darwin.tar.gz -P <你留的那份公鑰>
```

Linux 是 `xj-0.1.2-linux-x64.tar.gz` 或 `xj-0.1.2-linux-arm64.tar.gz`（SHA-256 用 `sha256sum -c` 驗），Windows 是 `xj-0.1.2-windows.zip`
（PowerShell 的驗法見下面 0.1.0 那一段）。第一次用的話，照 `docs/install.md` 裝好、跑 `xj setup`。

修正：

- `xj reset`：照 `docs/install.md` 留了公鑰（`~/.xing-janus/minisign.pub`）之後也刪得掉（以前說「不像是 Xing Janus 的東西」就停了）；
  Windows 的 `acl-fix.json`、寫到一半被中斷留下的暫存檔也認得
- 安全：`id_dsa.pub` 也算機密檔（另外三種 SSH 金鑰的 `.pub` 本來就擋）
- 抓網頁（`web_fetch`）：同一個網站的轉址照說明最多跟 5 次（以前第 5 次就停）
- 使用者文件：快速開始補上安裝、換平台、更新；README 補上怎麼更新；疑難排解補上「右邊那一欄對不齊」。
  設定、使用說明、安裝、隱私、安全說明整個對過程式，改正寫錯、漏寫的地方，例如：`anthropic.fallback` 是請求被安全分類器擋下時才換模型；
  `limits` 的輸出上限碰到時只提醒、不停下來問；`dangerRemove` 加 `"sudo"` 會讓 `su`、`doas` 這幾個一起不問；Homebrew 移除的順序（先 `xj reset`）；
  Windows 移除時漏掉的檔案和認證管理員裡的金鑰；設定裡的 `tools`（`"reliable"`／`"unreliable"`）、`parallelToolCalls` 目前沒有作用

## 0.1.1

**需要**：跟 0.1.0 一樣 —— macOS、Linux 或 Windows，Node 24 以上、`ripgrep`；Linux 另外要 bubblewrap、socat 和 sandbox-runtime 0.0.79，
Windows 要 Git for Windows（見 `docs/install.md`）。

**從 0.1.0 更新**：0.1.0 還沒有 `xj update`，這一次照 `docs/install.md`「更新」那一節的最後一段手動換；之後打 `xj update` 就好
（Homebrew 裝的用 `brew upgrade xj`）。設定檔、金鑰、對話紀錄都不用動。

**下載之後先驗**（兩個檔案跟壓縮檔一起放）：

```bash
shasum -a 256 -c xj-0.1.1-darwin.tar.gz.sha256
minisign -Vm xj-0.1.1-darwin.tar.gz -P <你留的那份公鑰>
```

Linux 是 `xj-0.1.1-linux-x64.tar.gz` 或 `xj-0.1.1-linux-arm64.tar.gz`（SHA-256 用 `sha256sum -c` 驗），Windows 是 `xj-0.1.1-windows.zip`
（PowerShell 的驗法見下面 0.1.0 那一段）。第一次用的話，照 `docs/install.md` 裝好、跑 `xj setup`。

新的：

- macOS 可以用 Homebrew 裝（`docs/install.md`）；tap 更新 formula 之前先用公鑰驗過簽章
- `xj update`：你打了才連 GitHub，只裝比現在新的版本，SHA-256 和簽章都驗過才換上；`xj update --check` 只看有沒有新版
- 同一家可以有好幾把 API key：各取名字，照資料夾選（`keys.<廠商>.byFolder`）、`--key` 或 `/key` 換；選到的那一把找不到就不啟動；
  花費照 key 分開記，每一把可以另外設每天、每月的上限
- 開 xj 時先量這個終端機把 emoji 的組合、寬度不明確的字畫成幾格（macOS 的 Terminal.app、舊的 Windows 主控台跟其他終端機不一樣；
  「寬度不明確的字當成兩格」的設定也量得到），排版照量到的算：核准畫面有 emoji 的那一行不會再溢出、把右邊那一欄推走。一個字佔幾格改照 Unicode 的表
- 在 xj 裡換平台：`/model` 先選平台（標出本機的、連到哪裡、金鑰放在哪裡）、再選模型；這台的本機伺服器（Ollama、LM Studio、llama.cpp、vLLM）
  自己找出來，也能新增 OpenAI 相容的服務。還沒設金鑰的當場貼上（畫面上只顯示 •，查得到模型清單才存，存法跟 `xj setup` 一樣）。
  換到別的平台之前先確認連得上，連不上就照舊用原本的；對話要送到別台電腦之前先問。換了就記進設定檔的 `defaultModel`（以前 `/model` 只換這次對話）

修正：

- 安全：search 也照 read 擋指到機密檔的 hard link；危險指令不分大小寫、帶 `.exe` 的也認（`BASH -c`、`NOHUP`）；
  用系統管理員權限跑的 PowerShell 引號跳脫得更完整；`SKILL.md` 不能是指到別處的連結；PATH 上的網路路徑在問你之前不去讀
- 花費：換到別家之前的摘要也先看預算；子代理用別家的模型時照那一把 key 的上限
- Windows：有時把別的檔案（例如 README.md）當成 `.env` 的 hard link、整個指令被擋；沙盒的帳號第一次登入等得比較久；
  xj 結束時就收回上層資料夾的權限（以前要等下一次開沙盒才收）；剪貼簿不會多一個看不見的 U+FEFF；
  裝在使用者資料夾的 PowerShell 7 在沙盒裡跑不了：沙盒開著時改用 Program Files 的 7 或內建的 5.1，開場和 `xj doctor` 講一聲；
  在 Git Bash 自己的視窗（mintty）裡開的話，講清楚那不是 Windows 的主控台、請換到 Windows Terminal（以前只說「要在終端機裡」），
  Ctrl-C 也跟終端機裡一樣（中斷這一輪，連按兩次離開；以前按一次就結束）；Cygwin 裡打 `xj` 開得起來（以前啟動器把 `/cygdrive/…` 的路徑原封不動交給 node）；
  電腦很忙的時候（例如背景在 build）開始用，cmd、where 這類程式的中文輸出不會整個對話都變亂碼（以前問系統的字碼頁逾時一次就整個對話照 UTF-8 讀），
  開全部不問模式也不會被誤判成用系統管理員權限在跑而開不了
- 畫面：核准畫面照看得到的一個字換行（國旗、emoji 不會被切開）；換成全螢幕、改視窗大小之後，詢問的內容照新的寬度重排；
  拖曳複製折下來的那幾列不會帶進縮排的空白
- 其他：沙盒要講一聲的事開場也講（以前只有 `xj doctor` 講）；`xj -p` 也印設定檔的警告；告訴模型的日期照這台電腦的時區；
  `xj doctor` 金鑰那一列寫出找不到的原因；開場還在準備（例如開沙盒）的時候按 Ctrl-C 會收尾、結束（以前沒反應，要等開場做完）
- 使用者文件：每一項設定都寫出預設值

## 0.1.0

第一個版本。

**需要**：macOS、Linux 或 Windows，Node 24 以上（24 是現役 LTS，26 也可以）、`ripgrep`（`brew install ripgrep`）。
Linux 要另外裝 bubblewrap、socat，和沙盒要用的 sandbox-runtime（壓縮檔不附，見 `docs/install.md`）：
`npm install -g --ignore-scripts @anthropic-ai/sandbox-runtime@0.0.79`。
Windows 要 Git for Windows（bash 工具用 Git Bash）；沙盒是 alpha、要系統管理員權限裝一次，可以不裝（不裝的話每個指令都先問你）。
WSL2 也可以（裡面跑 Linux 版）。

**下載之後先驗**（兩個檔案跟壓縮檔一起放）：

```bash
shasum -a 256 -c xj-0.1.0-darwin.tar.gz.sha256
minisign -Vm xj-0.1.0-darwin.tar.gz -P <README 上的公鑰>
```

Linux 的檔名是 `xj-0.1.0-linux-x64.tar.gz` 或 `xj-0.1.0-linux-arm64.tar.gz`（照 CPU 選），SHA-256 用 `sha256sum -c` 驗。
Windows 是 `xj-0.1.0-windows.zip`，在 PowerShell 裡驗：

```powershell
(Get-FileHash xj-0.1.0-windows.zip -Algorithm SHA256).Hash.ToLower()   # 跟 .sha256 檔裡的一樣
minisign -Vm xj-0.1.0-windows.zip -P <README 上的公鑰>
```

公鑰請自己留一份，之後每次更新都用你留的那份驗（只從這裡拿的話，這裡被動手腳時公鑰也會跟著被換）。
照壓縮檔裡的 `docs/install.md`（英文的在 `docs/en/install.md`）解開、把 `xj` 放進 PATH（macOS 預設的 PATH 沒有 `~/.local/bin`，那份文件有各個 shell 和 Windows 的做法），
再跑 `xj setup`，到你的專案裡打 `xj`。

目前有：

- 對話、讀檔、搜尋、改檔、跑指令；改檔前秀 diff、`/undo` 還原
- Cerberus：權限規則、OS 沙盒（macOS、Linux；Windows 是 alpha）、信任的資料夾、機密檔、危險指令
- Windows 原生版：bash 工具用 Git Bash（權限判斷跟 macOS、Linux 一樣），另外有 powershell 工具（白名單只認很單純的唯讀寫法）
- Windows：PATH 上有、沙盒裡跑不了的工具（裝在你帳號底下的、用系統管理員權限 winget 裝到整台電腦的）開場和指令失敗時都會講是哪一個、怎麼辦，模型不會把它當成沒裝
- 模型：Anthropic（原生）、OpenAI、Ollama 和其他 Chat Completions 相容的服務；對話中途可以換
- 摘要（`/compact`）、接續之前的對話（`/resume`）、事後感測（`/checks`）、計劃模式、全自動模式（`--auto`）、附加檔案和圖片
- 在沙盒外跑的指令（設定檔的 `sandbox.excludedCommands`，例如 `docker`）：每次都問，按「是」才在沙盒外跑；預設是空的
- 全部不問模式（`--dangerously-skip-permissions`）：沒有沙盒、什麼都不問，只剩機密檔、受保護的檔案照樣擋、切換成 root（`sudo`）照樣問；
  每次啟動都要打字確認，用 root 跑的時候開不了。只建議在容器、虛擬機裡用
- Windows：開沙盒時發現沙盒的帳號寫得進 PATH 上的資料夾（指到不存在的資料夾也算），問你要不要用系統管理員權限擋（跳 UAC；可以永久允許，
  `/permissions` 拿得掉）；全部不問模式不問就擋。沙盒外會跑的程式（git、Git Bash、flutter 這類、MCP server…）放在那些地方的話，開沙盒時另外問一次要不要照樣跑
  （設定檔的 `sandbox.writablePrograms`）
- Agent Skills、MCP（只接設定檔列出的 server，預設在沙盒裡跑）
- 花費和預算、`xj doctor`、`xj setup`、`xj reset`
- 畫面：全螢幕模式（預設；`/tui classic` 換回一般模式）、每一行標出是誰（你／AI／xj）、寬的視窗右邊放待辦清單和目前的狀態（可以用滑鼠點）、滑鼠拖曳選取複製（反白時按 Ctrl-C 也是複製，不會中斷）
- 介面有繁體中文和英文（第一次啟動時選，`/language ui` 換）；給模型的說明和工具結果一律是英文
- 待辦清單：多步驟的工作模型自己列、有編號；你用講的就能加、改、刪
- 模型在做事的時候也能打字：做完手上這一步就交給它
- 抓網頁（`web_fetch`）：網域跟沙盒同一份清單、新的要問；本機、內網、雲端的 metadata 位址不問就擋。還沒有搜尋
- 子代理：找東西的 explore（唯讀，可以同時跑）、會改檔的 general（照你的模式問你）；每個動作照樣經過 Cerberus，xj 自己算改了什麼交給主代理檢查
- 永久允許（寫進設定檔，所有專案都適用）、`/permissions` 看和拿掉
- 詢問時按 Enter 就是「是」（設定檔的 `enterToApprove`，預設開；危險的、還沒看完的、永久允許的照樣要按 y）

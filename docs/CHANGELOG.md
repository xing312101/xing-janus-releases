# 更新說明

版本號照 semver。

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

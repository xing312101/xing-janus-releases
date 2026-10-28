# 疑難排解

先跑 `xj doctor`，它會說哪些能用、哪些不能、為什麼。不夠的話用 `xj --debug` 重現一次，送給 API 的原始請求和回應會存在對話紀錄的 `debug/` 裡（金鑰先遮掉）。要回報問題時貼 `xj doctor --report` 的輸出（不含 key 和對話內容）。

| 狀況 | 怎麼辦 |
|---|---|
| 沙盒啟動不了 | 照 `xj doctor` 說的裝好缺的東西（macOS：ripgrep；Linux：bubblewrap、socat、ripgrep、sandbox-runtime）。這時所有指令一律拒絕；真的要不經過沙盒跑，啟動時加 `--no-sandbox`（每個指令都要問） |
| Linux：找不到 apply-seccomp、版本不對 | 照訊息裡的那一行重裝 sandbox-runtime（`npm install -g --ignore-scripts @anthropic-ai/sandbox-runtime@<版本>`，見 [install.md](install.md) 的「Linux 的沙盒」）。用 nvm 的話，用跑 xj 的那個 Node 版本來裝 |
| 401、403 | key 錯了或沒有權限。`xj doctor` 會說 key 從哪裡讀的 |
| 429 | 被限流。Xing Janus 會照 retry-after 退避，最多重試 3 次 |
| 額度用完 | 到各家的後台看用量和預算 |
| 上下文太長 | 會自動摘要再試一次；還是太長就 `/clear` 開新的對話 |
| Ollama 說上下文不夠 | 用 `OLLAMA_CONTEXT_LENGTH=32768` 重新啟動 Ollama |
| 推理區塊被丟掉的提醒、歷史檢查回 400 | 這是 Xing Janus 的 bug：用 `--debug` 重現，回報 |
| 終端機壞掉（捲動區域沒還原） | 打 `reset` 或 `printf '\e[r'` |
| 啟動時說「xj 不採用 NODE_OPTIONS 裡的…」 | `NODE_OPTIONS` 裡的 `--import`、`--require` 這類會讓別的程式在 xj 裡面執行（沙盒外、拿得到金鑰），所以 xj 只接受 `--use-system-ca`、`--use-openssl-ca`、`--max-old-space-size`，其他的不用。是你自己設的就可以不管；不是的話，查一下是誰設的（例如專案的 `.envrc`） |

## docker 指令在沙盒裡失敗

`Cannot connect to the Docker daemon`、`permission denied ... docker.sock`：沙盒擋住了 Docker 的連線（能叫 Docker 做事就等於能碰整台電腦）。
要用的話把 `docker` 列進設定檔的 `sandbox.excludedCommands`（每次都問、按「是」才在沙盒外跑，見 config.md 的「Docker」），
或在可以整個丟掉的容器、虛擬機裡用全部不問模式。

## 在 WSL 裡用，但給的是 Windows 的路徑

`C:\\Users\\...` 這種寫法在 WSL 裡不是路徑，要用 `/mnt/c/Users/...`。不過**也不建議把專案放在那裡**：
跨檔案系統很慢，而且 hard link 和 inode 的語意不同（機密檔的檢查靠 inode，等於少一層保護）。
把專案放在 WSL 自己的檔案系統裡（例如 `~/code/專案`），`xj doctor` 會幫你確認。

## Windows

原生版（沙盒是 alpha）。

| 狀況 | 怎麼辦 |
|---|---|
| 「沒有裝 Windows 的沙盒：每個指令都要問」 | 這是還沒裝沙盒的正常狀態（要系統管理員權限才裝得了）。要裝的話，用「以系統管理員身分執行」的 PowerShell 照 [install.md](install.md) 的「Windows」把 sandbox-runtime 裝到整台電腦（`%ProgramFiles%\sandbox-runtime`），再跑一次 `windows-install`。裝好之後重開 xj |
| 「srt-win.exe 裝在使用者的資料夾裡（沙盒用的 srt-sandbox 帳號打不開它）」 | 照以前的步驟用一般權限 `npm install -g` 裝的，會在你帳號底下的 `%APPDATA%\npm`：沙盒的帳號讀不到那裡，沙盒起不來。照上一列重裝到整台電腦（xj 這時每個指令都問，不會不問就跑） |
| 「沙盒裡的指令寫得進 srt-win.exe 的資料夾」 | sandbox-runtime 裝在一般使用者也改得了的地方（例如 `C:\tools`、另一顆磁碟上自己建的資料夾）：沙盒裡的指令就能換掉沙盒本身，所以 xj 不用它（每個指令都問）。請照上上一列裝到 Program Files |
| 裝了沙盒，xj 卻說沙盒啟動不了 | 這時所有指令一律拒絕（跟 macOS、Linux 一樣）。照 `xj doctor` 的訊息處理；常見的是防火牆的規則被清掉（再跑一次 `windows-install`）、工作目錄在網路磁碟機上（沙盒的帳號看不到你對應的磁碟機，請把專案放在本機的磁碟） |
| 沙盒裡找不到 node、python 這類工具 | 沙盒用另一個帳號（`srt-sandbox`）跑，裝在你帳號底下的工具它讀不到。**開場如果有「PATH 上有、沙盒裡讀不到」的工具，xj 會直接講是哪一個、裝在哪裡、怎麼開**（2026-10-27 加的；指令真的因此失敗時也會告訴模型「這台有，只是沙盒讀不到」，免得它叫你去裝已經裝好的東西）。xj 照專案的種類開放常見的位置：一律開 Node 的安裝資料夾、per-user 的 Git、Scoop、winget 裝在你帳號底下的（`WinGet\Packages`、`Links`）；Node 專案（有 `package.json`）再開 npm 全域安裝；Python 專案開 per-user 的 Python；**還看不出是哪種專案時（全新的空資料夾，例如「幫我開一個 Python 專案」）Python 和 Node 的位置照樣開**；Android 專案（有 `build.gradle` 這類 Gradle 檔）開 Android SDK、`~/.gradle`。Flutter 專案不開 Flutter／fvm、Pub cache（flutter、dart 一律在沙盒外跑，見 config.md「沙盒裡跑不了的工具」）。其他的請改裝在整台電腦（Program Files；用 winget 的話見下一列），或加進設定的 `sandbox.toolchainReadable`（自己寫了這一項的話，xj 就照你寫的開，不再自己加） |
| 沙盒裡找不到用 winget 裝到整台電腦的工具（jq 這類），用路徑叫是 Permission denied | 用系統管理員權限 `winget install`（沒加 `--scope user`）裝的可攜式工具放在 `Program Files\WinGet`：資料夾大家都讀得到，執行檔本身卻只有裝的帳號和系統管理員能跑（winget 先下載到你的暫存資料夾再搬過去，權限跟著走），沙盒的帳號跑不了。**xj 會講是哪一個和 winget 的套件 ID**（開場、指令真的因此失敗時都講，Git Bash 和中文的 PowerShell 都認得；也會告訴模型不要說「沒有安裝」）。要在沙盒裡用：改用 `winget install --scope user --id <套件 ID>` 重裝（裝在你帳號底下，xj 會開放給沙盒），再重開 xj；或把那個指令加進設定檔的 `sandbox.excludedCommands`（在沙盒外跑、每次問）。加進 `toolchainReadable` 沒用：擋住沙盒的是檔案自己的權限 |
| 沙盒裡的 Gradle 說 `~/.gradle` 底下的某個資料夾建不了（Access is denied） | Windows 的沙盒只開放 `~/.gradle` 底下已經有的資料夾寫入，`wrapper`、`init.d` 只給讀（整個開放的話，會在沙盒外被執行的 wrapper、init 腳本要另外擋，啟動、離開各多十幾秒）。新的資料夾請在沙盒外跑一次 Gradle（例如 `flutter build apk`）讓它建好；或在設定檔的 `sandbox.writable` 自己列（例如 `["~/.gradle", "C:\\Users\\<你>\\AppData\\Local\\Pub\\Cache"]`：自己寫了這一項，xj 就照你寫的整個開，不再拆開，比較慢） |
| 沙盒裡的 git 說 `detected dubious ownership` | Windows 的沙盒用另一個帳號跑，你的 repo 對它來說「擁有者是別人」，git 要列在 safe.directory 才肯用。xj 自己列：專案、專案裡**開沙盒時已經有的**巢狀 repo（submodule 這類）、可寫的範圍、工具鏈，和你自己在 `~/.gitconfig` 設的 `safe.directory`（單獨一個 `*` 除外）。**用 `/add-dir`、`extraReadable` 開放讀的 repo 不算，是故意的**：git 會照那個 repo 的 `.git/config` 執行指令，而 `git status` 在白名單裡、不問就跑。對話途中才建的巢狀 repo（例如在專案裡 `git clone`）：Git 2.46 以上本來就認得；舊的 Git 請重開 xj 或更新 Git |
| 開著沙盒的 xj 啟動要等好一陣子（Android 專案；Flutter 專案不會） | Flutter 專案不再開放 Flutter SDK、Pub cache、Android SDK、`~/.gradle`（flutter、dart 一律在沙盒外跑；那台的 Flutter 專案從 42～77 秒變成 5～13 秒）。以下是以前的情況，Android 專案照舊：Windows 的沙盒開放一個資料夾，要逐一改底下每個檔案的權限：Flutter SDK、Pub cache、Android SDK、`~/.gradle` 加起來常常有幾十萬個檔案（2026-10-27 實測：三十幾萬個，那台啟動要 45～55 秒；開機後第一次會更久。離開不用等，權限在背景收回）。畫面上會講在等什麼，每 10 秒講一次做到哪個資料夾。想快一點：在設定檔設 `"sandbox": { "excludedCommands": ["flutter", "dart"], "toolchainReadable": [], "writable": [] }`：flutter、dart 改成在沙盒外跑、每次都先問你，其他指令照樣在沙盒裡 |
| 開著沙盒，白名單卻不生效、每個指令都問，而且說「寫得進沙盒外會被執行的地方」 | xj 開沙盒時會實際試寫 PATH 上的每個資料夾（還有沙盒讀得到的工具鏈、Git Bash 自己的資料夾、`excludedCommands` 的 SDK）：寫得進去的話，沙盒裡的指令就能在那裡放程式，之後在沙盒外以你的權限執行，所以白名單不生效（2026-10-27 加的）。常見原因是工具裝在 `C:\tools`、`D:\` 這種「所有登入的帳號都改得了」的地方。**PATH 上指到不存在的資料夾也算**（例如軟體移除了、PATH 那一筆還在）：沙盒在上層把它建出來、在裡面放程式就行 —— 最簡單的修法是把它從 PATH 拿掉；上層是磁碟的根目錄時，印的 icacls 只擋那一層建檔、建資料夾（`(WD,AD)`，不往下繼承）。要讓白名單回來：把那些工具改裝到 `Program Files`，或用系統管理員權限跑警告裡印的 `icacls "<資料夾>" /deny *<SID>:(OI)(CI)(WD,AD,WEA,WA,DC,DE)`（印的是往上連續寫得進去的最上層，例如 `C:\src`），再重開 xj。xj 會問你要不要直接幫你擋（用系統管理員權限，跳 UAC；可以永久允許），全部不問模式不問就擋，見 usage.md 的「全部不問模式」 |
| 沙盒開著，`flutter`、`dart` 每次都問「在沙盒外跑」 | 正常：它們在沙盒裡跑不了（每次都要寫 SDK 的 `bin\cache`，沙盒裡會卡住），xj 直接告訴模型，由它要求在沙盒外跑那一個指令、每次問你。不想被問：用 `xj --no-sandbox` 開（整個沒有沙盒），或在設定檔設 `"sandbox": { "excludedCommands": ["flutter", "dart"] }`（一樣每次問）。見 config.md「沙盒裡跑不了的工具」 |
| 沙盒裡 `curl`、`git` 連 https 出現 `CRYPT_E_REVOCATION_OFFLINE` | Windows 內建的 TLS（schannel）查憑證撤銷時連不出去（被沙盒的防火牆擋）。`curl --ssl-no-revoke`、`git -c http.schannelCheckRevoke=false` |
| 找不到 Git Bash | 裝 Git for Windows，或在設定檔寫 `"windows": { "gitBash": "C:\\…\\bin\\bash.exe" }`。`C:\Windows\System32\bash.exe` 是 WSL 的啟動程式，不是 Git Bash，xj 不會用它 |
| 沙盒裡跑 node 的腳本（`npm test`、`node --test`、`node 檔案.js`）出現 `EPERM: operation not permitted, lstat 'C:\Users\<你>\Desktop'` | Node 找腳本的實際路徑時，要讀每一層上層資料夾的屬性。xj 開沙盒時會替家目錄裡的那幾層加「只能讀屬性」的權限（見下面「不一樣的地方」），正常就不會這樣。還是出現的話，看開場有沒有「上層資料夾加不上權限」的提醒（例如公司的電腦擋了 PowerShell）：這時這類指令改在沙盒外跑（設定檔的 `sandbox.excludedCommands` 加 `npm`、`node`，每次都先問你） |
| 沙盒裡的 PowerShell 印出 `Project:\src\…` 這種路徑 | 沙盒的帳號讀不到你家目錄這些上層的資料夾，PowerShell 自己進不了工作目錄（會停在 `C:\`），所以 xj 開一個指到工作目錄的 `Project:` 磁碟機讓它進去。`Project:\src\a.ts` 就是專案裡的 `src\a.ts` |
| PowerShell 的指令（不是 `Get-Content` 這種）每個都問 | PowerShell 的白名單只認很單純的寫法：唯讀的 cmdlet（`Get-ChildItem`、`Get-Content`、`Select-String`、`Select-Object`…），參數是一般的字或單引號，沒有 `$`、括號、雙引號、`>`。問你的時候會講是哪裡不算（例如「有 $（變數）」）。其他的指令請讓它用 bash（Git Bash） |
| 在 cmd 裡用 `xj -p` 時按了 Ctrl-C，xj 結束後 cmd 問「要終止批次工作嗎 (Y/N)?」 | 那是 cmd.exe 自己問的（xj.cmd 是批次檔），回答 Y 或 N 都一樣。對話（不加 `-p`）裡的 Ctrl-C 是 xj 自己處理的按鍵，不會這樣；用 PowerShell 跑 `xj -p` 也不會 |
| MCP server 說「Windows 的沙盒現在不能跑 MCP server」 | Windows 的沙盒還沒辦法把 stdin 接給沙盒裡的程式，而 MCP（stdio）靠的就是 stdin。要用的話在設定檔把那個 server 的 `sandbox` 設成 `false`：它會以你的權限在沙盒外跑，每次呼叫照樣先問你（除非列在 `autoApprove`）。只給你信任的 server 這樣設 |
| 擔心在別人的 repo 裡打 `xj` 會跑到 repo 裡的同名程式 | xj.cmd 自己找 node 時不找目前的資料夾，也不用工作目錄底下的 node（PATH 上經過 junction 指進工作目錄的認不出來：批次檔沒辦法查實際路徑，Git Bash 用的 sh 啟動器認得）。但「找 xj.cmd 本身」是 cmd.exe 的事：cmd 會先找目前的資料夾，所以 repo 裡放一個 `xj.cmd` 就會被執行。建議用 PowerShell（它不找目前的資料夾），或在系統的環境變數設 `NoDefaultCurrentDirectoryInExePath=1` |
| 啟動時說「xj 不採用 NODE_OPTIONS」 | Windows 的 xj.cmd 整個不採用 NODE_OPTIONS（批次檔拆不了引號，沒辦法只留安全的那幾個）；裡面有 `--use-system-ca`、`--use-openssl-ca` 的話會直接傳給 node |
| 中文的輸出變亂碼 | PowerShell 工具的輸出、`Get-Content`／`Set-Content`／`Out-File`／`>` 讀寫檔，xj 都已經改成 UTF-8（Windows 內建的 PowerShell 5.1 寫出來的檔案會帶 BOM，所以建檔、改檔請讓它用 write、edit 工具）。Git Bash 裡跑 Windows 內建的程式（`ipconfig`、`cmd //c dir`）會用主控台的字碼頁，這個 xj 改不了 |
| xj doctor 的「程序收尾」說 job object 不能用 | xj 用 Windows 內建的 PowerShell 5.1 在背景管理 job object（Ctrl-C、逾時時收掉指令開的每個程序）。公司的電腦把 PowerShell 或 Add-Type 擋掉的話就用不了：這時 Ctrl-C 可能收不乾淨 Git Bash 開的背景程序，xj 結束時看門狗會再收一次 |
| 指令開的程式（視窗程式、`start` 開的、Gradle daemon）指令一結束就不見了 | xj 把每個指令開的程式都放進同一個 job object，指令結束時整個收掉（Git Bash 開的背景程序才收得乾淨；開著沙盒時是沙盒自己的 job object，一樣會收），比 macOS、Linux 嚴。要留著用的程式（開發伺服器、模擬器、編輯器）請在 xj 外面自己開。Gradle 的 daemon 每次 build 會重開一個（每次多等幾秒，結果不受影響） |
| Git Bash 的指令找不到 `~/.gitconfig`、`~/.ssh`（網域帳號、家目錄在網路磁碟上） | 沒有沙盒時 xj 直接跑 Git 的 `usr\bin\bash.exe`，沒有 HOME 的話照 Git Bash 的規則補：`HOMEDRIVE`＋`HOMEPATH` 是存在的資料夾就用它，不然用 `USERPROFILE`。要固定用某個資料夾，在 Windows 的使用者環境變數設 `HOME` |

**Windows 的沙盒跟 macOS、Linux 不一樣的地方**（alpha）：

- 機密檔是在沙盒啟動時一個一個寫進權限（ACL）的。之後才出現的機密檔，xj 每個指令執行前會另外擋。
  專案很大（超過約兩萬個檔案，不含 node_modules 這類）時找不完，沒找到的沙盒擋不到：這時白名單不生效、每個指令都問，開場和 `xj doctor` 會講。
- 還不存在的受保護檔案（例如 `.vscode/tasks.json`、`.git/hooks/` 底下的新檔），沙盒擋不到「建立」：
  xj 的工具和重新導向照樣擋，指令建了的話會在結果裡提醒你檢查。
- `/add-dir` 之後沙盒要重新啟動（Windows 的權限只能這樣換），開著的 MCP server 也會重開。
- 工作目錄（和開放的工具鏈，例如 npm 的全域安裝）在家目錄底下時，上面每一層資料夾（例如 `Desktop`、`AppData`）開著沙盒的時候多一條
  「`srt-sandbox`：只能讀屬性」的權限（沒有它，沙盒裡的 node 腳本會 EPERM）。沙盒看得到的只有那幾層資料夾本身的時間、隱藏這些屬性，
  列不出裡面有什麼、讀不到檔案。xj 結束時在背景拿掉；同時開著好幾個 xj 的話，最後一個結束才拿；xj 被強制結束的話，下一次開沙盒時收掉。
- 所有 xj 的沙盒都用同一個 `srt-sandbox` 帳號：同時開著兩個 xj 的話，一個開放給它的資料夾（專案、工具鏈），
  另一個的沙盒裡也讀得到，直到前一個結束、權限收回。不要一邊開著可信的專案、一邊在不可信的專案裡開 xj。
- 沙盒啟動時要替整個專案和開放的工具鏈設權限：檔案多的話（node_modules、Flutter／Android 的工具鏈）要等一陣子，畫面上會講在等什麼。
  結束時權限交給背景的程式收回，xj 不用等；收完之前，沙盒的帳號還留著那些權限（大的工具鏈要半分鐘以上）。
- 背景的程式被中斷的話（例如收完之前就登出、關機），沙盒的帳號會暫時留著工具鏈資料夾的權限，
  要到下一次在這台電腦上用到沙盒時才清掉（srt-win 會順便清掉已經結束的 xj 留下的）。
- 專案放在 D: 這種另外的磁碟、而那顆磁碟對所有使用者開放時，沙盒的帳號也讀得到、改得到那顆磁碟上別的資料夾（見 cerberus.md）。
  建議把專案放在你的家目錄（`C:\Users\<你>\…`）底下：那裡預設只有你自己讀得到。
- 沙盒裡的指令拿到的是沙盒帳號自己的環境：xj 會把它自己設的幾個（不等輸入帳密、npm 的快取、沙盒用的 git 設定）和設定檔的 `env.passthrough`
  寫在指令前面，所以會出現在 srt-win 的指令列上；沙盒的 proxy 帳密（程式庫產生的，每次啟動都換）也在那裡。
  同一個帳號的程式都看得到指令列：**不要把金鑰放進 `env.passthrough`**。
- 不在白名單的 PowerShell 指令提到受保護的檔案（`.git/hooks`、`.vscode`…）一律擋：看不出是讀還是寫。`Get-Content .vscode\tasks.json`
  這種白名單的讀法照常跑；要看內容也可以讓它用 read。

# 安裝、更新、移除

## 需要先裝的

- **macOS**（完整支援）。Linux 要另外裝 bubblewrap、socat、ripgrep，和 sandbox-runtime（見下面「Linux 的沙盒」）。
- **Windows**：原生版要 Git for Windows、Node 24 以上，沙盒是 alpha、可以不裝（見下面「Windows」「Windows 的安裝」）。
  也可以用 WSL2（裡面就是 Linux，沙盒是 bubblewrap）。用 WSL2 的話，四件事要知道：
  1. 在 WSL 裡裝 `bubblewrap`、`socat`、`ripgrep`、Node 24 以上，和 sandbox-runtime（同下面「Linux 的沙盒」）。
  2. **專案放在 WSL 自己的檔案系統裡**（例如 `~/code/專案`），不要放 `/mnt/c/...`：跨檔案系統很慢，
     而且 hard link 和 inode 的語意不同 —— 機密檔的檢查靠 inode（`xj doctor` 會提醒）。
  3. 沒有鑰匙圈，金鑰走金鑰檔（權限 600）或環境變數。**金鑰檔和 `~/.xing-janus` 不要放在 `/mnt/c`**：
     那裡的 `chmod 600` 是沒用的，等於沒有權限保護（`xj doctor` 會直接標成沒過）。
  4. 要能建 user namespace（bubblewrap 的前提）；新版 WSL2 預設可以。
- **Node 24 以上**：`node -v` 看版本。用 LTS 的版本（偶數的主版本）；奇數版只支援半年，很快就結束支援，不要用。
- **ripgrep**：`brew install ripgrep`（沙盒和 search 都要用）。Windows：`winget install BurntSushi.ripgrep.MSVC`（只有 search 用）。

### Windows

- **Node 24 以上**：從 nodejs.org 下載安裝程式，裝在整台電腦（預設的 `C:\Program Files\nodejs`）。
  建議用 24.20.0 以上：更舊的 Node 在 Windows 上被強制結束時偶爾會當掉，`xj doctor` 會提醒。
  用 nvm-windows 也可以；沙盒的帳號讀得到它的資料夾（xj 會開放）。
- **Git for Windows**（附 Git Bash）：https://git-scm.com/download/win 。bash 工具用它跑指令，權限判斷跟 macOS、Linux 一樣；
  沒裝的話只剩 powershell 工具，而 PowerShell 只有白名單裡的唯讀寫法（`Get-ChildItem`、`Get-Content`…）不用問，其他的每個都要問。
  裝在別的地方的話，設定檔寫 `windows.gitBash`（見 config.md）。
- **終端機**：用 Windows Terminal（Windows 11 內建）。舊的主控台顏色和貼上多行文字不一定正常，`xj doctor` 會提醒。
  Git Bash 自己的視窗（mintty）不是 Windows 的主控台：xj 在那裡只能用陽春的畫面，第一次的使用提醒、`xj setup` 也做不了。
  要用 Git Bash 的話，在 Windows Terminal 裡開（Git 的安裝程式可以在 Windows Terminal 新分頁的選單加一個 Git Bash）。
- **沙盒（alpha，可以不裝）**：指令用一個專用的 `srt-sandbox` 帳號跑，網路用 Windows 防火牆擋、檔案用權限（ACL）擋。
  要用系統管理員權限設定一次：在開始功能表的 Windows PowerShell 上按右鍵 →「以系統管理員身分執行」，然後：

  ```powershell
  npm install -g --prefix "$env:ProgramFiles\sandbox-runtime" --ignore-scripts @anthropic-ai/sandbox-runtime@0.0.79
  & "$env:ProgramFiles\sandbox-runtime\srt.cmd" windows-install
  ```

  **一定要裝在整台電腦**（上面的 `--prefix`），不要用 npm 預設的位置（你帳號底下的 `%APPDATA%\npm`）：
  沙盒的帳號讀不到你帳號底下的檔案，裝在那裡的話沙盒起不來；Program Files 也只有系統管理員改得了，沙盒裡的指令換不掉它。
  xj 會自己找到這個位置（`xj doctor` 會印出用的是哪一份）。以前照舊的步驟裝在 `%APPDATA%\npm` 的，用一般權限的
  PowerShell 跑 `npm uninstall -g @anthropic-ai/sandbox-runtime` 拿掉那一份（不拿掉 xj 也不會用它）。

  **沒裝沙盒的時候，每個指令都會先問你**（跟 `--no-sandbox` 一樣，白名單不生效）；裝了之後白名單的指令才會直接跑。
  沙盒的限制見 troubleshooting.md 的「Windows」。

### Linux 的沙盒

Linux 的沙盒還要一支 `apply-seccomp`（擋 Unix socket，例如借用 ssh-agent、docker.sock）。它靜態連結了
GNU C Library（LGPL），所以 xj 的壓縮檔不附，請自己從 npm 裝（版本要跟 xj 用的一樣）：

```bash
npm install -g --ignore-scripts @anthropic-ai/sandbox-runtime@0.0.79
```

- `--ignore-scripts`：不執行套件的安裝腳本（它也不需要）。
- 用 nvm 的話，用跑 xj 的那個 Node 版本來裝。npm 的 prefix 改過的（`~/.npm-global` 這類）也找得到。
- 裝的時候說沒有權限（`EACCES`）：可以改用 nvm，或把 npm 的 prefix 設到家目錄（`npm config set prefix ~/.npm-global`）再裝一次，xj 都找得到。
- 裝在沙盒寫得到的地方（專案資料夾、暫存區）的那一份不會用：沙盒裡的指令換得掉它。
- `xj doctor` 會印出用的是哪一份。找不到或版本不對時，沙盒那一列會直接給出要跑的指令；
  xj 更新後如果要換版本，也是照那一行重裝。
- 不裝的話沙盒起不來，xj 會拒絕跑任何指令。要不經過沙盒跑，啟動時加 `--no-sandbox`（每個指令都會先問你）。

## 安裝

### macOS：用 Homebrew

```bash
brew tap xing312101/xing-janus
brew trust --formula xing312101/xing-janus/xj
brew install xj
```

Homebrew 預設不載入第三方 tap 的 formula，`brew trust` 只信任 xj 這一個（不是整個 tap）。Node 和 ripgrep 會一起裝好。xj 自己一律用 brew 裝的 node 跑（nvm 切到哪一版都不影響），你專案裡的指令照舊用 PATH 上的 node。
formula 由 tap 自己的 workflow 照這個發佈頁面最新的版本更新：更新之前先用公鑰驗過簽章（brew 本身只核對 SHA-256），
下載的檔案 brew 會照 formula 核對 SHA-256。想自己再驗一次的話，照下面手動安裝的第 2 步。裝好之後跑 `xj doctor`。

### 手動安裝（Linux、Windows，或不用 Homebrew）

1. 到 releases 頁面下載你那個平台的 `xj-<版本>-<平台>.tar.gz`、`.sha256` 和 `.minisig`。`<平台>`：
   - macOS：`darwin`（Apple 晶片和 Intel 共用一個）
   - Linux（含 WSL）：照 CPU 選 `linux-x64` 或 `linux-arm64`。`uname -m` 印出 `x86_64` 就選 x64，`aarch64` 就選 arm64
   - Windows：`xj-<版本>-windows.zip`（x64 和 arm64 共用一個；步驟見下面「Windows 的安裝」）。用 WSL2 的話照上面選 Linux 的檔案。
     arm64 的 Windows（例如 Snapdragon 的筆電）還沒實際測過：碰到問題請在發佈頁面所在的 repo 開 issue
2. 確認檔案沒壞、而且是發佈者發的：

   ```bash
   shasum -a 256 -c xj-<版本>-<平台>.tar.gz.sha256      # Linux 沒有 shasum 的話用 sha256sum -c
   minisign -Vm xj-<版本>-<平台>.tar.gz -P <公鑰>
   ```

   **公鑰第一次拿到之後請自己留一份**（例如存成 `~/.xing-janus/minisign.pub`），之後每次更新都用你留的那一份驗。`xj reset` 會連 `~/.xing-janus` 一起刪，所以在別的地方也留一份。
   公鑰跟程式放在同一個 repo，所以只從那裡拿的話，repo 被動手腳時公鑰會跟著被換掉，簽章就驗不出問題。

   驗完請看 minisign 印出來的那行說明（trusted comment），裡面有版本和檔名：

   ```
   Trusted comment: xing-janus v<版本> xj-<版本>-<平台>.tar.gz
   ```

   版本要跟你要裝的一樣。簽章驗得過只代表「這是發佈者簽的某一版」，不代表是最新版：
   有人可以把舊版（連同它的簽章）換上來，所以不要裝比你現在用的更舊的版本。

3. 解壓，放到你喜歡的地方，把 `xj` 連到 PATH 裡：

   ```bash
   mkdir -p ~/.local/share ~/.local/bin
   tar -xzf xj-<版本>-<平台>.tar.gz -C ~/.local/share
   ln -sf ~/.local/share/xj-<版本>-<平台>/xj ~/.local/bin/xj
   ```

   macOS 預設的 PATH 沒有 `~/.local/bin`（有些 Linux 有）。`command -v xj` 什麼都沒印的話，照你用的 shell（`echo $SHELL` 看）加進它的設定，
   再開一個新的終端機視窗：

   | shell | 指令 |
   |---|---|
   | zsh（macOS 預設） | `echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc` |
   | bash（Linux） | `echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc` |
   | bash（macOS：終端機開的是 login shell，讀的是另一個檔） | `echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bash_profile` |
   | fish | `fish_add_path ~/.local/bin` |

   其他的 shell 照它自己的方式把 `~/.local/bin` 加進 PATH。新的視窗裡 `xj --version` 印得出版本就好了。

4. Linux：照上面的「Linux 的沙盒」裝 sandbox-runtime。
5. 跑 `xj doctor`，每一項都是 ✓ 或 -（不用的功能）就好了。

壓縮檔裡已經有打包好的程式（`xj.mjs`）和它需要的東西，不用 `npm install`，也不會執行任何套件的安裝腳本。

### Windows 的安裝

先裝好上面「Windows」列的東西。在 PowerShell 裡：

1. 確認檔案沒壞：

   ```powershell
   (Get-FileHash xj-<版本>-windows.zip -Algorithm SHA256).Hash.ToLower()
   Get-Content xj-<版本>-windows.zip.sha256      # 前面那串要一樣
   ```

2. 驗簽章：minisign 的 Windows 版從它的 GitHub releases 下載（或 `scoop install minisign`），然後

   ```powershell
   minisign -Vm xj-<版本>-windows.zip -P <公鑰>
   ```

   公鑰一樣請自己留一份，也一樣要看 trusted comment 裡的版本（見上面第 2 步）。留在 `~/.xing-janus/minisign.pub` 的話，`xj update` 每次都會核對它：

   ```powershell
   New-Item -ItemType Directory -Force "$env:USERPROFILE\.xing-janus" | Out-Null
   Set-Content "$env:USERPROFILE\.xing-janus\minisign.pub" '<公鑰>'
   ```
3. 解壓到一個不帶版本號的資料夾（`%LOCALAPPDATA%\Programs\xj`），把它加進你的 PATH（只加給你自己）：

   ```powershell
   $programs = "$env:LOCALAPPDATA\Programs"
   $dir = "$programs\xj"
   Expand-Archive xj-<版本>-windows.zip -DestinationPath $programs -Force
   if (Test-Path $dir) { Remove-Item -Recurse -Force $dir }     # 更新的時候：拿掉舊版
   Rename-Item "$programs\xj-<版本>-windows" xj
   $key = [Microsoft.Win32.Registry]::CurrentUser.OpenSubKey('Environment', $true)
   $path = $key.GetValue('Path', '', 'DoNotExpandEnvironmentNames')   # 原本的寫法（%USERPROFILE% 這類不展開）
   if ((($path -split ';') | ForEach-Object { [Environment]::ExpandEnvironmentVariables($_) }) -notcontains $dir) {   # PATH 只加一次
     $key.SetValue('Path', ((@($path -split ';') + $dir) | Where-Object { $_ }) -join ';', 'ExpandString')
     # 通知已經開著的程式（檔案總管）PATH 改了：設一個環境變數再拿掉（只有這個會廣播）
     [Environment]::SetEnvironmentVariable('XJ_PATH_CHANGED', '1', 'User'); [Environment]::SetEnvironmentVariable('XJ_PATH_CHANGED', $null, 'User')
   }
   $key.Close()
   ```

   資料夾不帶版本號：更新的時候照樣做這一步，PATH 不用再改（每一版各加一個資料夾的話，舊版排在前面，打 `xj` 跑的會是舊版）。
   已經開著的視窗讀不到新的 PATH：開一個新的終端機視窗（Windows Terminal 開新分頁還是找不到的話，整個關掉再開），
   `xj --version` 印得出版本就好了。cmd、PowerShell 用的是 `xj.cmd`，Git Bash 用的是同一個資料夾裡的 `xj`。
   這段照原本的寫法改 PATH：`%USERPROFILE%` 這類寫法和值的種類（`REG_EXPAND_SZ`）都不變。也可以用開始功能表搜尋
   「編輯您帳戶的環境變數」，在 Path 新增 `%LOCALAPPDATA%\Programs\xj`。
4. 要沙盒的話，照上面「Windows」用系統管理員權限把 sandbox-runtime 裝到整台電腦、跑 `windows-install`。
5. 跑 `xj doctor`。金鑰用 `xj setup` 存（存進 Windows 的認證管理員；存不進去的話存成只給你自己讀的金鑰檔），或設環境變數 `XING_JANUS_ANTHROPIC_API_KEY`。

## 更新

用 Homebrew 裝的：`brew upgrade xj`。

手動裝的（macOS、Linux、Windows）：`xj update`（打 `xj upgrade` 也一樣）。`xj update --check` 只看有沒有新版、不下載。v0.1.0 還沒有 `xj update`：從 v0.1.0 要照下面最後一段手動更新一次。
從 v0.1.1、v0.1.2 更新的話（換版是那兩版的程式在做）：macOS、Linux 不要在家目錄打 `xj update`（換到別的資料夾，例如你的專案裡），在家目錄打的話新版會裝好，`xj` 的連結卻不會換過去（見 troubleshooting.md）；
Windows 不要在 xj 的資料夾裡打（見下面）；你留的公鑰是用 PowerShell 的 `>` 存的話，那兩版認不得，從你原本那一份轉存（不要從發佈頁面重抄）：`$f = "$env:USERPROFILE\.xing-janus\minisign.pub"; (Get-Content $f) | Set-Content $f`。

- Xing Janus 不會自己連網路檢查更新（它平常對外的連線只有模型的 API 和你允許的網域）：你打 `xj update` 才連，只連 GitHub（查 releases 頁面最新的版本、下載那一版的檔案）。
  要經過 proxy 的話，設 `NODE_USE_ENV_PROXY=1` 和 `HTTPS_PROXY`。
- 只裝比現在新的版本；下載之後先驗 SHA-256 和簽章（用這一版裡內建的公鑰，不從網路拿），簽章的附註也要是那一版、那個檔，都過了才換上。
  你照上面第 2 步自己留了公鑰（`~/.xing-janus/minisign.pub`）的話，它要跟內建的是同一把，不一樣就停下來（可能換了金鑰，也可能檔案被換過：先看 README 和 CHANGELOG）。
- macOS、Linux：新版解在舊版的旁邊（例如 `~/.local/share/xj-<新版本>-<平台>`），PATH 上指到舊版的 `xj` 連結改指新版；舊版留著，確定沒問題之後自己刪。
- Windows：新版先解在旁邊（`%LOCALAPPDATA%\Programs\xj.update`），這個 xj 結束之後才換上（舊的改名成 `xj.old`），下次開的就是新版；PATH 不用改。
  換上要把資料夾改名，資料夾被開著就換不成：不要在 xj 的資料夾裡打 `xj update`（0.1.3 起它會先停下來叫你換資料夾）；別的視窗開著裡面的檔案而沒換成的話，0.1.3 起下次開 xj 會講（troubleshooting.md）。
- 下載、解壓的時候按 Ctrl-C 會中斷：0.1.3 起下載和解到一半的檔案都收掉，不會換。
- 新版要的 sandbox-runtime 版本不一樣的話（Linux、Windows 是你自己裝的），`xj update` 會印出要重裝的那一行。

也可以照上面的步驟自己下載、驗證、解壓，再把 `xj` 的連結指到新的資料夾。
Windows：先關掉正在跑的 xj，照「Windows 的安裝」的第 3 步再做一次（換掉 `%LOCALAPPDATA%\Programs\xj` 裡的檔案；PATH 已經有了就不會再加）。設定和對話紀錄在 `~/.xing-janus/`，不受影響。

## 移除

用 Homebrew 裝的：先跑下面的 `xj reset`（`brew uninstall` 之後就沒有 `xj` 可以跑了），再 `brew uninstall xj`、`brew untap xing312101/xing-janus`（代替下面刪掉程式的那一行），最後刪掉鑰匙圈裡的金鑰。

```bash
xj reset                                           # 刪掉 ~/.xing-janus 和 ~/.xing-janus-cache（會先列出來、問過再刪）
rm ~/.local/bin/xj && rm -r ~/.local/share/xj-*    # 刪掉程式
security delete-generic-password -s xing-janus-anthropic   # 鑰匙圈裡的金鑰（openai 等也一樣）
```

同一家好幾把 key 的話，有名字的那幾把在鑰匙圈（Windows 是認證管理員）裡是另外的項目（`xing-janus-<廠商>--<名字>`，例如 `xing-janus-anthropic--work`），要各刪一次。

Windows：`xj reset` 之後，在一般權限的 PowerShell 裡刪掉程式的資料夾（連同 `xj update` 留下的舊版 `xj.old` 和紀錄 `xj-update.log`）、
xj 記沙盒權限用的 `%LOCALAPPDATA%\xing-janus`，從 PATH 拿掉程式的資料夾，再刪掉認證管理員裡的金鑰（PATH 是你自己帳號的，不要用系統管理員權限跑）：

```powershell
$dir = "$env:LOCALAPPDATA\Programs\xj"
Remove-Item -Recurse -Force $dir
Remove-Item -Recurse -Force "$dir.old", "$dir.update", "$env:LOCALAPPDATA\Programs\xj-update.log", "$env:LOCALAPPDATA\xing-janus" -ErrorAction SilentlyContinue
$key = [Microsoft.Win32.Registry]::CurrentUser.OpenSubKey('Environment', $true)
$path = $key.GetValue('Path', '', 'DoNotExpandEnvironmentNames')   # 跟安裝的第 3 步一樣：照原本的寫法改
$key.SetValue('Path', (($path -split ';') | Where-Object { $_ -and [Environment]::ExpandEnvironmentVariables($_) -ne $dir }) -join ';', 'ExpandString')
$key.Close()
[Environment]::SetEnvironmentVariable('XJ_PATH_CHANGED', '1', 'User'); [Environment]::SetEnvironmentVariable('XJ_PATH_CHANGED', $null, 'User')   # 通知 PATH 改了
cmdkey /delete:xing-janus-anthropic     # 認證管理員裡的金鑰（openai 等、有名字的那幾把也是各刪一次）
```

PATH 裡的 xj 那一項，寫成 `%LOCALAPPDATA%\Programs\xj` 的也會拿掉；其他項目照原本的寫法留著。

裝過沙盒的話，再在系統管理員權限的 PowerShell 裡：

```powershell
& "$env:ProgramFiles\sandbox-runtime\srt.cmd" windows-uninstall    # 拿掉 srt-sandbox 帳號和防火牆的規則
Remove-Item -Recurse "$env:ProgramFiles\sandbox-runtime"
```

`%ProgramData%\sandbox-runtime` 是程式庫留下的資料夾，要清乾淨的話也刪掉。

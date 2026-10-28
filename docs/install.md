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
- **Node 24 以上**：`node -v` 看版本（24 是現役 LTS；26 也可以，23、25 已經結束支援，不要用）。
- **ripgrep**：`brew install ripgrep`（沙盒和 search 都要用）。Windows：`winget install BurntSushi.ripgrep.MSVC`（只有 search 用）。

### Windows

- **Node 24 以上**：從 nodejs.org 下載安裝程式，裝在整台電腦（預設的 `C:\Program Files\nodejs`）。
  用 nvm-windows 也可以；沙盒的帳號讀得到它的資料夾（xj 會開放）。
- **Git for Windows**（附 Git Bash）：https://git-scm.com/download/win 。bash 工具用它跑指令，權限判斷跟 macOS、Linux 一樣；
  沒裝的話只剩 powershell 工具，而 PowerShell 只有白名單裡的唯讀寫法（`Get-ChildItem`、`Get-Content`…）不用問，其他的每個都要問。
  裝在別的地方的話，設定檔寫 `windows.gitBash`（見 config.md）。
- **終端機**：用 Windows Terminal（Windows 11 內建）。舊的主控台顏色和貼上多行文字不一定正常，`xj doctor` 會提醒。
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

1. 到 releases 頁面下載你那個平台的 `xj-<版本>-<平台>.tar.gz`、`.sha256` 和 `.minisig`。`<平台>`：
   - macOS：`darwin`（Apple 晶片和 Intel 共用一個）
   - Linux（含 WSL）：照 CPU 選 `linux-x64` 或 `linux-arm64`。`uname -m` 印出 `x86_64` 就選 x64，`aarch64` 就選 arm64
   - Windows：`xj-<版本>-windows.zip`（x64 和 arm64 共用一個；步驟見下面「Windows 的安裝」）。用 WSL2 的話照上面選 Linux 的檔案
2. 確認檔案沒壞、而且是發佈者發的：

   ```bash
   shasum -a 256 -c xj-<版本>-<平台>.tar.gz.sha256      # Linux 沒有 shasum 的話用 sha256sum -c
   minisign -Vm xj-<版本>-<平台>.tar.gz -P <公鑰>
   ```

   **公鑰第一次拿到之後請自己留一份**（例如存成 `~/.xing-janus/minisign.pub`），之後每次更新都用你留的那一份驗。
   公鑰跟程式放在同一個 repo，所以只從那裡拿的話，repo 被動手腳時公鑰會跟著被換掉，簽章就驗不出問題。

   驗完請看 minisign 印出來的那行說明（trusted comment），裡面有版本和檔名：

   ```
   Trusted comment: xing-janus v0.1.0 xj-0.1.0-darwin.tar.gz
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
5. 跑 `xj doctor`，每一項都是 ✓ 或 －（不用的功能）就好了。

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

   公鑰一樣請自己留一份，也一樣要看 trusted comment 裡的版本（見上面第 2 步）。
3. 解壓到一個不帶版本號的資料夾（`%LOCALAPPDATA%\Programs\xj`），把它加進你的 PATH（只加給你自己）：

   ```powershell
   $programs = "$env:LOCALAPPDATA\Programs"
   $dir = "$programs\xj"
   Expand-Archive xj-<版本>-windows.zip -DestinationPath $programs -Force
   if (Test-Path $dir) { Remove-Item -Recurse -Force $dir }     # 更新的時候：拿掉舊版
   Rename-Item "$programs\xj-<版本>-windows" xj
   $path = [Environment]::GetEnvironmentVariable('Path', 'User')
   if (($path -split ';') -notcontains $dir) {                  # PATH 只加一次
     [Environment]::SetEnvironmentVariable('Path', ((@($path -split ';') + $dir) | Where-Object { $_ }) -join ';', 'User')
   }
   ```

   資料夾不帶版本號：更新的時候照樣做這一步，PATH 不用再改（每一版各加一個資料夾的話，舊版排在前面，打 `xj` 跑的會是舊版）。
   已經開著的視窗讀不到新的 PATH：開一個新的終端機視窗（Windows Terminal 開新分頁還是找不到的話，整個關掉再開），
   `xj --version` 印得出版本就好了。cmd、PowerShell 用的是 `xj.cmd`，Git Bash 用的是同一個資料夾裡的 `xj`。
   這段會把你 PATH 裡 `%USERPROFILE%` 這類寫法換成展開後的路徑（用起來一樣）；要保留原本的寫法，改用開始功能表搜尋
   「編輯您帳戶的環境變數」，在 Path 新增 `%LOCALAPPDATA%\Programs\xj`。
4. 要沙盒的話，照上面「Windows」用系統管理員權限把 sandbox-runtime 裝到整台電腦、跑 `windows-install`。
5. 跑 `xj doctor`。金鑰用 `xj setup` 存（存進 Windows 的認證管理員；存不進去的話存成只給你自己讀的金鑰檔），或設環境變數 `XING_JANUS_ANTHROPIC_API_KEY`。

## 更新

Xing Janus 不會自己連網路檢查更新（它對外的連線只有模型的 API）。看 releases 頁面有沒有新版，照上面的步驟下載、驗證、解壓，再把 `xj` 的連結指到新的資料夾。
Windows：先關掉正在跑的 xj，照「Windows 的安裝」的第 3 步再做一次（換掉 `%LOCALAPPDATA%\Programs\xj` 裡的檔案；PATH 已經有了就不會再加）。設定和對話紀錄在 `~/.xing-janus/`，不受影響。

## 移除

```bash
xj reset                                           # 刪掉 ~/.xing-janus（會先列出來、問過再刪）
rm ~/.local/bin/xj && rm -r ~/.local/share/xj-*    # 刪掉程式
security delete-generic-password -s xing-janus-anthropic   # 鑰匙圈裡的金鑰（openai 等也一樣）
```

Windows：`xj reset` 之後，在一般權限的 PowerShell 裡刪掉程式的資料夾、從 PATH 拿掉它（PATH 是你自己帳號的，不要用系統管理員權限跑）：

```powershell
$dir = "$env:LOCALAPPDATA\Programs\xj"
Remove-Item -Recurse -Force $dir
$path = [Environment]::GetEnvironmentVariable('Path', 'User')
[Environment]::SetEnvironmentVariable('Path', (($path -split ';') | Where-Object { $_ -and $_ -ne $dir }) -join ';', 'User')
```

裝過沙盒的話，再在系統管理員權限的 PowerShell 裡：

```powershell
& "$env:ProgramFiles\sandbox-runtime\srt.cmd" windows-uninstall    # 拿掉 srt-sandbox 帳號和防火牆的規則
Remove-Item -Recurse "$env:ProgramFiles\sandbox-runtime"
```

`%ProgramData%\sandbox-runtime` 是程式庫留下的資料夾，要清乾淨的話也刪掉。

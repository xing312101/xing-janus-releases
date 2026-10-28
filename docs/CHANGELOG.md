# 更新說明

版本號照 semver。

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

# Xing Janus（xj）

[English](README.en.md)

在終端機裡跟你對話、幫你寫程式的 coding agent。用你自己的 API key，支援 Anthropic、OpenAI、Gemini 和其他 OpenAI 相容的服務：
本機的 Ollama、LM Studio、llama.cpp、vLLM（不用 key，可以離線），雲端的 OpenRouter 這類。

預設在改檔、跑白名單以外的指令、連新的網域之前先問你；`.env`、私鑰這類機密檔，xj 的工具和沙盒裡的指令都讀不到。指令在作業系統的沙盒裡跑（Windows 的沙盒要另外裝），擋得住什麼、擋不住什麼見[安全說明](docs/cerberus.md)。

> **使用前請知道**
>
> - AI 會出錯：它改的程式碼、要跑的指令，都要自己看過。
> - 只在你信任的程式碼裡使用：檔案裡可能藏著提示注入。
> - Xing Janus 本身不收集任何資料；你打的字、它讀到的檔案內容和指令的輸出，都會傳到你選的模型服務，費用由你的 key 負擔。
> - 沙盒和權限規則能降低風險，但不保證擋下所有問題；因為 AI 造成的任何損失由你自行承擔。
>
> 完整的條款見 [terms.md](docs/terms.md)。

## 安裝

需要：

- macOS、Linux（含 WSL2）或 Windows
- Node 24 以上、ripgrep
- Windows：另外要 Git for Windows，在 Windows Terminal 裡開 xj（Git Bash 自己的視窗只能用陽春的畫面）；沙盒是 alpha，可以不裝：不裝的話每個指令都先問你，你答應了就以你的權限在沙盒外跑

macOS 用 Homebrew 最簡單，Node 和 ripgrep 會一起裝好：

```bash
brew tap xing312101/xing-janus
brew trust --formula xing312101/xing-janus/xj
brew install xj
```

其他平台或不用 Homebrew：照 [install.md](docs/install.md) 下載、驗證、解壓。驗證用的公鑰（minisign，第一次拿到請自己留一份）：

```
RWR8QI6mlwg4yrxuGNlQojUwQGy82MQyHSPAbSMdvTn1+6CEQYe9OWPd
```

## 開始

```bash
xj setup      # 選服務、設金鑰、預設模型、語言、提示音；每一題按 Enter 就用預設值
cd 你的專案
xj
```

第一次直接打 `xj` 也可以：它會先問介面語言、顯示使用提醒，再帶你跑 `xj setup`。第一題選 4 就能接本機或其他 OpenAI 相容的服務（Ollama、LM Studio、llama.cpp、vLLM、OpenRouter…：列出這台找到的，也能自己填網址），本機的不用金鑰；
要金鑰的，已經設好（環境變數、鑰匙圈、Windows 的認證管理員）就直接用，沒有的話請你貼上。
其他設定都有預設值，要改的時候看 [config.md](docs/config.md)。

- **換平台、換模型**：在 xj 裡打 `/model`。這台的 Ollama、LM Studio 會自己找出來，也能新增別的 OpenAI 相容服務；缺金鑰當場貼上，換了就記住。
- **換介面語言**：`/language ui en`、`/language ui zh-TW`。
- **更新**：手動裝的打 `xj update`，驗過 SHA-256 和簽章才換上；Homebrew 裝的用 `brew upgrade xj`（簽章由 tap 更新 formula 之前驗）。從 v0.1.1、v0.1.2 更新的話，先看 [install.md](docs/install.md) 的「更新」。

## 文件

- [快速開始](docs/quickstart.md)：從拿到 API key 到第一次改檔
- [使用說明](docs/usage.md)：模式、權限詢問、斜線指令、狀態列、花費
- [設定檔](docs/config.md)、[疑難排解](docs/troubleshooting.md)（有問題先跑 `xj doctor`）
- [讓別的程式叫 xj（MCP）](docs/mcp-server.md)：Claude Desktop、IDE、你自己寫的通訊軟體橋接；預設唯讀
- [資料在哪、怎麼刪](docs/privacy.md)、[安全說明](docs/cerberus.md)、[回報安全問題](docs/SECURITY.md)
- [更新說明](docs/CHANGELOG.md)、[使用條款](LICENSE)（中文翻譯見 [docs/terms.md](docs/terms.md)）、第三方授權（壓縮檔裡的 `THIRD_PARTY_LICENSES`）

## 原始碼、回報問題

原始碼不公開。這裡的文件和發佈檔由同一個發佈流程推出，所以文件和程式永遠是同一版。

- 用起來有問題、想要新功能：在發佈頁面所在的 repo 開 issue。
- 安全問題請**私下**回報，不要開公開的 issue：見 [SECURITY.md](docs/SECURITY.md)。

Xing Janus 跟 Anthropic、OpenAI、Google 沒有關係，也沒有被它們背書。

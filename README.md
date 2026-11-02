# Xing Janus（xj）

[English](README.en.md)

在終端機裡對話、寫程式的 coding agent。用你自己的 API key（本機的模型不用），支援 Anthropic、OpenAI，和其他 OpenAI 相容的服務：本機（離線）的 Ollama、LM Studio、llama.cpp、vLLM，雲端的 OpenRouter、Gemini 這類。

> **使用前請知道**：AI 會出錯，改的程式碼、要跑的指令都要自己看過；只在你信任的程式碼裡使用（檔案裡可能藏著提示注入）；你送出的內容會傳到你選的模型服務，費用由你的 key 負擔；沙盒和權限規則能降低風險，但不保證擋下所有問題；因為 AI 造成的任何損失由你自行承擔。完整的條款見 [terms.md](docs/terms.md)。

## 安裝

需要 macOS、Linux 或 Windows（Windows 要 Git for Windows，在 Windows Terminal 裡開 xj —— Git Bash 自己的視窗只能用陽春的畫面；沙盒是 alpha，可以不裝。WSL2 也可以），Node 24 以上、ripgrep（`brew install ripgrep`）。macOS 可以用 Homebrew：`brew tap xing312101/xing-janus && brew trust --formula xing312101/xing-janus/xj && brew install xj`（Node 和 ripgrep 一起裝好）。手動下載、驗證、解壓的步驟見 [install.md](docs/install.md)。

發佈者的簽章公鑰（minisign）—— 發佈流程會驗證簽出來的檔案真的能用這把公鑰驗過，才放上來：

```
RWR8QI6mlwg4yrxuGNlQojUwQGy82MQyHSPAbSMdvTn1+6CEQYe9OWPd
```

## 開始

```bash
xj setup      # 設定精靈：用哪一家、金鑰（存進鑰匙圈）、預設模型、語言、提示音
cd 你的專案
xj
```

第一次打 `xj`（還沒有設定檔）的話，會先問介面的語言、顯示使用提醒，同意之後接著跑 `xj setup`。
`xj setup` 每一題直接按 Enter 就用括號裡的預設值：用哪一家 → Anthropic；金鑰 → 已經找得到（環境變數、鑰匙圈、Windows 的認證管理員、`keyFiles`）就照用，找不到才請你貼上（存進鑰匙圈或認證管理員）；
要不要再加一把 → 不加；預設的模型 → 照這把 key 能用的建議一個（Anthropic 是 `anthropic:opus`，同系列最新的）；回覆的語言 → 跟介面一樣；提示音 → 不響。
**其他設定都有預設值，不用先設**：要改的時候看 [config.md](docs/config.md)，每一項都寫了預設值，或 xj 自己怎麼決定。

介面有繁體中文和英文：第一次啟動時會問，之後用 `/language ui zh-TW`、`/language ui en` 換。

換平台、換模型：在 xj 裡打 `/model`，先選平台、再選模型。這台的本機伺服器（Ollama、LM Studio…）會自己找出來，也能新增 OpenAI 相容的服務；
還沒設金鑰的當場貼上（畫面上只顯示 •）。換了就記住，下次開 xj 也用它（見 [使用說明](docs/usage.md) 的「換平台、換模型」）。

- [快速開始](docs/quickstart.md)：從拿到 API key 到第一次改檔
- [使用說明](docs/usage.md)：模式、權限詢問、斜線指令、狀態列、花費
- [設定檔](docs/config.md)、[疑難排解](docs/troubleshooting.md)
- [資料在哪、怎麼刪](docs/privacy.md)、[安全說明](docs/cerberus.md)、[回報安全問題](docs/SECURITY.md)
- [更新說明](docs/CHANGELOG.md)、[使用條款](LICENSE)（中文翻譯見 [docs/terms.md](docs/terms.md)）、第三方授權（壓縮檔裡的 `THIRD_PARTY_LICENSES`）

## 原始碼

原始碼不公開。發佈檔和這份文件由發佈流程從私有的開發 repo 一起推出，所以**文件和程式永遠是同一版**。

- 用起來有問題、想要新功能：在發佈頁面所在的 repo 開 issue。
- 安全問題請**私下**回報，不要開公開的 issue：見 [SECURITY.md](docs/SECURITY.md)。

Xing Janus 跟 Anthropic、OpenAI、Google 沒有關係，也沒有被它們背書。

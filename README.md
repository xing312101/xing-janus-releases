# Xing Janus（xj）

[English](README.en.md)

在終端機裡對話、寫程式的 coding agent。用你自己的 API key，支援 Anthropic、OpenAI、Ollama（本機）和其他 Chat Completions 相容的服務。

> **使用前請知道**：AI 會出錯，改的程式碼、要跑的指令都要自己看過；只在你信任的程式碼裡使用（檔案裡可能藏著提示注入）；你送出的內容會傳到你選的模型服務，費用由你的 key 負擔；沙盒和權限規則能降低風險，但不保證擋下所有問題；因為 AI 造成的任何損失由你自行承擔。完整的條款見 [terms.md](docs/terms.md)。

## 安裝

需要 macOS、Linux 或 Windows（Windows 要 Git for Windows；沙盒是 alpha，可以不裝。WSL2 也可以），Node 24 以上、ripgrep（`brew install ripgrep`）。下載、驗證、解壓的步驟見 [install.md](docs/install.md)。

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

介面有繁體中文和英文：第一次啟動時會問，之後用 `/language ui zh-TW`、`/language ui en` 換。

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

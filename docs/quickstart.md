# 快速開始：從拿到 API key 到第一次改檔

## 1. 準備一把 API key

Xing Janus 用你自己的 API key，不用任何一家的訂閱登入（ChatGPT、Claude 的訂閱都不含 API）。

| 廠商 | 去哪裡申請 | 注意 |
|---|---|---|
| Anthropic | console.anthropic.com | 開一個專用的 key；建議設好用量上限 |
| OpenAI | platform.openai.com | 開一個專用的 Project 和 key，設月預算 |
| Ollama | 不用 key | 裝好 Ollama，用 `OLLAMA_CONTEXT_LENGTH=32768` 啟動（預設的 4096 太小） |

## 2. 跑設定精靈

```bash
xj setup
```

它會依序問：用哪一家、金鑰、預設的模型、回覆的語言、要不要提示音（預算不問，要改就改設定檔）。金鑰不會顯示在畫面上，也不會留在指令歷史裡：
macOS 由 `security` 指令請你貼上、存進鑰匙圈；Windows 存進認證管理員；其他的存成只給你自己讀的金鑰檔。最後會自動跑 `xj doctor`。

第一次打 `xj` 時會先問介面的語言、顯示使用提醒，同意之後自動跑設定精靈。

## 3. 第一次對話

```bash
cd ~/你的專案
xj
```

- 第一次在某個資料夾開，會問「信任這個資料夾嗎？」。只有你信任的資料夾，白名單裡的指令（測試、`git status`）才會直接跑，AGENTS.md 和專案裡的技能才會載入。別人的 repo 先選「否」。
- 打字問它，例如「README 的第二行改成…」。它會先讀檔，再秀出 diff 問你「套用？」，按 `y` 才改。
- 改錯了打 `/undo` 還原；`/help` 看全部的指令。
- 介面要換成英文：`/language ui en`（換回來是 `/language ui zh-TW`）。

# 快速開始：從拿到 API key 到第一次改檔

還沒裝的話，先照 [install.md](install.md) 裝好（macOS 可以用 Homebrew，Node 和 ripgrep 一起裝好）。

## 1. 準備一把 API key

Xing Janus 用你自己的 API key，不用任何一家的訂閱登入（ChatGPT、Claude 的訂閱都不含 API）。

| 廠商 | 去哪裡申請 | 注意 |
|---|---|---|
| Anthropic | console.anthropic.com | 開一個專用的 key；建議設好用量上限 |
| OpenAI | platform.openai.com | 開一個專用的 Project 和 key，設月預算 |
| Ollama | 不用 key | 裝好 Ollama，用 `OLLAMA_CONTEXT_LENGTH=32768` 啟動（預設的 4096 太小） |
| 其他 OpenAI 相容的服務 | 看那個服務 | 本機的 LM Studio、llama.cpp、vLLM 不用 key；雲端的 OpenRouter 這類要 key。在 xj 裡用 `/model` 選或新增（見第 3 步） |

同一家要分開用好幾把 key 的話，見 [config.md](config.md) 的「同一家好幾把 key」。

## 2. 跑設定精靈

```bash
xj setup
```

它會依序問：用哪一家、金鑰、要不要再加一把、預設的模型、回覆的語言、要不要提示音（預算不問，要改就改設定檔）。金鑰不會顯示在畫面上，也不會留在指令歷史裡：
macOS 由 `security` 指令請你貼上、存進鑰匙圈；Windows 存進認證管理員；其他的存成只給你自己讀的金鑰檔。最後會自動跑 `xj doctor`。

第一次打 `xj` 時會先問介面的語言、顯示使用提醒，同意之後自動跑設定精靈。

## 3. 第一次對話

```bash
cd ~/你的專案
xj
```

- 第一次在某個資料夾開，會問「信任這個資料夾嗎？」。只有你信任的資料夾，白名單裡的指令（測試、`git status`）才會直接跑，AGENTS.md 和專案裡的技能才會載入。別人的 repo 先選「否」。
- 打字問它，例如「README 的第二行改成…」。它會先讀檔，再秀出 diff 問你「套用？」，按 `y` 才改（整個 diff 都看過的話，按 Enter 也算「是」）。
- 改錯了打 `/undo` 還原；`/help` 看全部的指令。
- 換平台、換模型：打 `/model`，先選平台、再選模型。這台的本機伺服器（Ollama、LM Studio…）會自己找出來，還沒設金鑰的當場貼上（畫面上只顯示 •）；
  換了就記住，下次開 xj 也用它（見 [usage.md](usage.md) 的「換平台、換模型」）。
- 介面要換成英文：`/language ui en`（換回來是 `/language ui zh-TW`）。

## 4. 之後更新

有新版時打 `xj update`（Homebrew 裝的用 `brew upgrade xj`）：只裝比現在新的版本，SHA-256 和簽章都驗過才換上。見 [install.md](install.md) 的「更新」。

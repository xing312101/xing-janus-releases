# Quick start: from an API key to your first edit

## 1. Get an API key

Xing Janus uses your own API key; it does not sign in with any subscription (ChatGPT and Claude subscriptions do not include the API).

| Provider | Where to get one | Notes |
|---|---|---|
| Anthropic | console.anthropic.com | Create a dedicated key; set a spending limit |
| OpenAI | platform.openai.com | Create a dedicated Project and key, and set a monthly budget |
| Ollama | No key needed | Install Ollama and start it with `OLLAMA_CONTEXT_LENGTH=32768` (the default 4096 is too small) |

## 2. Run the setup wizard

```bash
xj setup
```

It asks, in order: which provider, the key, the default model, the language for AI replies, and whether to beep. The key is stored without appearing on screen or in your command history:
on macOS the `security` command asks you to paste it into the keychain; on Windows it goes into Credential Manager; elsewhere into a key file only you can read. At the end it runs `xj doctor` for you.

The first time you type `xj`, it asks for the interface language, shows a notice, and runs the setup wizard after you accept.

## 3. Your first conversation

```bash
cd ~/your-project
xj
```

- The first time you start it in a folder, it asks "Trust this folder?". Only in folders you trust do whitelisted commands (tests, `git status`) run directly, and AGENTS.md and the project's skills get loaded. Answer no for someone else's repo at first.
- Type a request, for example "change the second line of the README to …". It reads the file first, then shows the diff and asks "Apply?"; nothing changes until you press `y`.
- If it got something wrong, type `/undo` to restore; `/help` lists all commands.
- To switch the interface between English and Chinese, use `/language ui en` or `/language ui zh-TW`.

# Quick start: from an API key to your first edit

If you have not installed it yet, follow [install.md](install.md) first (on macOS you can use Homebrew, which installs Node and ripgrep too).

## 1. Get an API key

Xing Janus uses your own API key; it does not sign in with any subscription (ChatGPT and Claude subscriptions do not include the API).

| Provider | Where to get one | Notes |
|---|---|---|
| Anthropic | console.anthropic.com | Create a dedicated key; set a spending limit |
| OpenAI | platform.openai.com | Create a dedicated Project and key, and set a monthly budget |
| Gemini | aistudio.google.com | Create a key restricted to the Gemini API; for code you do not want used to improve products or reviewed by people, use a project with billing (free-tier content is, see "Gemini" in config.md); set a budget alert in Cloud |
| Ollama | No key needed | Install Ollama and start it with `OLLAMA_CONTEXT_LENGTH=32768` (the default 4096 is too small) |
| Other OpenAI-compatible services | Depends on the service | Local ones (LM Studio, llama.cpp, vLLM) need no key; cloud ones such as OpenRouter do. Choose 4 in the first question of the setup wizard, or pick or add them with `/model` inside xj (see step 3) |

To keep several keys for one provider apart, see "Several keys for one provider" in [config.md](config.md).

## 2. Run the setup wizard

```bash
xj setup
```

It asks, in order: which provider (1 Anthropic, 2 OpenAI, 3 Gemini, 4 a local or other OpenAI-compatible service: choice 4 lists Ollama and the local servers found on this computer, with a last item for entering an address, then asks for a name and whether it needs a key), the key, whether to add another key, the default model, the language for AI replies, and whether to beep (it does not ask about the budget; change that in the config file). The key is stored without appearing on screen or in your command history:
on macOS the `security` command asks you to paste it into the keychain; on Windows it goes into Credential Manager; elsewhere into a key file only you can read. At the end it runs `xj doctor` for you.

The first time you type `xj`, it asks for the interface language, shows a notice, and runs the setup wizard after you accept.

## 3. Your first conversation

```bash
cd ~/your-project
xj
```

- The first time you start it in a folder, it asks "Trust this folder?". Only in folders you trust do whitelisted commands (tests, `git status`) run directly, and AGENTS.md and the project's skills get loaded. Answer no for someone else's repo at first.
- Type a request, for example "change the second line of the README to …". It reads the file first, then shows the diff and asks "Apply?"; nothing changes until you press `y` (once you have seen the whole diff, Enter counts as yes too).
- If it got something wrong, type `/undo` to restore; `/help` lists all commands.
- To switch platforms or models, type `/model`: pick a platform, then a model. Local servers on this computer (Ollama, LM Studio…) are found for you, and if a key is missing you paste it right there (only • appears on screen);
  the choice is remembered, so the next xj starts with it too (see "Switching platforms and models" in [usage.md](usage.md)).
- To switch the interface between English and Chinese, use `/language ui en` or `/language ui zh-TW`.

## 4. Updating later

When a new version is out, type `xj update` (if you installed with Homebrew, use `brew upgrade xj`): it only installs a newer version, and only after the SHA-256 and the signature check out. See "Update" in [install.md](install.md).

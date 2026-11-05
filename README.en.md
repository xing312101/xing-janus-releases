# Xing Janus (xj)

[繁體中文](README.md)

A coding agent you talk to in the terminal. It uses your own API key and supports Anthropic, OpenAI, Gemini and other OpenAI-compatible services:
local ones such as Ollama, LM Studio, llama.cpp and vLLM (no key needed, works offline), and hosted ones such as OpenRouter.

By default it asks you before editing files, running commands outside the whitelist or connecting to a new domain; neither xj's tools nor sandboxed commands can read secret files such as `.env` and private keys. Commands run in the operating system's sandbox (on Windows, once you install it); for what it does and does not stop, see [Security](docs/en/cerberus.md).

> **Before you use it**
>
> - AI makes mistakes: review the code it changes and the commands it wants to run.
> - Only use it on code you trust: files can hide prompt injection.
> - Xing Janus itself collects no data; what you type, the files it reads and command output all go to the model service you chose, and the cost is charged to your key.
> - The sandbox and permission rules reduce risk but do not guarantee that every problem is stopped; any loss caused by the AI is your own responsibility.
>
> The full terms are in [LICENSE](LICENSE).

## Install

You need:

- macOS, Linux (including WSL2) or Windows
- Node 24 or later, and ripgrep
- Windows: also Git for Windows, and run xj in Windows Terminal (Git Bash's own window only gets a bare screen); the sandbox is alpha and optional: without it every command asks you first, and once you approve, it runs outside any sandbox with your permissions

On macOS, Homebrew is the easiest way, and it installs Node and ripgrep too:

```bash
brew tap xing312101/xing-janus
brew trust --formula xing312101/xing-janus/xj
brew install xj
```

Other platforms, or without Homebrew: download, verify and unpack as in [install.md](docs/en/install.md). The public key for verifying (minisign; keep your own copy the first time you get it):

```
RWR8QI6mlwg4yrxuGNlQojUwQGy82MQyHSPAbSMdvTn1+6CEQYe9OWPd
```

## Getting started

```bash
xj setup      # pick a service, set the key, default model, language, sound; Enter takes each default
cd your-project
xj
```

You can also just type `xj` the first time: it asks for the interface language, shows the usage notice, then takes you through `xj setup`. Choice 4 of its first question connects a local or other OpenAI-compatible service (Ollama, LM Studio, llama.cpp, vLLM, OpenRouter…: it lists the ones found on this computer, or you enter an address); local ones need no key.
For a provider that needs one, a key that is already set (environment variable, keychain, Windows Credential Manager) is used as is; otherwise it asks you to paste one.
Every other setting has a default; to change one, see [config.md](docs/en/config.md).

- **Switching platforms and models**: type `/model` inside xj. Servers on this computer (Ollama, LM Studio…) are found for you, and you can add other OpenAI-compatible services; a missing key is pasted right there, and the choice is remembered.
- **Interface language**: `/language ui en` or `/language ui zh-TW`.
- **Updating**: if you installed by hand, `xj update` only installs after the SHA-256 and the signature check out; with Homebrew, use `brew upgrade xj` (the tap checks the signature before updating the formula). When updating from v0.1.1 or v0.1.2, first read "Update" in [install.md](docs/en/install.md).

## Documents

- [Quick start](docs/en/quickstart.md): from getting an API key to your first edit
- [Usage](docs/en/usage.md): modes, permission questions, slash commands, the status bar, cost
- [The config file](docs/en/config.md), [Troubleshooting](docs/en/troubleshooting.md) (run `xj doctor` first when something is wrong)
- [Letting other programs call xj (MCP)](docs/en/mcp-server.md): Claude Desktop, IDEs, a chat-app bridge you write; read-only by default
- [Where your data is and how to delete it](docs/en/privacy.md), [Security](docs/en/cerberus.md), [Reporting security issues](docs/en/SECURITY.md)
- [Changelog](docs/en/CHANGELOG.md), [Terms of use](LICENSE), third-party licenses (`THIRD_PARTY_LICENSES` in the archive)

## Source code and feedback

The source code is not public. These documents and the release files are published by the same release pipeline, so the documents always match the program version.

- Problems or feature requests: open an issue in the repo that hosts the releases page.
- Report security issues **privately**, not in a public issue: see [SECURITY.md](docs/en/SECURITY.md).

Xing Janus is not affiliated with or endorsed by Anthropic, OpenAI or Google.

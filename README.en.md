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
- **Changing settings**: type `/config` inside xj (the common settings, each saying what changing it does); or just ask xj "I need docker, what should I set?": it answers from your current settings and can propose changes, written to the config only if you approve ("Changing settings" in [usage](docs/en/usage.md)).
- **Working on another folder**: `/add-dir ../api` asks whether it is read-only or editable too (editable ones must be trusted and follow the same rules as the working folder); folders to add every time you open xj in this project go in `permissions.addDirs` in the config.
- **Seeing permissions**: `/permissions` first shows the current mode, sandbox and folders, then lists what you always allowed (removable), what the config changed, and the built-in rules.
- **Budget**: by default $5 per turn, $30 per day and $100 per month, and at a limit xj stops and asks you; `/budget` shows what was spent and changes a limit (for example `/budget day 50`).
- **Long commands**: a command is stopped after 2 minutes by default (the model can ask for up to 10 minutes; `/config` changes the limits); for whole test suites or dev servers, ask it to run them in the background and use `/jobs` to see the output or stop them.
- **Updating**: if you installed by hand, `xj update` only installs after the SHA-256 and the signature check out; with Homebrew, use `brew upgrade xj` (the tap checks the signature before updating the formula). When updating from v0.1.1 or v0.1.2, first read "Update" in [install.md](docs/en/install.md).

## Sandbox and modes

By default **the sandbox is on and the mode is normal**: commands run in the operating system's sandbox (reaching only the working folder and folders you add), edits show a diff and ask, whitelisted commands run directly, and the rest ask. Most people do not need to change anything.

| | Sandbox | Mode |
|---|---|---|
| Decides | What can be reached: whether commands are confined, the scope the tools can read and edit | Whether xj asks you first |
| Options | On, off | Read-only, plan, normal, auto-edit, full-auto, skip all permissions |
| Default | On | Normal |
| Change | In xj, `/sandbox off`, `/sandbox on`; `sandbox.enabled` in the config, `--no-sandbox` on the command line | In xj, Shift-Tab, `/mode`; `defaultMode` in the config, `--auto`, `--dangerously-skip-permissions` on the command line |

| Your situation | Sandbox | Mode |
|---|---|---|
| Everyday development | On | Normal (or auto-edit) |
| You trust the code and want fewer questions | On | Full-auto |
| Let it run for a while without stopping for you | On | Skip all permissions |
| You need docker or flutter | On: list docker in `excludedCommands`; flutter runs outside the sandbox by itself (asking each time) | As above |
| A throwaway container or virtual machine | Off (if the sandbox cannot start there, write `sandbox.enabled: false` in that machine's config) | Skip all permissions |

For how each mode behaves with the sandbox on and off, see "Sandbox and modes" in the [usage guide](docs/en/usage.md); for the settings to write in each situation, see [What to set](docs/en/setup-guide.md).

## Documents

- [Quick start](docs/en/quickstart.md): from getting an API key to your first edit
- [Usage](docs/en/usage.md): sandbox and modes, permission questions, added folders, slash commands, the status bar, background jobs, cost and budget, changing settings
- [What to set](docs/en/setup-guide.md): set the sandbox and mode for your situation (Docker, other folders, fewer questions, letting it run, containers)
- [The config file](docs/en/config.md), [Troubleshooting](docs/en/troubleshooting.md) (run `xj doctor` first when something is wrong)
- [Letting other programs call xj (MCP)](docs/en/mcp-server.md): Claude Desktop, IDEs, a chat-app bridge you write; read-only by default
- [Where your data is and how to delete it](docs/en/privacy.md), [Security](docs/en/cerberus.md), [Reporting security issues](docs/en/SECURITY.md)
- [Changelog](docs/en/CHANGELOG.md), [Terms of use](LICENSE), third-party licenses (`THIRD_PARTY_LICENSES` in the archive)

## Source code and feedback

The source code is not public. These documents and the release files are published by the same release pipeline, so the documents always match the program version.

- Problems or feature requests: open an issue in the repo that hosts the releases page.
- Report security issues **privately**, not in a public issue: see [SECURITY.md](docs/en/SECURITY.md).

Xing Janus is not affiliated with or endorsed by Anthropic, OpenAI or Google.

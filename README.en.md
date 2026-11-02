# Xing Janus (xj)

[繁體中文](README.md)

A coding agent you talk to in the terminal. It uses your own API key (local models need none) and supports Anthropic, OpenAI, and other OpenAI-compatible services: local (offline) ones such as Ollama, LM Studio, llama.cpp and vLLM, and hosted ones such as OpenRouter and Gemini.

> **Before you use it**: AI makes mistakes, so review the code it changes and the commands it wants to run; only use it on code you trust (files can hide prompt injection); what you send goes to the model service you chose, and the cost is charged to your key; the sandbox and permission rules reduce risk but do not guarantee that every problem is stopped; any loss caused by the AI is your own responsibility. The full terms are in [LICENSE](LICENSE).

## Install

Needs macOS, Linux or Windows (Windows needs Git for Windows; run xj in Windows Terminal, since Git Bash's own window only gets a bare screen; the sandbox there is alpha and optional; WSL2 also works), Node 24 or later, and ripgrep (`brew install ripgrep`). On macOS you can use Homebrew: `brew tap xing312101/xing-janus && brew trust --formula xing312101/xing-janus/xj && brew install xj` (Node and ripgrep come with it). Steps for downloading, verifying and unpacking by hand: [install.md](docs/en/install.md).

The publisher's signing public key (minisign) — the release pipeline checks that the signed files really verify with this key before publishing them:

```
RWR8QI6mlwg4yrxuGNlQojUwQGy82MQyHSPAbSMdvTn1+6CEQYe9OWPd
```

## Getting started

```bash
xj setup      # setup wizard: provider, key (stored in the keychain), default model, language, sound
cd your-project
xj
```

The first time you type `xj` (with no config file yet), it asks for the interface language, shows the usage notice, and once you agree, runs `xj setup`.
At each `xj setup` question, pressing Enter takes the default in brackets: provider → Anthropic; key → if one is already found (environment variable, keychain, Windows Credential Manager, `keyFiles`) it is used, otherwise you paste one (stored in the keychain or Credential Manager);
add another key → no; default model → one suggested from the models this key can use (for Anthropic, `anthropic:opus`: the newest in that series); reply language → the same as the interface; sound → off.
**Every other setting has a default, so there is nothing else to set first**: when you want to change something, see [config.md](docs/en/config.md), which gives each setting's default or how xj decides it.

The interface is in English or Traditional Chinese: xj asks on the first start, and `/language ui en` or `/language ui zh-TW` switches it.

Switching platforms and models: type `/model` inside xj to pick a platform, then a model. Servers on this computer (Ollama, LM Studio…) are found
for you, and you can add an OpenAI-compatible service; a missing key is pasted right there (only • is shown). The choice is remembered for next time
(see "Switching platforms and models" in [Usage](docs/en/usage.md)).

- [Quick start](docs/en/quickstart.md): from getting an API key to your first edit
- [Usage](docs/en/usage.md): modes, permission questions, slash commands, the status bar, cost
- [The config file](docs/en/config.md), [Troubleshooting](docs/en/troubleshooting.md)
- [Where your data is and how to delete it](docs/en/privacy.md), [Security](docs/en/cerberus.md), [Reporting security issues](docs/en/SECURITY.md)
- [Changelog](docs/en/CHANGELOG.md), [Terms of use](LICENSE), third-party licenses (`THIRD_PARTY_LICENSES` in the archive)

## Source code

The source code is not public. The release files and these documents are published together by the release pipeline from the private development repo, so **the documents always match the program version**.

- Problems or feature requests: open an issue in the repo that hosts the releases page.
- Report security issues **privately**, not in a public issue: see [SECURITY.md](docs/en/SECURITY.md).

Xing Janus is not affiliated with or endorsed by Anthropic, OpenAI or Google.

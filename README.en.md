# Xing Janus (xj)

[繁體中文](README.md)

A coding agent you talk to in the terminal. It uses your own API key and supports Anthropic, OpenAI, Ollama (local) and other Chat Completions-compatible services.

> **Before you use it**: AI makes mistakes, so review the code it changes and the commands it wants to run; only use it on code you trust (files can hide prompt injection); what you send goes to the model service you chose, and the cost is charged to your key; the sandbox and permission rules reduce risk but do not guarantee that every problem is stopped; any loss caused by the AI is your own responsibility. The full terms are in [LICENSE](LICENSE).

## Install

Needs macOS, Linux or Windows (Windows needs Git for Windows; the sandbox there is alpha and optional; WSL2 also works), Node 24 or later, and ripgrep (`brew install ripgrep`). Steps for downloading, verifying and unpacking: [install.md](docs/en/install.md).

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

The interface is in English or Traditional Chinese: xj asks on the first start, and `/language ui en` or `/language ui zh-TW` switches it.

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

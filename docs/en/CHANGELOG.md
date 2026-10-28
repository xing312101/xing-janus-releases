# Changelog

Versions follow semver.

## 0.1.0

The first version.

**Requires**: macOS, Linux or Windows; Node 24 or later (24 is the active LTS; 26 also works); `ripgrep` (`brew install ripgrep`).
On Linux, also install bubblewrap, socat, and sandbox-runtime for the sandbox (not included in the archive; see `docs/en/install.md`):
`npm install -g --ignore-scripts @anthropic-ai/sandbox-runtime@0.0.79`.
Windows needs Git for Windows (the bash tool uses Git Bash); the sandbox is alpha, needs a one-time install with administrator rights, and is optional (without it every command asks you first).
WSL2 also works (the Linux version runs inside it).

**Verify after downloading** (keep the two files next to the archive):

```bash
shasum -a 256 -c xj-0.1.0-darwin.tar.gz.sha256
minisign -Vm xj-0.1.0-darwin.tar.gz -P <the public key in the README>
```

The Linux files are `xj-0.1.0-linux-x64.tar.gz` or `xj-0.1.0-linux-arm64.tar.gz` (pick by CPU); check the SHA-256 with `sha256sum -c`.
Windows is `xj-0.1.0-windows.zip`; verify it in PowerShell:

```powershell
(Get-FileHash xj-0.1.0-windows.zip -Algorithm SHA256).Hash.ToLower()   # must match the .sha256 file
minisign -Vm xj-0.1.0-windows.zip -P <the public key in the README>
```

Keep your own copy of the public key and verify every later update with it (if you only ever take it from here, tampering here would swap the key too).
Unpack as described in `docs/en/install.md` and put `xj` on your PATH (macOS's default PATH has no `~/.local/bin`; that document covers each shell and Windows),
then run `xj setup` and type `xj` in your project.

What is in it:

- Conversation, reading, searching and editing files, running commands; diffs shown before edits, `/undo` to restore
- Cerberus: permission rules, the OS sandbox (macOS, Linux; Windows is alpha), trusted folders, secret files, dangerous commands
- A native Windows version: the bash tool uses Git Bash (with the same permission checks as macOS and Linux), plus a powershell tool (its whitelist only accepts very simple read-only forms)
- Windows: tools on your PATH that cannot run inside the sandbox (installed under your account, or installed for the whole computer by winget as administrator) are named at start-up and when a command fails, with what to do; the model does not treat them as missing
- Models: Anthropic (native), OpenAI, Ollama and other Chat Completions-compatible services; switch mid-conversation
- Summarizing (`/compact`), resuming earlier conversations (`/resume`), checks after edits (`/checks`), plan mode, full-auto mode (`--auto`), attaching files and images
- Commands that run outside the sandbox (`sandbox.excludedCommands` in the config, for example `docker`): they ask every time and only run outside the sandbox after you say yes; empty by default
- Skip-all-permissions mode (`--dangerously-skip-permissions`): no sandbox and nothing asks; only secret files and protected files stay blocked, and switching to root (`sudo`) still asks;
  you confirm by typing on every start, and it cannot start as root. Only recommended in a container or VM
- Windows: when the sandbox start finds folders on your PATH that the sandbox account can write to (including entries pointing to folders that do not exist), xj asks whether to block them
  as administrator (UAC asks too; you can allow it permanently and remove that with `/permissions`); in skip-all-permissions mode it blocks them without asking.
  If programs that run outside the sandbox (git, Git Bash, tools such as flutter, MCP servers…) are in those folders, xj also asks once whether to run them anyway
  (`sandbox.writablePrograms` in the config)
- Agent Skills, MCP (only servers listed in the config; they run in the sandbox by default)
- Cost and budget, `xj doctor`, `xj setup`, `xj reset`
- The screen: full-screen mode (default; `/tui classic` switches to classic mode), every line labelled with who is speaking (You / AI / xj), the to-do list and current status on the right in wide windows (clickable), select-and-copy by dragging (Ctrl-C while text is highlighted copies instead of interrupting)
- The interface in English or Traditional Chinese (chosen on the first start, switched with `/language ui`); instructions and tool results for the model are always in English
- To-do lists: the model writes them for multi-step work, numbered; add, change or remove items just by saying so
- Typing while the model works: your message is handed over after its current step
- Fetching web pages (`web_fetch`): domains share the sandbox's list and new ones ask; local, private and cloud metadata addresses are blocked without asking. No search yet
- Subagents: explore for finding things (read-only, can run in parallel) and general for edits (asks according to your mode); every action still goes through Cerberus, and xj works out what changed for the main agent to check
- Always allow (written into the config, for every project), viewed and removed with `/permissions`
- Enter means yes at questions (`enterToApprove` in the config, on by default; dangerous commands, content not fully viewed and always-allow confirmations still need y)

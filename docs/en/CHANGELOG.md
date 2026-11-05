# Changelog

Versions follow semver.

## 0.1.4

**Requires**: the same as 0.1.3 — macOS, Linux or Windows; Node 24 or later (on Windows, 24.20.0 or later is recommended); `ripgrep`. On Linux also bubblewrap, socat and sandbox-runtime 0.0.79;
on Windows, Git for Windows (see `docs/en/install.md`).

**Updating from 0.1.3**: run `xj update` (if you installed with Homebrew, use `brew upgrade xj`). Your config, keys and transcripts stay as they are.
- The numbers in the first question of `xj setup` changed: 3 is Gemini and 4 is a local or other OpenAI-compatible service (in 0.1.3, 3 was Ollama).
- If you connect a service on another machine over unencrypted `http://` and it needs a key: from this version on the key is not sent (it would go in plain text).
  Use https; if you are sure, add `"allowPlainHttpKey": true` to `providers.<name>` in the config (see "Connecting other model services" in `docs/en/config.md`).
- If your `~/.xing-janus/prices.json` was created by an older version, it has no Gemini prices (costs show as "unknown"): `xj doctor` says how to replace it.

If you are still on 0.1.1 or 0.1.2, read the 0.1.3 section below first (the update is done by the old version's code, and there are a few things to avoid).

**Verify after downloading** (keep the two files next to the archive):

```bash
shasum -a 256 -c xj-0.1.4-darwin.tar.gz.sha256
minisign -Vm xj-0.1.4-darwin.tar.gz -P <your saved copy of the public key>
```

The Linux files are `xj-0.1.4-linux-x64.tar.gz` or `xj-0.1.4-linux-arm64.tar.gz` (check the SHA-256 with `sha256sum -c`); Windows is `xj-0.1.4-windows.zip`
(verify it as in steps 1–2 of "Installing on Windows" in `docs/en/install.md`, with your saved copy of the public key). Then check that the `Trusted comment:` minisign prints
reads `xing-janus v0.1.4 xj-0.1.4-<platform>…`; if not, do not install it (an old version renamed together with its signature passes both lines above).
If this is your first install, follow `docs/en/install.md`, then run `xj setup`.

New:

- `xj mcp`: other programs (Claude Desktop, IDEs, a chat-app bridge you write) can call xj through MCP ([mcp-server.md](mcp-server.md)). What it may do follows `mcpServer` in the config:
  read-only by default, nothing outside the sandbox, only the folder it was started in, spending $2 a turn / $10 a day (it stops at the limit); when relaxed, approvals are asked through MCP elicitation (keys masked, only "allow once" and "deny").
  The sandbox is always required and the folder must have been trusted at the computer; `@path` mentions in the text it is given do not attach files
- Gemini has a native adapter (it used to go through Google's OpenAI-compatible endpoint): choose 3 in `xj setup`, or set `XING_JANUS_GEMINI_API_KEY`; the new keys starting with `AQ.` work too
  and are masked in transcripts. Thinking is sent back as received, so multi-turn tool calls do not break. A `providers.gemini` with a `baseURL` in the config keeps using the compatible endpoint.
  With a free-tier key, Google uses what you send to improve its products (see "Gemini" in `docs/en/config.md`)
- Choice 4 in the first question of `xj setup`: a local or other OpenAI-compatible service (Ollama, LM Studio, llama.cpp, vLLM, OpenRouter…). It lists Ollama and the local servers found on this computer,
  with a last item for entering an address (or paste one right away), then asks for a name and whether a key is needed, so people who only use local models can finish the setup wizard
  (before, you needed an Anthropic or OpenAI key to start xj and get to `/model`). Enter picks Ollama; the other servers found need their number
  (on a shared computer, someone else's program may be listening on those ports)
- Price table: limited-time prices (`until`, `after`), switched to the later price by date; the default price table includes the main Gemini models
- The default limit of tool calls per turn went from 50 to 100 (`limits.toolCallsPerTurn` in the config)

Fixed:

- Security: keys are not sent over unencrypted http to another machine. Before, only the new key that `xj setup` or `/model` was about to store was held back; a key the same name already had
  (keychain, key file, environment variable) still went to the new address in plain text (for example when re-running the setup with an `s` missing from the address). Starting xj, switching to it
  inside xj, setting a key there, switching keys with `/key` and `xj doctor` all hold it back (see "Updating from 0.1.3" above for the setting)
- Security: when `xj setup` or `/model` adds a service, the address is checked the way the config checks it: forms such as `HTTP://`, `http:/` and `http:\\` used to send the key in plain text
  first and only fail at the config check at the end; addresses with a user name or password are refused
- Security: platform names may not contain `--` (`anthropic--work` overwrote the Anthropic key named work) or be reserved names such as `prototype` and `con`;
  `xj setup` checks the whole config before storing a key, and writes the config to a temporary file first, replacing the old one only after it checks out (before, a broken one made every later start report a config error)
- Security: Gemini turns off the SDK's automatic continuation (when the service returned a continuation token, the SDK sent the whole conversation again by itself, billed every time, out of the budget's reach,
  and retried by itself after a dropped connection): a reply with a token counts as cut off. Tool arguments are checked like the other two providers do (keys such as `__proto__` are dropped)
- Security: OpenAI-compatible services (OpenAI, Ollama, OpenRouter…) no longer follow redirects either: a cross-site redirect used to send the whole conversation to the new place (not the key).
  When Gemini or an OpenAI-compatible service answers with a redirect, the message says so, as with Anthropic (it used to say it could not connect)
- Security: every row of `xj doctor` and the first line of the start screen turn line breaks from services and folder names into spaces (they could forge whole rows, such as a fake "Sandbox: blocks");
  the first line and the "This folder" row mark bidi controls in folder names (they used to be printed as is, reversing the rest of the line and pushing the right-hand column in Windows Terminal)
- Security: when step 3 of `xj setup` cannot reach a service, keys are masked in the reason (a service that put the Authorization header it received in its error message had the whole key printed);
  listing the models waits at most 15 seconds (it used to wait forever if the service never answered)
- Security: addresses ending in `.localhost` no longer count as this computer (Node resolves them through DNS): `/model` asks before switching to one
- Windows: the sandbox sometimes failed to start ("could not bind a loopback listener outside the WFP permit range": the sandbox library tried only 5 ephemeral ports) → xj restarts it, up to 3 times
- Windows: the Microsoft Store version of PowerShell 7 is found (used without the sandbox; with the sandbox on, the one in Program Files or the built-in 5.1 is used as before); the message about skipping PowerShell 7 is corrected.
  The programs it starts are not in xj's job object (Store apps let child processes leave the job Windows gives them): on Ctrl-C or timeout they are stopped by parent, and programs left behind after the command ends are not (cerberus.md).
  Execution aliases count only when they point into `%ProgramFiles%\WindowsApps` (security: an alias that a sandboxed command created in the working folder would otherwise be chosen later without the sandbox)
- Price table: a limited-time entry whose regular price has a long-context rate but whose `after` has none is not used (after `until`, the long-context surcharge used to disappear and costs came out too low)
- `xj setup`: when no model can be suggested (cannot connect, or the list is empty), there is no default like `ollama:` (the config rejects it, so pressing Enter failed the whole wizard);
  an answer that is not in provider:model form is explained and asked again, and after three tries nothing is saved. The message when a service cannot be reached no longer ends with an extra period
- English `xj doctor`: when a row joins two sentences, the second one starts in lowercase (it used to print "v0.1.3; The last…")
- In the classic screen, making the window narrower made the terminal fold the old status bar into two rows; the top half stayed on screen (later output did not cover it) and, when widening again, was joined back and pushed into the middle: it is cleaned up now.
  Only in terminals xj can tell refold lines like this (Windows Terminal, tmux, Terminal.app, iTerm2, VS Code, VTE-based ones such as GNOME Terminal, kitty, WezTerm, Ghostty, Alacritty, foot);
  the old console host (conhost), terminals that do not refold such as xterm, and terminals xj cannot identify over SSH are unchanged (cleaning up there would delete the last lines of the conversation)
- After `/model` switches, the platform is named too ("Now using Claude Sonnet 5.5 (anthropic, series sonnet)"); the `/resume` list shows `platform:model` and resuming names the platform: models with the same name on different platforms can be told apart
- `budget.unknownRequestsPerTurn` in the config file: after how many requests with no known price in one turn xj stops and asks (still 10 by default; 0 turns it off, for example with a local service). The question itself now says how to turn it off
- Help text shown while typing in xj no longer starts a row with closing punctuation (a closing bracket or the Chinese full stop) or ends a row with an opening bracket when it wraps
- User docs: README, quick start, `config.md` (Gemini, limited-time prices, choice 4 of `xj setup`, `allowPlainHttpKey`, platform names), usage, privacy, troubleshooting;
  `install.md` says Windows on arm64 has not been tested yet, and the Windows install and uninstall lines that edit PATH keep it as written (forms like `%USERPROFILE%` and the value type `REG_EXPAND_SZ` stay; the old lines turned it into `REG_SZ`)

## 0.1.3

**Requires**: the same as 0.1.2 — macOS, Linux or Windows; Node 24 or later (on Windows, 24.20.0 or later is recommended); `ripgrep`. On Linux also bubblewrap, socat and sandbox-runtime 0.0.79;
on Windows, Git for Windows (see `docs/en/install.md`).

**Updating from 0.1.1 or 0.1.2**: run `xj update` (if you installed with Homebrew, use `brew upgrade xj`). The update is done by the old version's code, so some of the fixes below still bite this once:
- macOS, Linux (installed by hand): **do not run it in your home folder**; run it from another folder, such as one of your projects. Run in the home folder, `xj update` in 0.1.1 and 0.1.2
  installs the new version but leaves the `xj` link pointing at the old one; if you already did, run the `ln -sf …` line it printed.
- Windows: **do not run it inside xj's install folder** (`%LOCALAPPDATA%\Programs\xj`): run there, the new version is unpacked but never swapped in, and nothing tells you.
  If you saved your copy of the public key (`~/.xing-janus/minisign.pub`) with PowerShell's `>`, the old version says it "is not a minisign public key" and does not update:
  convert your own copy (do not copy it again from the releases page) with `$f = "$env:USERPROFILE\.xing-janus\minisign.pub"; (Get-Content $f) | Set-Content $f`.

If you are still on 0.1.0, update by hand this once, following the last paragraph of "Update" in `docs/en/install.md`.
Your config, keys and transcripts stay as they are.

**Verify after downloading** (keep the two files next to the archive):

```bash
shasum -a 256 -c xj-0.1.3-darwin.tar.gz.sha256
minisign -Vm xj-0.1.3-darwin.tar.gz -P <your saved copy of the public key>
```

The Linux files are `xj-0.1.3-linux-x64.tar.gz` or `xj-0.1.3-linux-arm64.tar.gz` (check the SHA-256 with `sha256sum -c`); Windows is `xj-0.1.3-windows.zip`
(verify it as in steps 1–2 of "Installing on Windows" in `docs/en/install.md`, with your saved copy of the public key). Then check that the `Trusted comment:` minisign prints
reads `xing-janus v0.1.3 xj-0.1.3-<platform>…`; if not, do not install it (an old version renamed together with its signature passes both lines above).
If this is your first install, follow `docs/en/install.md`, then run `xj setup`.

Fixed:

- `xj update` (macOS, Linux): run in the home folder (or in `/`), it now switches the `xj` link on PATH from the old version to the new one (it used to say "No xj link on PATH points to this version" and leave `xj` on the old version;
  run in `/`, even the unpacking failed). The update is done by the old version's code, so this takes effect when updating from this version onwards
- Security: `xj update` no longer looks up programs on your PATH while unpacking (tar on Linux looks up gzip on PATH: if your PATH has a place the sandbox can write to,
  such as a project with an activated venv, a fake gzip planted there would run outside the sandbox and could replace what gets installed)
- Security: `xj setup` only suggests a default model whose name the config accepts (it used to print the first name the model service returned as is, so a malicious service could change your clipboard or clear the screen);
  the config is checked before it is written (it used to be written first, and if it was broken every start of xj said so)
- Security: PuTTY private keys (`*.ppk`) and SSH keys kept on a hardware key (`id_ed25519_sk`, `id_ecdsa_sk` and their `.pub`) are secret files too
- Security: before the sandbox starts, places the sandbox can write to are also removed from the PATH used outside the sandbox: the sandbox library looks up `which` on PATH, so if a folder you added to `sandbox.writable`
  is also on your PATH, a `which` that a sandboxed command put there would run outside the sandbox (the places on PATH inside `~/.pub-cache` and `~/.gradle`, writable by default, were already blocked from writing)
- Security: yes/no questions (trust this folder?, a pasted slash command, switching platforms) show control and invisible characters (bidi controls, zero-width) like the approval screens do:
  a bidi control in a folder name would reverse the rest of the line. Other one-line names on screen (models, file names, errors returned by services) show bidi controls too
- `xj update` (Windows): run inside xj's install folder, it stops first and asks you to leave (it used to say the next start runs the new version, but the background swap failed);
  if the background swap fails anyway (for example another window has a file in the folder open), the next start of xj and `xj doctor` say why and what to do
- `xj update`: a saved public key in UTF-16 is recognized too (as written by `'<public key>' > file` in Windows PowerShell 5.1; it used to say the file "is not a minisign public key" and not update)
- `xj update`: pressing Ctrl-C while it downloads or unpacks removes the download and the partly unpacked files, and nothing is swapped (they used to be left one level above the install folder, in `.xj-update-<hex>`, which you can simply delete)
- `/model`: models whose name is a file path (llama.cpp's `llama-server` without `--alias`) are remembered too, so the next start uses them again (the config format used to reject backslashes and spaces, so it was not saved);
  invisible characters (bidi controls, zero-width) are still rejected
- Windows: the start screen says "Commands in the sandbox can create folders in the root of these drives" once (it used to say it twice), and also when the sandbox is degraded (it used not to); the same for `xj -p` and `xj doctor`.
  `xj -p` also shows the sandbox notice like the start of a conversation does (on macOS, when `sandbox.allowLocalBinding` is on)
- When typing in xj (entering a key, adding a service), the explanation wraps at word boundaries: English words are no longer split, and wrapped rows do not start with a space
- User docs: the README is shorter and its security statements more precise; install.md says what to do when updating from 0.1.1 or 0.1.2, how to keep the public key on Windows (`%USERPROFILE%`),
  and that `xj reset` deletes your saved key too; Security adds "secret files are recognized by name" and "sandboxed commands can change secret files indirectly"; config.md covers `llama-server`'s `--alias`;
  troubleshooting covers both update problems

## 0.1.2

**Requires**: the same as 0.1.1 — macOS, Linux or Windows; Node 24 or later (on Windows, 24.20.0 or later is recommended); `ripgrep`. On Linux also bubblewrap, socat and sandbox-runtime 0.0.79;
on Windows, Git for Windows (see `docs/en/install.md`).

**Updating from 0.1.1**: run `xj update` (if you installed with Homebrew, use `brew upgrade xj`). If you are still on 0.1.0, update by hand this once, following the last paragraph of "Update" in `docs/en/install.md`.
Your config, keys and transcripts stay as they are.

**Verify after downloading** (keep the two files next to the archive):

```bash
shasum -a 256 -c xj-0.1.2-darwin.tar.gz.sha256
minisign -Vm xj-0.1.2-darwin.tar.gz -P <your saved copy of the public key>
```

The Linux files are `xj-0.1.2-linux-x64.tar.gz` or `xj-0.1.2-linux-arm64.tar.gz` (check the SHA-256 with `sha256sum -c`); Windows is `xj-0.1.2-windows.zip`
(for verifying in PowerShell, see the 0.1.0 section below). If this is your first install, follow `docs/en/install.md`, then run `xj setup`.

Fixed:

- `xj reset`: works after you have saved the public key as `docs/en/install.md` suggests (`~/.xing-janus/minisign.pub`); it used to stop, saying the folder did not look like Xing Janus's.
  It also recognizes Windows's `acl-fix.json` and temporary files left behind by an interrupted write
- Security: `id_dsa.pub` is a secret file too (the `.pub` of the other three SSH key types already was)
- Fetching web pages (`web_fetch`): redirects within the same site are followed up to 5 times, as documented (it used to stop at the 5th)
- User docs: the quick start now covers installing, switching platforms and updating; the README says how to update; troubleshooting covers the right column going out of line.
  Configuration, usage, install, privacy and the security notes were checked against the code, fixing what was wrong or missing, for example: `anthropic.fallback` switches models
  only when the safety classifier blocks a request; hitting the `limits` output cap only warns instead of stopping to ask; adding `"sudo"` to `dangerRemove` also stops asking for `su`, `doas` and the rest;
  the Homebrew removal order (run `xj reset` first); files and Credential Manager keys the Windows removal steps missed; the `tools` (`"reliable"`/`"unreliable"`) and `parallelToolCalls` settings currently have no effect

## 0.1.1

**Requires**: the same as 0.1.0 — macOS, Linux or Windows; Node 24 or later; `ripgrep`. On Linux also bubblewrap, socat and sandbox-runtime 0.0.79;
on Windows, Git for Windows (see `docs/en/install.md`).

**Updating from 0.1.0**: 0.1.0 has no `xj update` yet, so update by hand this once, following the last paragraph of "Update" in `docs/en/install.md`;
from then on `xj update` does it (if you installed with Homebrew, use `brew upgrade xj`). Your config, keys and transcripts stay as they are.

**Verify after downloading** (keep the two files next to the archive):

```bash
shasum -a 256 -c xj-0.1.1-darwin.tar.gz.sha256
minisign -Vm xj-0.1.1-darwin.tar.gz -P <your saved copy of the public key>
```

The Linux files are `xj-0.1.1-linux-x64.tar.gz` or `xj-0.1.1-linux-arm64.tar.gz` (check the SHA-256 with `sha256sum -c`); Windows is `xj-0.1.1-windows.zip`
(for verifying in PowerShell, see the 0.1.0 section below). If this is your first install, follow `docs/en/install.md`, then run `xj setup`.

New:

- macOS: install with Homebrew (`docs/en/install.md`); the tap verifies the signature with the public key before it updates the formula
- `xj update`: connects to GitHub only when you run it, installs only a newer version, and switches over only after the SHA-256 and the signature check out;
  `xj update --check` only tells you whether there is one
- Several API keys for the same provider: give each a name, pick by folder (`keys.<provider>.byFolder`), switch with `--key` or `/key`;
  if the chosen key cannot be found, xj does not start; spending is recorded per key, and each key can have its own daily and monthly limits
- On start, xj measures how this terminal draws emoji sequences and ambiguous-width characters (macOS Terminal.app and the old Windows console differ from other terminals;
  the "treat ambiguous-width characters as wide" setting is measured too) and lays out the screen accordingly: a line with emoji in the approval screen
  no longer overflows and pushes the right column away. Character widths now follow the Unicode tables
- Switching platforms inside xj: `/model` picks a platform first (showing which are on this computer, where each connects and where its key is kept),
  then a model; servers on this computer (Ollama, LM Studio, llama.cpp, vLLM) are found for you, and you can add an OpenAI-compatible service.
  A missing key is pasted right there (only • is shown; it is saved only if the model list answers, in the same place as `xj setup`).
  xj checks that the other platform answers before switching, and keeps the current one if it does not; it asks before the conversation goes to
  another machine. The choice is saved as `defaultModel` in the config (`/model` used to switch for the current session only)

Fixed:

- Security: search blocks hard links to secret files the same way read does; dangerous commands are recognized regardless of case and with `.exe`
  (`BASH -c`, `NOHUP`); quoting in PowerShell run with administrator rights is escaped more thoroughly; `SKILL.md` cannot be a link to somewhere else;
  network paths on PATH are not read before you are asked
- Spending: the summary made before switching to another provider checks the budget first; a subagent using another provider's model follows that key's limits
- Windows: another file (README.md, for example) was sometimes taken for a hard link to `.env`, blocking the whole command; the first logon of the sandbox account
  gets more time; the read-attributes-only permission on parent folders is now removed when xj exits (before, only the next time the sandbox started);
  the clipboard no longer gets an invisible U+FEFF; PowerShell 7 installed in your user folder cannot run in the sandbox, so while the sandbox is on
  xj uses PowerShell 7 under Program Files or the built-in 5.1 and says so at start and in `xj doctor`;
  opened in Git Bash's own window (mintty), xj explains that it is not a Windows console and to use Windows Terminal instead (before, it only said it needed a terminal),
  and Ctrl-C works as in a terminal there (interrupts the turn, press twice to leave; before, one press ended xj); `xj` starts in Cygwin
  (before, the launcher handed the `/cygdrive/…` path to node unchanged); starting while the computer is busy (a build running in the background, for example)
  no longer garbles non-English output of programs such as cmd and where for the whole session (before, one timeout while asking for the system code page
  meant reading everything as UTF-8 until xj was restarted), and the skip-permissions mode is no longer refused because the check for administrator
  rights timed out
- Screen: the approval screen wraps by visible character (flags and emoji are not split); after switching to full screen or resizing,
  the question is rewrapped to the new width; copying wrapped rows by dragging no longer includes the indentation
- Other: what xj has to tell you about the sandbox is also said at start (before, only in `xj doctor`); `xj -p` also prints config warnings;
  the date given to the model uses this computer's time zone; the key row of `xj doctor` says why a key was not found;
  Ctrl-C while xj is still starting up (for example while the sandbox starts) cleans up and exits (before, it did nothing until startup finished)
- User docs: every setting lists its default

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

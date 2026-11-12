# Changelog

Versions follow semver.

## 0.1.5

**Requires**: the same as 0.1.4 — macOS, Linux or Windows; Node 24 or later (on Windows, 24.20.0 or later is recommended); `ripgrep`. On Linux also bubblewrap, socat and sandbox-runtime 0.0.79;
on Windows, Git for Windows (see `docs/en/install.md`).

**Updating from 0.1.4**: run `xj update` (if you installed with Homebrew, use `brew upgrade xj`). Your config, keys and transcripts stay as they are. A few things work differently:
- `--dangerously-skip-permissions` only switches to skip-all mode; it no longer turns the sandbox off as well. Where the sandbox cannot start (in a container, for example), also add `--no-sandbox`, or put `"sandbox": { "enabled": false }` in that machine's config.
- With the sandbox off, the read, edit and search tools can also reach places outside the working folder (they used to be blocked): reads ask first, edits show the diff and ask. Secret files, credential folders and xj's own folders are still always blocked ("Scope follows the sandbox" below).
- The whole status bar turns red only in skip-all mode without a sandbox (it used to turn red with no sandbox, or in skip-all mode).
- If you wrote your own `statusBar.fields` in the config, add the new fields yourself: folders (`folders`), background commands (`jobs`) and this turn's cost (`turn-cost`).
- Each turn sends the model a few more tools (`skill`, `settings_propose`, and `job_output`, `job_stop` for background commands), so the descriptions are longer and it costs a little more (about 4–7% more on the same eval tasks).

**Verify after downloading** (keep the two files next to the archive):

```bash
shasum -a 256 -c xj-0.1.5-darwin.tar.gz.sha256
minisign -Vm xj-0.1.5-darwin.tar.gz -P <your saved copy of the public key>
```

The Linux files are `xj-0.1.5-linux-x64.tar.gz` or `xj-0.1.5-linux-arm64.tar.gz` (check the SHA-256 with `sha256sum -c`); Windows is `xj-0.1.5-windows.zip`
(verify it as in steps 1–2 of "Installing on Windows" in `docs/en/install.md`, with your saved copy of the public key). Then check that the `Trusted comment:` minisign prints
reads `xing-janus v0.1.5 xj-0.1.5-<platform>…`; if not, do not install it (an old version renamed together with its signature passes both lines above).
If this is your first install, follow `docs/en/install.md`, then run `xj setup`.

New:

- **Sandbox and mode are separate settings**: whether the sandbox is on follows `sandbox.enabled` in the config (on by default) and `--sandbox`, `--no-sandbox` on the command line; the mode only decides what xj asks you first.
  Full-auto works without a sandbox (edits without asking, every command asks), and `--auto --no-sandbox` can be used together.
  Skip-all with the sandbox on: commands still run inside the sandbox, domains you have not allowed (for commands and `web_fetch`) are blocked without asking, `excludedCommands` run outside the sandbox directly, tools that cannot run in the sandbox are blocked without asking (with a note on what to do), and MCP servers stay in the sandbox.
  The README has a new "Sandbox and modes" section, the mode table in the usage guide became four tables for the sandbox on and off, and a new "What to set" guide ([setup-guide.md](setup-guide.md)) gives the config lines for common situations
- `allowFullAuto` and `allowBypass` in the config: Shift-Tab and `/mode` can then switch to full-auto and skip-all inside xj (`--auto`, `--dangerously-skip-permissions` and an existing `defaultMode: "full-auto"` work as before);
  `defaultMode` accepts `"bypass"` (needs `allowBypass`). The first switch to skip-all after each start of xj asks you to type a confirmation; Shift-Tab says once when it skips a mode you have not allowed, and `/mode` lists the modes you cannot switch to and why
- In skip-all mode, deleting important places (the root, folders directly under it, your home folder, the working folder and the folders above it, added folders) and commands you added to `permissions.danger` still ask
  (`permissions.bypassAsksCriticalDelete` in the config, `true` by default)
- **Turn the sandbox off and on inside xj**: `/sandbox off` (needs `y`, Enter does not count; the question says what turning it off means), `/sandbox on` (switches only if the sandbox starts, otherwise it says why and keeps things as they were), `/sandbox` shows the state; clicking "Sandbox" in the right panel in full-screen mode does the same.
  It applies to this conversation only and does not switch while the model is working; the switch is noted in the conversation and the model is told; folders added with `/add-dir` and domains you allowed carry over to the new sandbox.
  Windows: if the PowerShell used at start (without a sandbox) cannot run under the sandbox account, xj switches to one that can when the sandbox is turned on
- The status bar and the right panel say where the sandbox was turned off ("off (config)", "off (flag)", "off (/sandbox)"). In the right panel, mode and sandbox get a grey line underneath (what the mode actually does with the current sandbox, and what it can reach);
  the order is now Mode, Sandbox, Trusted, Edits ok, Always ok, Checks, Enter=yes, Beep; the common commands include `/sandbox` and `/add-dir` (`/tui` was removed there and is still in `/help`); in short windows the grey lines go first, so the to-do list stays visible
- **Added folders can be writable**: `/add-dir <path>` asks "Should … be editable too?" (needs `y`). A writable folder follows the same rules as the working folder (edits ask according to the mode, `/undo` restores them, commands in the sandbox can write there, secret and protected files stay blocked);
  it must have been trusted at the computer, and it cannot be your home folder, a folder above it, the root, or a credential folder, nor contain programs xj runs outside the sandbox, a folder on PATH, or xj's own program; xj tells you when it has its own `AGENTS.md` or `CLAUDE.md`. It lasts for this conversation.
  `/add-dir` without arguments lists the folders added in this conversation and `/add-dir remove <number>` removes one; `/resume` adds the folders of that conversation again under the current rules
- Adding folders when xj starts: `--add-dir <path>` on the command line (writable, can be given several times) and `permissions.addDirs` in the config (listed per folder you start xj in, each marked read-only or writable); these stay when you switch conversations.
  `xj -p` adds them too but does not ask about trust (folders not trusted yet are read-only); `xj mcp` does not accept them. The status bar has a new "folders N" field and the right panel a new "Folders" item (click to list; "any" when the sandbox is off)
- After-edit checks run where the edits were: if only an added folder changed, that folder's own checks run (asking the first time), not the working folder's
- **Scope follows the sandbox**: with the sandbox on, the read, edit and search tools have the same scope as the sandbox (as before); with the sandbox off there is no scope: reads outside the working folder ask first, edits show the diff and ask, marked in red
  (also in auto-edit and full-auto; skip-all does not ask), and `/undo` restores them. Secret files, credential folders (`~/.ssh`, `~/.aws`, `~/.gitconfig`…) and xj's own folders are still always blocked;
  edits are also blocked for protected files (`.git`, `.vscode`… at any level) and places that run things automatically later (shell startup files, things that run at boot and login, schedules, `/etc` and so on;
  on Windows the PowerShell profile `$PROFILE`, at its real location when Documents is moved to OneDrive or redirected by your organisation)
- **`/permissions` is easier to read**: it first states the current setup and how to change it, then lists, in groups, what you allowed permanently (numbered, removable), what the config changed, and the built-in rules (the whole allow list grouped by purpose, the danger list, secret-file patterns);
  in full-screen mode a menu opens after the list (↑↓ to choose, Enter for details, `y` to remove), and clicking "Always ok" in the right panel opens the menu directly
- **Budgets you can see and change**: `/budget` lists the three limits, what you have spent and how to change them; change one like `/budget day 50` (Enter changes this conversation only; press `y` as well to write it to the config).
  The status bar shows today's limit, plus this turn's cost while the model works; the right panel has a new "Today" item for today's budget (yellow at 80%, red at the limit, click for `/budget`); the 80% and limit messages say how to raise it; `/cost` lists the limits too
- **Commands can run longer, or in the background**: `commands` in the config (the default `timeoutSec` and the longest `maxTimeoutSec` for foreground commands, 2 and 10 minutes as before; the longest `backgroundMaxMinutes` for background commands, 60 minutes by default, 0 for no limit);
  the time-out message says which setting applied and its value. The model can run things like a dev server in the background, read its output, wait for it to finish and stop it; it goes through the same questions (the question says that it runs in the background, and for how long at most) and runs in the sandbox, at most 8 at a time; each can print at most 5 GB (past that it is stopped, xj says why, and its output file keeps only the start and the end).
  `/jobs` lists them, shows output and stops them; the status bar shows "jobs N" and the right panel in full-screen mode lists the running ones; when one finishes, a line is added to the conversation (with a sound); switching the sandbox or `/exit` asks first while some are still running. Not available in `xj -p`, `xj mcp` or subagents.
  When the Windows sandbox has to re-apply its file permissions (a secret file was replaced by another file, a folder was removed, an editable folder was added, a new secret file appeared directly in an editable folder itself) while background jobs are still running, the next command does not run and xj says which ones to stop first
  (while the permissions are re-applied, a command still running could read secret files and change protected ones)
- **`/config`**: the common settings with their current values; pick one to change it, with a note on what the change does (sandbox at start, default mode, allow full-auto, budgets, command time limits, interface language, sound, Enter = yes, screen, folders to add;
  allowing skip-all can only be viewed). Loosening needs `y`; only that field is changed and the whole file is checked before writing. Budgets, interface language, sound, Enter = yes and screen also change right away in this conversation
- **Ask xj how to configure it**: just ask ("this project needs docker, what should I set?"); it answers from your current settings (the built-in `xj-settings` skill, which comes with a snapshot of your current settings, keys and the like hidden).
  It can also propose changes: each one is marked as loosening, tightening or not affecting permissions, and xj asks you first (loosening in red, needs `y`); only after you agree does xj write the config. Mode, allowing full-auto, budgets, the allow list and domains also apply right away in this conversation.
  It cannot propose skip-all mode, removing built-in protections, MCP servers, `xj mcp` settings, keys and services, or environment variables; untrusted folders do not take loosening proposals; not available in `xj -p`, `xj mcp` or subagents
- `xj doctor` has a new "Relaxed in the config" row: sandbox turned off, `allowFullAuto`, `allowBypass`, a loose `defaultMode`, `bypassAsksCriticalDelete: false`, `"*"` in `allowedDomains`,
  removed built-in danger rules and secret files, `gitconfig: "full"`, `allowLocalBinding`, `writablePrograms: "allow"`
- `xj setup`: when saving a key on macOS it explains that "password data for new item:" is macOS's own prompt from `security` (the key still goes straight to `security`, not through xj);
  step 3 lists models one per line as "provider:model", at most 15, and says how many more there are and that `/model` inside xj shows them all; at the end it mentions the default budgets and `/budget`
- User docs: the danger list in `config.md` now matches the code (every recognised command, plus the PowerShell list), and how to write `sandbox.allowedDomains` is spelled out (ports, `*.github.com`, `"*"` for any domain; localhost, private networks and metadata stay blocked);
  the Homebrew section of `install.md` says brew also installs node's dependencies and upgrades older packages

Fixed:

- **In the Windows sandbox, a command could rename the whole `.git` (or another folder that holds protected files) and put a new one in its place**, getting around the block on `.git/config` and `.git/hooks`
  (git you run yourself later would then run programs from the new config; git run by xj itself is not affected). While the sandbox is on, the folders that hold protected files now get "the sandbox cannot delete this",
  once for the sandbox account and once for the sandbox group, before any folder is opened (whenever srt-win changes permissions, also for another xj, it removes the account's entry and leaves the group's alone);
  if that cannot be added, or the sandbox group is missing, the whitelist does not apply and every command asks. macOS and Linux did not have this problem
- **In skip-all mode, a command that names a secret file inside `bash -c '…'`, `sh -c '…'` or `eval '…'` still ran** (`bash -c 'cat .env'`) and its contents went to the model:
  the danger list and "deleting something important" both look inside such strings, the secret-file check did not (up to 0.1.4 skip-all always meant no sandbox; from this version the operating system still blocks it while the sandbox is on).
  They now share one way of taking such commands apart. A file name that is written out in the command is now blocked whatever the quoting (`cat $'.env'`), in braces (`cat {.env,README.md}`, `{cat,.env}`),
  in a redirect at any level (`echo "$(< .env)"`), as `dd if=.env`, or inside `bash -c` / `sh -c` / `eval` (nested, `find -exec sh -c '…'`, behind a wrapper command xj does not know).
  A name that has to be computed (variables, wildcards, code for another program) is still not caught; that takes the sandbox (see cerberus.md)
- **Some ways of writing a command were invisible to the danger list, to "deleting something important" and to the secret-file check**:
  everything between a double-quoted string ending in `$` and the next double quote (`grep "x$" f; rm -rf ~; echo "ok"`: the part in between was taken as part of a string);
  options or `--` after `-c` (`bash -c -- 'rm -rf ~'`, `bash -ceo pipefail '…'`) and `bash +c '…'`; a `case` inside `$( )`. All fixed.
  A command nested too deeply, or with braces expanding to too many words, to be checked completely used to pass as harmless; it now asks (in skip-all and full-auto too).
  A long command nested many levels deep used to take a long time to check (over ten seconds for a few dozen KB, with xj unresponsive meanwhile); now under a second
- **Windows: with a `\r` (CR) in a command, the danger list, "deleting something important" and the secret-file check saw something other than what Git Bash runs**:
  Git Bash drops every `\r` in a command before running it (also inside quotes, `eval` and `bash -c`), while xj treated it as a space (`cat .e\rnv` really reads `.env`;
  in `true\r#x; cat .env` the second half was taken as a comment). xj now drops it first, like Git Bash; a command with a `\r` is still not whitelisted
- **Windows: in skip-all mode without a sandbox, a command that reads a secret file through `cmd` or PowerShell from Git Bash still ran** (`cmd //c "type .env"`, `powershell -c "Get-Content .env"`) and its contents went to the model:
  bash takes the quoted string as one word. That string is now checked with the PowerShell tool's rules: secret and protected files are refused, deleting something important asks, and cmd's `^` escapes are dropped first.
  The PowerShell tool (and this check) also sees quotes in the middle of a word: PowerShell joins `type .e'n'v` into `.env`, which used to be split at the quotes and missed
- **A multi-line private key inside an ordinary file was only masked on its first line**: `read` (which adds line numbers) and `search` (which returns results line by line) handed the body of the key to the model as it was. A key in a secret file (`*.pem`, `id_rsa`…) was always blocked completely;
  this was about keys embedded in config files, documents or test data. They are now masked completely: `read` replaces each line and keeps the line numbers (also when you start reading in the middle of the key); in `search` results a single line that looks like the body of a key is replaced too.
  Command output also gained a few forms that were not masked before: the old passphrase-protected format (the body follows a blank line after `Proc-Type` and `DEK-Info`), PGP private keys, keys written on one line (line breaks written as `\n`), keys whose body is not wrapped,
  keys with something in front of every line (the `> ` of a diff, the path and line number of `grep -n`, a log prefix), and output that shows only the last part of a key and its END line (`tail`, the context lines of a diff).
  Output with a BEGIN line followed by more than a hundred thousand lines of base64 used to fail completely (the tool returned an error); it now comes back
- `[!…]` ("none of these characters") in your own secret-file patterns was taken literally: `key[!s].json` blocked `keys.json` and let `keya.json` through. `[!…]` and `[^…]` now both mean "not" (the built-in list does not use them)
- When pasted text looked like a slash command, the confirmation showed only its first line, while answering yes handed the whole text to the command (`/checks add` stored all of it as one check): the whole text is now shown, with control codes and invisible characters marked.
  The danger line in a question and the skill folder names on the opening line also mark direction controls and line breaks now
- Windows: when looking up a command that runs outside the sandbox (`sandbox.excludedCommands`), a Microsoft Store app execution alias now gets the same check as the PowerShell lookup (`..` resolved first, the target must really be a file); it used to compare only the start of the target path
- After switching conversations (`/clear`, `/resume`), skill folders could not be read in the sandbox (macOS, Linux); on Windows with skills, the first command after the sandbox started re-initialised it completely (several minutes with large toolchains) → they are given when the sandbox starts
- Windows: `/add-dir` no longer prepares the whole sandbox again (minutes for big toolchains): only the new folder is added (its secret and protected files are blocked first, then it is opened),
  and the same goes for folders added when xj starts; only removing a folder starts over. Adding a folder no longer restarts running MCP servers either
- What the model is told about the mode now follows the current sandbox: skip-all used to say "the sandbox is off too" even with the sandbox on, and with skip-all and no sandbox the opening description contradicted itself (it also said every command asks).
  On Windows the PowerShell tool description no longer depends on whether the sandbox is on (tool descriptions stay the same for the whole conversation)
- When switching conversations, a folder that is in `extraReadable` in the config and was also added as writable with `/add-dir` lost the config's read access as well
- With the monthly limit set to 0 (no limit), the status bar and `/cost` showed "/$0.00" (the daily and per-turn ones already showed nothing)
- How long something ran is written the same way everywhere ("1 min 33 s", "1 h 2 min" past an hour): the status bar used to show only seconds for a running command and for the wait for the model ("125s"), and the turn line said "1m 20s"
- **With an Anthropic server-side fallback, the context was counted twice**: one response then has two usage entries, each including the whole conversation, and they used to be added up, so automatic compaction came early (one more request, details lost) and subagents wrapped up early.
  The last entry is used now (the cost still counts every entry). When `/resume` continued a conversation whose last usage entry was a subagent's, that used to be taken as the main conversation's context
- After `/effort`, switching to another model of the same provider still sent the level switched to before, even to a model without reasoning levels. The level is now chosen for the new model (the one the status bar shows), and nothing is sent where it is not supported
- `/sandbox` did not count background jobs that had just been stopped but whose processes had not ended yet: switching right after stopping them, their cleanup went to the sandbox that had been replaced. It now waits for them to end first (a few seconds at most)
- Text cut in the middle of a character (emoji, CJK): the last part of a job that `job_output` gives the model, the config shown when a skill is read, reasoning progress and srt-win error messages could keep half an emoji (some services reject the whole request when the model gets one);
  when long output is cut to its start and end, a cut inside a CJK character left a "�". A background job printing without line breaks used to be cut every 64 KB at any byte: the character there became "�", and on Windows that whole piece was decoded with the console code page and garbled
- `search` stops at its time limit (60 seconds) and at 20 MB of results; what the model got used to look like all the results (and with nothing found yet it could even say "nothing matches"): it now says the results are partial and how to narrow the search

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

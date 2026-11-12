# Usage

## Sandbox and modes

Two separate settings: the **sandbox** decides what can be reached (whether commands are confined, and the scope the tools can read and edit), and the **mode** decides whether xj asks you first.
By default the sandbox is on and the mode is normal: edits show a diff and ask, whitelisted commands run directly, and the rest ask. Most people do not need to change anything; for what to set in your situation, see [setup-guide.md](setup-guide.md).

| | Sandbox | Mode |
|---|---|---|
| Decides | What can be reached: whether commands are confined, the scope the tools can read and edit | Whether xj asks you first |
| Options | On, off | Read-only, plan, normal, auto-edit, full-auto, skip all permissions |
| Default | On | Normal |
| When xj starts | `sandbox.enabled` in the config; `--no-sandbox`, `--sandbox` on the command line | `defaultMode` in the config; `--auto`, `--dangerously-skip-permissions` on the command line |
| Switching in xj | `/sandbox off`, `/sandbox on`, a click in the right-hand column | Shift-Tab, `/mode`, a click in the right-hand column |
| Confirmation | Off: press `y` (Enter does not count; the question says why) | Skip all permissions: type "I understand the risks" the first time in each run of xj |
| Must be enabled in the config first | No | Switching to full-auto (`allowFullAuto`) or skip all permissions (`allowBypass`) inside xj. Starting with `--auto`, `--dangerously-skip-permissions`, or with `defaultMode: "full-auto"` written, counts for that run |
| Status bar | "sandbox on", "sandbox off (config)" | "Normal mode", "Skip-all-permissions mode"… |
| Cannot be changed by | The model (it can only propose a config change, written if you approve; see "Changing settings" below), files in the project folder | Same |

When both are at their loosest (skip all permissions, no sandbox), the whole status bar turns red.

**Sandbox on** (scope = the working folder and added folders):

| Mode | Edits | Whitelisted commands | Other commands | Dangerous commands | New domains | Commands outside the sandbox |
|---|---|---|---|---|---|---|
| Read-only, plan | Not allowed | Read-only ones run directly | Not allowed | Not allowed | Ask | Not allowed |
| Normal | Shows the diff, then asks | Run directly | Ask | Ask | Ask | Ask |
| Auto-edit | No asking | Run directly | Ask | Ask | Ask | Ask |
| Full-auto | No asking | Run directly | No asking | Ask | Ask | Ask |
| Skip all permissions | No asking | Run directly | No asking | No asking† | Blocked | No asking (built-in ones blocked‡) |

- Files outside the scope cannot be read or edited in any mode (to widen the scope: `/add-dir`, see "Added folders" below); places in your home folder outside the scope are out of reach for both the tools and commands (commands can read the toolchain folders listed in `sandbox.toolchainReadable`).
- Every command runs in the sandbox (the ones that do not ask, too): it can write only inside the scope, plus the temp and cache folders.
  MCP servers run in the sandbox as well (except those with `sandbox: false` in the config; the Windows sandbox cannot run MCP servers, and xj says what to do at startup).
- New domains: anything other than `sandbox.allowedDomains` and those allowed this conversation. Commands outside the sandbox: those listed in `excludedCommands` in the config (docker and the like), and the built-in flutter, dart, fvm and installers (winget, scoop, brew).
- Plan is like read-only, and at the end it proposes an approach; after you approve it switches to normal mode and starts.
- †Deleting an important place, your own danger rules and switching to root still ask (see "The same whatever the sandbox and mode" below).
- ‡Those listed in `excludedCommands` run outside the sandbox directly; the built-in flutter, dart, fvm and installers are blocked, and the screen says what to do: switch to a mode that asks, list it in `excludedCommands`, or turn the sandbox off.

**Sandbox off** (no scope):

| Mode | Edits inside the scope | Files outside the scope | Commands (the whitelist does not count) | Dangerous commands |
|---|---|---|---|---|
| Read-only, plan | Not allowed | Read: ask; edit: not allowed | Read-only ones ask, others not allowed | Not allowed |
| Normal | Shows the diff, then asks | Ask | Ask | Ask |
| Auto-edit | No asking | Ask | Ask | Ask |
| Full-auto | No asking | Ask | Ask | Ask |
| Skip all permissions | No asking | No asking | No asking | No asking† |

- Commands run with your permissions and nothing watches the network: so every command asks, except in skip-all-permissions mode.
- Files outside the scope: reading and editing both ask (edits show the diff in red, without "allow all edits"; `/undo` can restore them). What stays blocked is in the next table.
- `web_fetch` still asks about new domains (except in skip-all-permissions mode). Without a sandbox, full-auto is the same as auto-edit.
- MCP servers without `sandbox: false` in the config do not start (except in skip-all-permissions mode: they start outside the sandbox, with your permissions).
- †As with the sandbox on: deleting an important place, your own danger rules and switching to root still ask.

**The same whatever the sandbox and mode**:

| | |
|---|---|
| Always blocked | Secret files (`.env`, key files…); protected files (`.git/hooks`, `.vscode/tasks.json`…); places that run automatically (shell startup files, things that run at boot and login, `~/.ssh`, `~/.xing-janus`…). With the sandbox off, commands are only stopped when they name these directly |
| Still asks in skip-all-permissions mode | Switching to root (`sudo`…); files it cannot tell are secret (files with several names); deleting the root, a top-level folder, your home folder, the working folder or a folder above it, or an added folder (can be turned off in the config); your own entries in `permissions.danger`; commands xj cannot check all the way through (nested too deeply, or braces that expand to too many words); settings the model proposes (`settings_propose`) |
| MCP tools | Ask (except those in `autoApprove`); read-only and plan allow only those marked read-only in the config; skip all permissions does not ask |
| Budget limits | Still apply: at a limit xj stops and asks (`/budget`) |
| Untrusted folders | Only read-only and plan |

- Dangerous commands (`rm -rf`, `git push`, `sudo`…) are marked in red with the reason when xj asks. The built-in ones and how to add or remove entries are all listed in [config.md](config.md) under "The built-in whitelist, danger list and secret files".
- Switch with `/mode` or Shift-Tab. In a folder you do not trust, only read-only and plan modes are available. Shift-Tab skips full-auto and skip-all-permissions when the config does not enable them
  (the first time it skips, it says how to enable them), and `/mode` without an argument lists the modes not available this time and why. Each switch adds a line to the conversation, such as "Mode: normal → auto-edit" (yellow when it loosens permissions);
  a Shift-Tab pressed in normal mode while you are typing is recorded when you send that line (otherwise it would mess up the input line).

### The sandbox

Commands run inside the operating system's sandbox: they can only write in the working folder and editable added folders (plus temporary and cache folders), cannot read secret files or the rest of your home folder, and only reach allowed network domains.
The sandbox and the mode are two separate settings: the sandbox decides what commands can reach, the mode decides whether you are asked first.

- **When xj starts**: `sandbox.enabled` in the config (on by default); `--sandbox` or `--no-sandbox` overrides it for one start.
- **Inside xj**: `/sandbox off` turns it off, `/sandbox on` turns it on, `/sandbox` shows the current state (if it failed to start or is not installed, why and what to do); in full-screen mode, clicking "Sandbox" in the right-hand column does the same. It only affects this conversation; the config file is not changed.
  - **Turning it off needs `y`** (Enter does not count, even with Enter = yes on; the question says why). After that, commands run with your permissions and can reach the whole machine, and the whitelist no longer applies:
    every command asks (except in skip-all-permissions mode, where nothing asks after this; the question says so in red). MCP servers set to run in the sandbox are stopped and do not start again (except in skip-all-permissions mode, where they start outside the sandbox).
  - **The tools' scope follows the sandbox**: with the sandbox on (or failed to start), the read, edit and search tools have the same scope as commands in the sandbox (the working folder and added folders), and anything outside is blocked.
    With the sandbox off there is no scope: reading outside the working folder asks first (in read-only and plan mode too), and editing there shows the diff and asks, marked red (auto-edit, full-auto and "allow all edits" ask too);
    skip-all-permissions mode does not ask. Edits you approved there can be restored with `/undo`. Always blocked: secret files, places for credentials (`~/.ssh`, `~/.aws`, `~/.gitconfig`…) and xj's own folder;
    for edits also protected files (`.git`, `.vscode`… at any level) and places that run things automatically later (shell startup files, things that start at boot or login, scheduled jobs, `/etc`, `~/.gradle/init.d` and the like; on Windows the PowerShell profile, at its real location when Documents is moved to OneDrive).
  - **Turning it on**: it switches only if the sandbox starts; if it cannot start, there is still no sandbox and xj says why (as at startup; on Windows without the sandbox installed, it says how to install it). Folders added with `/add-dir` and domains allowed this conversation still count;
    MCP servers then start inside the sandbox. On Windows, starting the sandbox takes a while (as at startup: minutes for big toolchains, with progress);
    if the PowerShell used while there was no sandbox cannot run in it (PowerShell 7 installed in your user folder), this conversation switches to one that can (xj says so).
  - Not while the model is working (a command halfway through uses the old sandbox): wait for the turn to finish. If background jobs are still running, xj asks you and stops them first (see "Command time limits and background jobs" below). The mode does not change.
  - After a switch, the conversation gets a line ("Sandbox: on → off (/sandbox)"), the status bar and the right-hand column say what turned it off (flag for `--no-sandbox`, config, `/sandbox`), and the model is told.
  - The model cannot switch the sandbox: only your `/sandbox`, a click in the right-hand column, the config file or the command line can (files in the project folder cannot).
    The model can only propose changing `sandbox.enabled` in the config: it is written only if you press `y`, and applies from the next start (see "Changing settings" below).

### Full-auto mode

Set `"allowFullAuto": true` in the config, and Shift-Tab and `/mode` can reach it; or start with `xj --auto` this time (`defaultMode` set to `"full-auto"` counts too). Edits and commands in the sandbox no longer ask first,
and the sandbox is the safety boundary. It only works in a **trusted folder**. The sandbox and the mode are two separate settings: without a sandbox (`--no-sandbox`, `/sandbox off`, or the Windows sandbox not installed), or when part of the sandbox cannot block
(a Windows project too large to finish scanning for secret files; the sandbox account can write to folders on PATH or other places that run outside the sandbox), it can still be entered, but only file edits skip asking: **every command asks** (xj says so at startup).

Still asks: commands on the danger list, new domains, commands that run outside the sandbox (`excludedCommands`, and tools such as flutter that cannot run in the sandbox), PowerShell commands outside the whitelist (PowerShell strings can build any path or command), MCP calls, and cases where it cannot tell whether a file is a secret
(for example a file with several names). Still always blocked: secret files, and files that later run outside the sandbox (`.git/hooks`, `.vscode/tasks.json`…). The budget limits still apply.

What the sandbox cannot block is up to you:

- **Things in the working folder**: the sandbox lets commands change the working folder, so deletes and overwrites not on the danger list do not ask, and `/undo` cannot restore files changed by commands. **git commit before you start.**
- **Things you later run outside the sandbox**: source code and `package.json` scripts are its work area. **Look at `git diff` before running build or `npm start` outside the sandbox.**
- **Allowed domains**: programs in the sandbox can send things to domains you already allowed without asking again.
- **Instructions hidden in files**: if it is led astray, anything not on the danger list happens directly. Only use it on code you trust.

Trusting the folder again (`/trust`) does not bring full-auto back by itself; switch again. `xj -p` only uses full-auto with an explicit `--auto`: when the config's default mode is full-auto or auto-edit, `-p` uses normal mode.

### Skip-all-permissions mode

Start with `xj --dangerously-skip-permissions`. Edits, commands (including those on the danger list), PowerShell and MCP calls never ask first.
The sandbox and the mode are two separate settings: this flag only changes the mode, and the sandbox follows its own setting (`sandbox.enabled` in the config; `--sandbox` or `--no-sandbox` overrides it for one start; inside xj, `/sandbox` switches it).

- **Sandbox on**: commands stay in the sandbox (so do MCP servers), cannot read secret files and cannot write outside the working folder. The network only reaches allowed domains (`sandbox.allowedDomains`, and those allowed this session);
  others are blocked without asking, `web_fetch` included; to allow one, add it to `sandbox.allowedDomains`. Commands listed in `excludedCommands` still run outside the sandbox, without asking;
  tools that cannot run in the sandbox (flutter, dart, fvm, installers) are blocked without asking, with what to do to use them (switch to a mode that asks, list it in `excludedCommands`, and only last turn the sandbox off).
  What the sandbox does not stop: deletions and changes in the working folder (/undo cannot restore what commands change) and pushes to allowed domains: **git commit before you start.**
- **No sandbox** (`--no-sandbox`, turned off in the config, or `/sandbox off`): the read and edit tools also reach outside the working folder without asking (what is always blocked stays blocked; see "The sandbox" above). Commands can do anything your account can: delete files, read the SSH keys and cloud credentials in your home folder, connect to any site, use Docker; MCP servers also run outside the sandbox.
  Instructions hidden in files and web pages (prompt injection) can tell it to do exactly that. **Only use it in an environment you can throw away (a container or VM), and git commit before you start.**
  In a container the sandbox often cannot start (then every command is refused): write `"sandbox": { "enabled": false }` in that machine's config, or start with `--no-sandbox`.

- **Still always blocked**: secret files named directly (`cat .env`, reading a key file…) and protected files (`.git/hooks`, `.vscode/tasks.json`…). When it cannot tell whether a file is a secret (a file with several names), it still asks.
- **Indirect reads are not blocked**: without a sandbox, xj only sees the text of the command. `grep -r`, `cat *`, `find -exec cat`, `tar`, PowerShell's `Get-ChildItem | Get-Content`,
  or a program the model writes can all read a `.env` in the folder, and what they read goes to the model service. **Keep secrets out of this folder.**
- **Switching to root still asks**: `sudo`, `su`, `doas`, `sudoedit`, `pkexec`, `run0` (also inside `env` or `bash -c`), and PowerShell's `-Verb RunAs`, `runas` and `sudo`.
  To stop `sudo` from asking, add `"sudo"` to `permissions.dangerRemove` in the config (it then is not marked red in normal mode either): the danger list's `sudo` entry covers all six bash commands above, so `su`, `doas` and the rest stop asking too.
  It only covers those six: PowerShell's `-Verb RunAs`, `runas` and `sudo` still ask.
- **Deleting an important place still asks**: deleting the root, a top-level folder (`/usr`, `/etc`…; on Windows `C:\` and folders such as `C:\Windows`, `C:\Users`), your home folder,
  the working folder or a folder above it, and added folders (`/add-dir`, `permissions.extraReadable`) or a folder above them. `rm`, `rmdir`, `find … -delete`, `shred` and PowerShell's `Remove-Item` all count,
  also inside `bash -c`, `env` or `xargs` and after a `cd`; a wildcard that deletes a whole level (`rm -rf *`, `rm -rf ~/*`) and a target it cannot tell (variables, `$( )`, `xargs rm`) ask too.
  It is marked red and needs `y`; with `xj -p` nobody can answer, so it is refused. Matching goes by the command's words: deleting through a program (`python -c "…"`) is not recognized.
  To stop being asked: set `permissions.bypassAsksCriticalDelete` to `false` in the config (only skip-all-permissions mode reads it: the other modes ask anyway).
- **What you added to `permissions.danger` still asks**: you said explicitly that it should ask. The built-in danger list still does not.
- **Settings the model proposes still ask** (`settings_propose`, see "Changing settings" below): the config decides what the model may do.
- **It cannot start as root**: `sudo xj --dangerously-skip-permissions`, or the default root user in a container, is refused; use a normal account instead.
  The same goes for a terminal opened with "Run as administrator" on Windows (when xj cannot tell, it assumes it is).
  If you are in a throwaway container and must use root, set the environment variable `XING_JANUS_ALLOW_ROOT_BYPASS=1` and start again.
- **Still in effect**: the budget limits and the per-turn tool-call limit.
- **How to enter it**: start with `--dangerously-skip-permissions` this time; or set `"allowBypass": true` in the config, and Shift-Tab and `/mode` can reach it;
  `defaultMode` can also be `"bypass"` (needs `allowBypass`; you confirm when xj starts, and cancelling starts in normal mode). `xj -p` only accepts the flag (the config does not count).
- **Confirm the first time you enter it after starting xj**: it shows a warning and the notice, and you must type "I understand the risks" yourself and press Enter (pasting does not count; Esc cancels); switching out and back in does not ask again until xj restarts.
- **Only in a trusted folder**: otherwise it is not entered, and xj starts in read-only mode (untrusted folders only allow read-only and plan).
- The status bar says "Skip-all-permissions mode"; without a sandbox the whole bar is red (both at their loosest). Shift-Tab can switch to other modes; the sandbox stays as it is (without a sandbox, every command asks after switching).
- **Windows: when the sandbox account can write to folders on your PATH**. If the sandbox start finds this, xj runs `icacls` as administrator to block it
  (it only blocks the sandbox account from writing; Windows shows a UAC prompt and it only runs if you agree; it runs in the background, so you do not wait):
  - With the sandbox on: xj **asks you once** before the input line appears (yes / no / always allow). "Always allow" sets `sandbox.autoFixWritable` to `true` in the config,
    after which it no longer asks (see and remove it with `/permissions`). "No" leaves it alone and does not ask about the same places again (the sandbox start still says the whitelist does not apply).
  - In skip-all-permissions mode: it applies what the last sandbox start recorded **without asking**.
  - If you decline UAC or it fails, it does not run automatically again and prints the commands for you instead. With `-p`, where nobody can answer, it only prints the commands.
  - The record is `~/.xing-janus/acl-fix.json` (the sandbox can neither read nor write it), and xj checks that it names the sandbox account's current SID before blocking.
- `xj -p --dangerously-skip-permissions`: nobody can type a confirmation, so the flag counts as consent; the warning and notice go to stderr. In an untrusted folder it exits (exit 2).
- It cannot be combined with `--auto`.

## Permission questions

- **Edits**: `y` applies, `n` declines, `a` allows all edits this session.
- **Commands**: `y` runs, `n` declines. When you decline, it tries another way instead of retrying the same thing.
- **Windows**: bash means Git Bash (the whitelist and danger list are the same as on macOS and Linux); there is also PowerShell, whose whitelist only accepts very simple read-only forms
  (`Get-ChildItem`, `Get-Content`, `Select-String`…, with plain words or single-quoted values as parameters), and read-only and plan modes can only run those; everything else asks.
  Without the Windows sandbox installed, every bash and PowerShell command asks.
- **Domains**: when a command in the sandbox or `web_fetch` (fetching a web page) wants to reach a domain that is not listed, it asks: `o` allow once, `s` allow this session, `p` always allow, `n` deny.
  Both use the same list: a domain allowed for `curl` does not ask again for fetching pages, and the other way around.
- **Enter means yes**: on by default (as in Claude Code; set `"enterToApprove": false` in the config to always press y). Enter answers yes to ordinary questions (the option shows `[y/Enter]`), and "allow once" for domain questions.
  Dangerous commands, content you have not seen in full (press `v` first), the confirmation for always-allow, and confirmations that loosen things (turning the sandbox off, making an added folder editable, loosening settings) still need `y`; an Enter within 0.5 seconds of a question appearing does not count.
- **Always allow (`p`)**: written into the config, **for every project**. After you press it, it asks once more, listing the file, the field and what will be written; nothing is written until you press `y` (`n` goes back to the original question).
  Where it is offered: domains in the sandbox (written to `sandbox.allowedDomains`), simple commands (written to the whitelist `permissions.allow` as the command plus its subcommand,
  for example `npm test`: `npm test --watch` then counts too), MCP tools (written to that server's `autoApprove`).
  Where it is not: commands on the danger list, commands that run outside the sandbox (`excludedCommands`), commands with `&&`, pipes or redirection, a bare shell or interpreter (`node`, `python`, `bash`),
  PowerShell commands outside the whitelist, and edits (for those, make auto-edit the default mode). Secret files can still only be changed in the config by hand. Always-allow for commands is only offered with the sandbox on and in a trusted folder (only then does the whitelist run them directly).
  See and remove always-allowed items with `/permissions`: it first gives the current state and how to change it (mode, sandbox, folders, trust, edits ok), then lists by group what you always allowed (numbered; `/permissions remove <number>` removes one),
  what the config changed (danger list, secret files, folders added at startup…; view only) and what is built in (the whole whitelist grouped by purpose, the danger list, secret files).
  In full-screen mode a menu follows the list: ↑↓ to pick, Enter for details, `y` to remove, Esc to finish; clicking "Always ok" in the right-hand column opens the menu directly (`/permissions menu`). Nothing can be added here.
  Other loosened settings (the sandbox turned off, `allowBypass` and so on) are listed in the "Relaxed in the config" row of `xj doctor`.
- A real question has `▍` at the start of its title and of the line that wants your answer, and the label on the left is "xj". Text from the model and file contents cannot print that mark: something that looks like a question without it was written by the model, not asked by Xing Janus.

## Added folders (/add-dir)

Open xj in A and you can read and edit a related folder B: `/add-dir ../api`. There is no need to open xj in the parent folder (the workspace with all your projects): that would put every project in it in scope.

- It asks "Should … be editable too?": no means read-only (read and search, no writing); to make it editable press `y` (Enter does not count).
- Read-only or editable, it cannot be your home folder, a parent of it, or the root, nor a folder that holds credentials (`~/.ssh`, `~/.aws`…) or xj's own folder (`~/.xing-janus`).
- **Editable folders** follow the same rules as the working folder: edits ask according to the mode (normal mode shows the diff first), `/undo` can restore them, and commands in the sandbox can write there too;
  secret files and protected files (that folder's own `.git/hooks`, `.vscode/tasks.json`…) stay blocked.
  - The folder must be trusted at the computer (its `package.json` scripts and hooks will run later, just as when you open xj there):
    if it is not trusted yet, xj asks the same question as at startup (saved in the same `trusted.json`); if you say no, it is read-only.
  - A folder that contains programs xj runs outside the sandbox (git, rg…), folders on its PATH or xj's own program cannot be editable (commands in the sandbox could replace them, and they would then run outside the sandbox with your permissions); read-only is fine.
  - If you add a parent of the working folder, xj warns that every project inside it is in scope.
  - If the folder has its own `AGENTS.md` or `CLAUDE.md`, xj says so, and the model reads it before changing anything there.
- `/add-dir` with no argument lists this conversation's folders (editable and read-only; those from `permissions.extraReadable` in the config are listed separately);
  `/add-dir remove <number>` removes one: the tools and the sandbox stop reaching it right away (MCP servers running in the sandbox restart), and `/undo` can still restore what was changed there.
- For this conversation only: `/clear` takes them back. When `/resume` continues an earlier conversation, the folders added then are added again under today's rules (editable ones must be trusted; xj asks if they are not).
- **Adding at startup**: `xj --add-dir ../api` (editable; repeat for more); for folders to add every time you open xj in this project, use `permissions.addDirs` in the config (see [config.md](config.md)).
  Folders added at startup stay for new conversations; only `/add-dir remove` takes them out. `xj -p` adds them too but cannot ask about trust, so untrusted ones are read-only. `xj mcp` does not take them (the folders it reaches are `mcpServer.projects`).
- **Checks after edits** run where the edits were: when only an added folder changed, that folder's own checks run inside it (the ones added with `/checks add` when xj was opened there, and its `.xing-janus/checks.json`:
  asked first the first time and whenever it changes), and the working folder's do not. `/checks` lists, and `/checks run` runs, the working folder's as before.
- "folders 3" in the status bar counts the working folder too (hidden when there is only the working folder); in full-screen mode, clicking "Folders" in the right-hand column lists them.
  With the sandbox off the tools have no scope (see "The sandbox" above), and it says "any".

## Slash commands

`/help` lists all commands and keys (for the command line, see `xj --help` or `xj -h`). The common ones:

| Command | What it does |
|---|---|
| `/model [platform:model]` | Switch platforms and models; without an argument, pick a platform, then a model (↑↓ to move, Enter to pick, a digit picks directly, Esc cancels). The choice is saved in the config. See "Switching platforms and models" below |
| `/key [name]` | When a provider has several keys, switch to another (for this conversation only); without an argument, pick from a list (names and where each is kept, never the contents). See "Several keys for one provider" in config.md |
| `/effort low…max` | Reasoning effort; only the levels the current model supports are listed |
| `/undo` | Restore the files `edit` and `write` changed in the last turn; can be repeated (changes made by `bash` cannot be restored) |
| `/diff` | What `edit` and `write` changed in this session |
| `/todo` | Show the model's to-do list again (how far it got); view only, tell it if you want changes |
| `/compact` | Condense the earlier conversation into a summary (it also happens automatically when the context reaches the threshold) |
| `/clear`, `/resume` | Start a new conversation; resume an earlier one in this folder |
| `/checks` | Checks that run automatically after edits (tests, lint); failures go back to the model, up to 3 times. `/checks add <command>` adds one (kept in `~/.xing-janus/checks.json`, per project), `/checks remove <number>` removes one, `/checks off`/`/checks on` skips them for this session/turns them back on, `/checks run` runs them now. When an added folder changes, that folder's checks run (see "Added folders" above) |
| `/image <path>` | Attach an image (you can also drag an image into the terminal, or type `@path` in your message); `/image clear` drops images not sent yet |
| `/mode [mode]` | Switch modes (`normal`, `auto-edit`, `full-auto`, `plan`, `read-only`, `bypass`; Shift-Tab switches too); without an argument, list the current mode, the ones available, and why the others are not available this time. See "Sandbox and modes" above |
| `/sandbox on`, `/sandbox off` | Turn the sandbox on or off (this conversation only; turning it off needs `y`); without an argument, show the current state. See "The sandbox" above |
| `/trust`, `/trust off`, `/add-dir` | Trust this folder, or stop trusting it (both are remembered for next time); `/add-dir <path>` adds a folder for this session (asks whether it is read-only or also editable), `/add-dir` lists them and `/add-dir remove <number>` removes one (see "Added folders" above) |
| `/permissions` | Show permissions: the current state and how to change it, what you always allowed, what the config changed, what is built in; `/permissions remove <number>` removes one, effective in this session too; in full-screen mode a menu follows (`/permissions menu` opens it directly). See "Always allow" above |
| `/config` | Change common settings: lists the current values, pick one to change it, and each says what changing it does (↑↓, Enter, Esc to finish); written to the config, and loosening ones need `y`. See "Changing settings" below |
| `/budget` | The budget: the three limits and what was spent; change one like `/budget day 50`. See "Cost and budget" below |
| `/jobs` | Background jobs: list them, see the output (`/jobs <number>`), stop one (`/jobs stop <number>`). See "Command time limits and background jobs" below |
| `/cost` | The cost of this turn, this session, today and this month |
| `/tui` | Full-screen mode (default) and classic mode: `/tui fullscreen`, `/tui classic` (saved in the config for next time) |
| `/language` | The interface language (`/language ui en`, `/language ui zh-TW`, saved in the config) and the language of AI replies for this session (`/language reply …`) |
| `/notify` | The beep (when a turn finishes or xj needs your answer): `/notify on`, `/notify off`, or no argument to flip it; for this session only (the default comes from `notify` in the config) |
| `/exit` | Quit (pressing Ctrl-C twice with an empty input, or Ctrl-D, quits too) |

Input: Enter sends; type `\` at the end of a line or press Alt-Enter for a new line (pasted multi-line text is kept together). Tab completes slash commands and their arguments (paths for `/image` and `/add-dir`), and ↑ ↓ bring back what you typed before.

## Switching platforms and models (/model)

Type `/model` (without an argument) to pick a platform first, then one of its models. There are four kinds of platform: Anthropic, OpenAI, Gemini, and
OpenAI-compatible services (local ones such as Ollama, LM Studio, llama.cpp and vLLM; hosted ones such as OpenRouter).
The menu shows:

- Whether it is on this computer (`localhost`, `127.0.0.1`, `::1`) or which machine it connects to (the host of its address; an Ollama on another
  computer in your network counts as another machine)
- Where its key is kept, or that it has none yet (Anthropic, OpenAI and Gemini need a key; OpenAI-compatible services may not)
- Servers found on this computer: opening the menu asks the usual ports on this computer only (Ollama 11434, LM Studio 1234, llama.cpp 8080,
  vLLM 8000; it waits less than a second), and lists the ones that answer with a model list; one is added to `providers` in the config only if you pick it
- The last item, "Add an OpenAI-compatible service": a name, an address and a key (optional); it is added to the config only once its model list answers

A platform with no key yet (or a service that refuses requests without one, such as OpenRouter): pick it and paste the key right there.
Only `•` is shown, and the key never enters the input history or the session log. It is used to list the models once and saved only if that works,
in the same place as `xj setup` (the keychain or Credential Manager, or a key file only you can read if that fails).
With an unencrypted `http://` address on another machine, the key would go in plain text, so xj does not send it or set one here (see "Connecting other model services" in config.md).
This does not work when the input is not a terminal (a pipe): use `xj setup`. Nor when that provider is set to use a named key (`keys`):
put that key where "Several keys for one provider" says.

Before switching to another platform, xj lists its models once: if it cannot be reached or the key is refused, nothing changes and you keep the one
you had. If the conversation has content, it asks first: the conversation so far (including file contents it read) is handed over as text to that
platform (the question says which machine), and a long one is first summarized by the current model (one more request). Switching to a platform on
this computer keeps the conversation on this computer, so it only asks when a summary comes first.

The choice is saved as `defaultModel` in the config (like `/tui`), so xj uses it next time too; if that cannot be written, it says so and the
switch is for this session only. Typing `/model platform:model` directly (for example `/model ollama:qwen3`) works the same way: a missing key is
set right there, it checks that the platform answers, asks before sending to another machine, and remembers the choice.

## Typing while it works

While the model is working, the input line stays open (at the very bottom in full-screen mode; on the status-bar line in classic mode):

- Type and press Enter: the line is **queued** (drawn above the input line in full-screen mode) and handed to the model after its current step (before its next tool call); if it has already finished talking, it answers you in the same turn.
  It is printed into the conversation at the moment it is handed over, so the order matches what the model saw.
- Use it to add details, correct something or stop it: "skip item 2", "use TypeScript instead", "do not touch the tests yet".
- Slash commands typed then wait until the turn ends (it never switches models or conversations while the model is halfway through).
- With an empty input line, Ctrl-C interrupts the turn (if there is text, it clears the text first). Lines not yet handed over when you interrupt are not sent automatically; they are listed so you can send them again.
- Ctrl-C while xj is still starting up (for example while the sandbox starts) cleans up and exits without waiting for the startup to finish; a question being asked (such as whether to trust the folder) takes Ctrl-C as no.
- If a permission question pops up while you are halfway through typing, the key you press at that moment does not count as an answer (otherwise a `y` in your sentence would count as yes): pause, read it, then answer.

## The screen: who is speaking

Each line has a column on the left showing who it is:

```
You ▏Please create hello.txt

AI  ▏Sure, here is the plan.
    ▏  Create hello.txt
xj  ▏▍ ── Create hello.txt ──
    ▏▍ Apply?  [y] yes  [n] no  y
AI  ▏    ✓ Created hello.txt (+1 −0 lines)
    ▏Done.
    ▏  This turn about $0.01
```

- **You**: what you type and your slash commands. **AI**: the model's replies, the tools it uses and their results, and the cost of the turn. **xj**: what Xing Janus itself says (replies to slash commands, permission questions, notices and errors).
- The model cannot pretend to be xj: the first columns of every line (5 in the English interface, 4 in Chinese; one more when the terminal draws ambiguous-width characters as two cells, since `▏` then takes two) are always drawn by Xing Janus, and the model's text starts after them; the `▏` bar, like the `▍` of questions, cannot be printed by the model's text, file names or command output.

## Full-screen mode

Full-screen mode is the default: like vim or htop, Xing Janus draws the whole screen. To use the terminal's own scrolling, search, and select-and-copy, type `/tui classic` to switch to classic mode (saved as `screen` in the config for next time); `/tui fullscreen` switches back:

- When the window is wide enough (100 columns or more), there is a column on the right: the to-do list at the top (this turn's subagents and running background jobs are listed there too), and below it the **current status** and common commands. When it is narrower, the to-do list sits above the input line (up to 5 items), and the status is in the status bar at the bottom.
- Each status item has a colored block in front and says what it currently is: Mode, Sandbox, Folders, Trusted, Edits ok (you pressed `a` at a question), Always ok, Today (the budget), Checks, Enter=yes, Beep.
  Blocks: **green** on, **gray** off, **yellow** on but looser (asks less), **red** needs attention (no sandbox, skip-all-permissions mode, a budget limit reached). The words say it all; the colors only help (no colors with `NO_COLOR`).
  - Under the mode, a gray line says what it actually does with the sandbox as it is now (normal mode: "edits ask; whitelist ok"; with the sandbox off: "edits ask; commands ask").
  - With the sandbox off, it says what turned it off ("off (config)", "off (flag)", "off (/sandbox)"; "not installed" when the Windows sandbox is not installed, "failed" when it could not start);
    the line under it gives the network (on: "network: 3 domains"; in skip-all-permissions mode "reaches only 3 domains") or what commands can reach (off: "commands run as you").
  - Folders: "cwd only" when there is only the working folder; otherwise how many including the working folder, with their names on the line under it ("edit: api  read: docs"; yellow when one is editable).
    With the sandbox off it says "any" (each read or edit outside the working folder asks, yellow; in skip-all-permissions mode nothing asks, red).
  - In modes that never ask about edits (auto-edit, full-auto, skip-all-permissions), edits ok says "not needed".
- **Click** (press and release on the same line): the gray text after each item says what a click does (flip, off, list); for items a click cannot change (trusted folder, edits ok when off), the next line says how to change them. Beep and Enter=yes flip on/off (for this session only); mode moves to the next one (like Shift-Tab);
  sandbox works like typing `/sandbox off` or `/sandbox on` (turning it off needs `y`; while the model is working it says to wait for the turn to finish);
  edits ok is taken back with a click when it is on (a click does not turn it on: press `a` at an edit question); trusted folder only says how to change it;
  folders and checks list their contents (like typing `/add-dir` and `/checks`), always ok opens the `/permissions` menu, Today runs `/budget`, and a running background job in the upper part shows its output (`/jobs <number>`);
  while the model is working, these say to look after the turn finishes. Slash commands among the common commands are filled into the input line (read it, then press Enter), while Shift-Tab and Ctrl-C act as if you pressed that key (switch mode, interrupt the turn). After a click, the bottom line says what changed. Mouse codes inside pasted text do not count as clicks.
- When the window is short, the status section first drops the gray line under each item, then the common commands, so the to-do list stays visible.
- Scrolling: PageUp/PageDown and the mouse wheel. While scrolled up, new output does not pull the view down (the bottom says how many lines you have not seen); sending a new message goes back to the bottom.
- **Select and copy**: drag in the conversation column to highlight, release to copy to the clipboard (only the conversation text, without the labels on the left or the column on the right; long lines that were wrapped are joined back). A click or any key clears the highlight.
  While the highlight is shown, Ctrl-C copies instead of interrupting (as in Windows Terminal: Ctrl-C copies when there is a selection and interrupts when there is none); if a turn is running, the status line says to press Ctrl-C again to interrupt it.
  Over SSH, or on Linux without wl-copy/xclip/xsel, it is handed to the terminal (OSC 52: supported by iTerm2, tmux and Windows Terminal, not by Terminal.app).
  To use the terminal's own selection: hold Option while dragging in iTerm2, or type `/tui classic` for classic mode.
- The terminal's own search (Cmd+F) cannot find the conversation inside full-screen mode. When you leave full-screen mode (`/tui classic`, or quitting xj), the whole conversation is printed back to the normal screen, and then search finds it.
- It cannot start when the output is not a terminal, with `TERM=dumb`, in a window that is too small, or in the old Windows console; classic mode is used instead.

## Non-interactive mode (for scripts)

```bash
xj -p "What is this project's test command?"
git diff | xj -p "Write a one-line commit message for this change"
```

The reply goes to stdout; tool actions and the cost go to stderr (`-p` can also be written `--print`). Anything that would ask you (edits, commands outside the whitelist, new domains) is refused; a folder you have never been asked about counts as untrusted.
When the config's default mode is auto-edit or full-auto, `-p` uses normal mode (edits ask, so they are refused); for full-auto, add `--auto` explicitly.
Before the first use, run xj once in a terminal and accept the notice: until then, `xj -p` exits right away (exit 2).

With several keys for one provider, `xj -p --key work "…"` uses the work key (without it, the `keys` setting picks); when that key is rejected and `keys.<provider>.onReject` is `"ask"`, it never switches (nobody can answer).

What comes in through a pipe is data, not your question: `@path` inside it does not attach files. To attach one, put it in the `-p` question (`git diff | xj -p "check this change against @docs/style.md"`).

## Letting other programs call xj (MCP)

`xj mcp` lets programs such as Claude Desktop, IDEs or a chat-app bridge you write call xj through MCP. It is read-only by default and only uses the folder it was started in;
you decide what it may do and which projects it may use with `mcpServer` in the config file. How to connect, the risks of relaxing it, and the interface for plugins: [mcp-server.md](mcp-server.md).

## The status bar

The bottom line: model · key (which one is in use when a provider has several; hidden with only one) · effort · mode · sandbox · folders (only when folders were added: how many including the working folder) · jobs (while background jobs run: how many) · context (used / summarizing threshold) · to-do (when a to-do list has open items, for example "to-do 2/5: Writing tests") · this turn's cost (while the model is working) · today's cost · this month's cost (costs are shown against their limits, for example "today about $3.10/$30"). While working, it shows the latest progress. When the window is too narrow, items are dropped from the end. With the sandbox off it says what turned it off ("sandbox off (config)" and so on); in skip-all-permissions mode without a sandbox, the whole bar is red.
Which items and in what order: `statusBar.fields` in the config (if you wrote that field yourself, add the new `folders`, `jobs` and `turn-cost` to it yourself).

A **`?`** after the context means the model's capabilities could not be found, so conservative defaults are used (32k context, 8k output) —
common for custom services. Put the real values in `capabilities` in the config (see [config.md](config.md)), or you will only use part of the model.

## The to-do list

For multi-step work (changing a feature, fixing a run of tests, several requests at once), the model first writes a to-do list and checks items off as it goes:

```
  Update the to-do list
    ✓ Read the existing code
    ► Write a test
    ☐ Implement it
```

- Every update shows the whole list (in full-screen mode it is drawn on the right or above the input line, not into the conversation); the status bar shows which item it is on and what it is doing. `/todo` shows it again at any time.
- The list only covers the work you asked for this time. When it is done, the to-do item leaves the status bar; ask for something new and it starts a new list (lists do not pile up).
- You do not need to manage the list; just talk to it (also while it works, see "Typing while it works" above): extra requests get added to the list (finished items stay);
  say "skip item 3" or "make the README item English" and it finds the item and changes or removes it. The list is numbered, and the numbers on screen are the ones it sees.
- The list touches no files and runs nothing, so it never asks you (it works in read-only and plan modes too).
- The list exists only on this computer (in xj's memory and in the session logs under `~/.xing-janus/sessions/`, for `/resume`); it is not sent to any other service. The model writes the list, so it is part of the conversation with the model anyway.
- It survives summarizing and `/resume` (after a summary it is shown to the model again, so it does not forget what is left); `/clear` starts a new conversation with an empty list.
- For things done in one step, or plain questions, the model does not write a list. Every model has this tool; to keep a model from using it, set `"todoTool": false` in `capabilities` in the config (for example `"anthropic:claude-opus-5-5": { "todoTool": false }`).

## Fetching web pages

The model can fetch a URL to read (package documentation, changelogs, GitHub issues). The tool is called `web_fetch`:

- GET only, without cookies or any of your login details. Text only (HTML is turned into plain text keeping headings, lists, code and links; scripts and styles are removed);
  images and PDFs are not accepted. Up to 5 MB and 30 seconds per page; up to 40,000 characters go to the model at once, and it can read on for longer pages. Fetching the same URL again within 15 minutes reuses the last result.
- **Domains ask**, using the same list as the sandbox's domains (see "Permission questions" above); the full URL is shown when it asks. Always-allow writes to `sandbox.allowedDomains`.
- **A URL containing something that looks like a key**: asks even for allowed domains, and can only be allowed once (the URL goes to that site, which sees it).
  Disguises count too: an extra character in front, percent-encoding (`%2D`), putting it in the path. Encodings such as base64 cannot be spotted, so allowed domains still need your judgment (see cerberus.md).
- **Blocked without asking**: the local machine (`localhost`, `127.0.0.1`), private addresses written directly (`10.x`, `192.168.x`…), cloud metadata addresses, URLs with a username and password (`user:pass@`),
  those addresses hidden in IPv6 forms (NAT64, 6to4), this machine's own addresses, and port 0;
  names that resolve to the local machine or metadata are blocked too (the connection uses the address that was checked). Names that resolve to private addresses still ask instead of being blocked (otherwise internal documentation sites of a company or organization would not work).
  To fetch from a local dev server, write it in `sandbox.allowedDomains` in the config (for example `127.0.0.1:3000`).
- **Redirects**: followed within the same site (up to 5 times); a redirect to another site is not followed, and the new URL is given to the model; fetching it is a new domain, which asks you again.
- Fetched content is marked as web-page content when given to the model (data, not instructions; see "Prompt injection" in [cerberus.md](cerberus.md)); the screen only shows how much was fetched, not the content.
- It works in read-only and plan modes and in untrusted folders too (it only reads; domains still ask). Full-auto mode still asks about domains; skip-all-permissions mode does not ask without a sandbox, and with the sandbox on only reaches allowed domains (others are blocked without asking); the "blocked without asking" list above applies in every mode.
  Non-interactive mode (`xj -p`) refuses every domain not allowed before.
- There is no search: to have it look something up, give it a URL, or let it work out the documentation URL itself.
- To keep a model from using it: set `"webFetch": false` in `capabilities` in the config.

## Subagents

The model can hand a task to a subagent: the subagent reads, searches and edits in its own context and only hands back a report, so the main conversation is not flooded with all the files it read;
several independent investigations can run at the same time.

- Every model has this tool. It usually works on its own; say so when you want it to split up a search or hand an edit to a subagent ("use two subagents to look into A and B at the same time").
  To keep a model from using it: set `"subagents": false` in `capabilities` in the config (for example `"anthropic:claude-opus-5-5": { "subagents": false }`).
- **Two kinds**: `explore` finds things and reads code, is **always read-only**, and up to 4 can run at once; `general` edits files and runs commands, one at a time.
- **Permissions are the same as the main conversation's**: every action of a subagent goes through Cerberus (the same permissions, sandbox, secret files and allowed domains), never looser than the main conversation:
  - `explore`: read-only even when the main conversation is in full-auto or skip-all-permissions mode.
  - `general`: follows the main conversation's mode (in normal mode, edits still show the diff and ask you); it cannot start in read-only or plan mode, or in an untrusted folder.
  - When it needs to ask, the first line of the question says which subagent it is ("Subagent "Fix the README": Edit README.md"). **You are the one who answers**; the main agent cannot answer for you.
    The exception is a command in the sandbox (such as curl) reaching a new domain: the sandbox's proxy asks, and it only knows the destination, not which subagent's command it is.
  - When several questions pop up in a row (subagents running at the same time), keys pressed within half a second of the next one appearing do not count, so a double press does not answer the next question.
- **On screen**: one line per subagent. In full-screen mode the line updates while it runs (how many actions, how many edits, how much it cost) and turns into the result when done; classic mode prints one line when it starts and one when it finishes.
  In a wide full-screen window, the column on the right lists this turn's subagents under the to-do list (how many are running and how far each got), cleared when the turn ends.
  The individual actions of a subagent are not printed (questions still pop up as usual).
- **xj works out what changed**: when `general` finishes, the main agent gets the report plus xj's own list of "which files changed, lines added and removed" (not the subagent's word),
  and the main agent is told to review those changes and run the tests before replying to you. `/undo` still restores the whole turn (including subagents' `edit` and `write` changes), and the checks after edits still run.
- **Model**: the same as the main conversation by default. To change it, say so ("use Haiku to find it"), or set `subagents.model` in the config (written like `/model`, for example `anthropic:haiku`).
  When it is not the main conversation's model, the line shows it. When the model itself wants another provider's model, you are asked first (the instructions and whatever it reads go to that provider);
  `subagents.model` in the config is your own setting, so it does not ask. Tools the main conversation was not given (for example `web_fetch` turned off in the capabilities) stay unavailable after switching models.
- **Limits**: up to 30 tool calls and 10 minutes per subagent; at the limit it is told to hand back what it found. Its cost counts toward the turn, and the per-turn budget still applies;
  its actions also count toward the per-turn tool-call limit (at the limit it still stops and asks you).
  All of these can be changed under `subagents` in the config (see [config.md](config.md)).
- Subagents cannot start subagents, and cannot use the to-do list, MCP, skills, background jobs or setting proposals. The full run is saved in `subagents/` next to the session log (`/resume` only resumes the main conversation).

## Command time limits and background jobs

- Commands (bash, PowerShell) are stopped after 2 minutes by default (the whole process group), and the model can ask for up to 10 minutes: `commands.timeoutSec` and `commands.maxTimeoutSec` in the config (see [config.md](config.md);
  "Command time limits" in `/config` changes them too, from the next start). The timeout line says which field it is and its current value.
- Long ones (whole test suites, a first build, dev servers) **run in the background**: the model adds `background` to the command and gets a job number right away, then uses `job_output` to see the output or wait for it to finish (no need to keep asking),
  and `job_stop` to stop it. It asks the same way (the question says that it runs in the background, and for how long at most) and runs in the sandbox; it runs for at most 60 minutes (`commands.backgroundMaxMinutes`; 0 means no limit, for example a dev server). At most 8 at a time.
- **Visible and stoppable**: "jobs 2" in the status bar; in full-screen mode, the upper part of the right-hand column lists the running ones (how long they have run; click for the output);
  `/jobs` lists them, `/jobs <number>` shows the end of the output (the full output is in this session's `outputs/job-<number>.log`), `/jobs stop <number>` stops one (the whole process group).
- **Too much output stops it**: a background job can print at most 5 GB (a broken script printing the same error over and over can write over 1 GB a minute); past that it is stopped and xj says why, and its output file keeps only the start and the end.
- **When one finishes**, the conversation gets a line (with a beep if beeps are on and no turn is running). If the model is working, the next tool result tells it; if the turn is over, it is told when you next send something (xj does not start a new turn by itself: that would cost money).
- Background jobs keep the sandbox they started with: to switch the sandbox (`/sandbox`) while some are running, xj asks you and stops them first. `/exit` asks too when some are still running (quitting stops them); when xj ends they are always stopped.
- The Windows sandbox: when it has to re-apply its file permissions (a secret file was replaced by another file, a folder was removed, an editable folder was added), the next command does not run while background jobs are still running,
  and xj says which ones to stop first; stop them and run it again (for the reason, see "Differences of the Windows sandbox from macOS and Linux" in [troubleshooting.md](troubleshooting.md)).
- `xj -p`, `xj mcp` and subagents cannot run things in the background (nobody is watching; a subagent is stopped when its turn ends). The output has keys masked just like any other command.

## Cost and budget

Amounts on screen are marked "about": they are **estimates** (local price table × usage), not a bill — providers change their prices, and their bill is what counts.

There are limits per turn, per day and per month (`budget` in the config; by default $5 per turn, $30 per day, $100 per month; 0 turns that one off). At 80% you get a notice, and at 100% it stops and asks you (saying how to raise the limit).

- **`/budget`**: lists the three limits, what was spent and how to change them (keys with their own limits, when a provider has several, are listed too). `/budget turn 10`, `/budget day 50`, `/budget month 200` change a limit (0 means no limit):
  it first asks whether it is for this conversation only (Enter) or should also be written to the config; writing to the config needs another `y` (only that field, and the whole file is checked before writing). The model cannot change the budget itself (it can propose a change that is written only if you approve, see "Changing settings" below). `xj mcp` still uses `mcpServer.budget`.
- Status bar: "today about $3.10/$30 · month about $12/$100", plus "turn about $0.40/$5" while the model is working (`turn-cost` in `statusBar.fields`).
- "Today" in the right-hand column of full-screen mode: today's spending, with this turn (while working) and this month on the line under it; green normally, yellow when one reaches 80%, red at a limit; clicking it runs `/budget`.
- `/cost` also lists the turn and day limits. `xj setup` does not ask about the budget; at the end it mentions the default limits and `/budget`. The price table is `~/.xing-janus/prices.json`, and every entry records its source and date; `xj doctor` warns if it has not been updated for three months. Local models (Ollama) count as $0. Requests for which the service reports no token usage show the cost as "unknown" (not $0), and if too many pile up in one turn it stops and asks you.

## Changing settings (/config, asking xj)

The config (`~/.xing-janus/config.json`, every field is in [config.md](config.md)) can be changed three ways:

- **`/config`**: lists the common settings with their current values (sandbox at start, default mode, allow full-auto, allow skip all permissions (view only), budget, command time limits,
  interface language, beep, Enter=yes, screen, folders added at start); pick one to change it, and each says what changing it does. Loosening ones (turning the sandbox off, a looser default mode,
  allowing full-auto, raising the budget or time limits, remembering writable folders) need `y` (Enter does not count); only that field changes, and the whole file is checked before writing.
  The budget, interface language, beep, Enter=yes and screen also change in this conversation at once; the rest (sandbox, default mode, allow full-auto, command time limits, folders added at start) apply from the next start
  (in this conversation switch the sandbox and mode with `/sandbox` and Shift-Tab).
  "Folders added at start" remembers the folders added with `/add-dir` in this conversation (added the next time xj starts in this folder, written to `permissions.addDirs`), or removes remembered ones.
- **Asking xj**: just ask, for example "this project needs docker, what should I set?". xj comes with a built-in skill, `xj-settings` (what each setting does, what to set for your situation, and what only you can change),
  which the model reads only when needed; when it does, it sees your current settings: the config as written (keys, MCP servers' environment variables and arguments, and user names, passwords and query strings in URLs hidden),
  plus this conversation's mode, sandbox, folders and budget.
  The model can **propose** changes (`settings_propose`): xj checks them against the config rules, marks each as "loosens", "tightens" or "no effect on permissions" with what it affects, and asks you
  like an edit (loosening ones in red, needing `y`; skip-all-permissions mode asks too); only if you approve does xj write them itself (the model still cannot write the config), and if the config changed while you were being asked, nothing is written.
  Mode, allow full-auto, budget, whitelist (`permissions.allow`) and domain (`sandbox.allowedDomains`) changes also apply to this conversation at once; the rest from the next start (for the sandbox and folders it says how to switch them in this conversation).
  What the model cannot propose (you change it yourself): skip all permissions (`allowBypass`, `defaultMode: "bypass"`), turning off "deleting important places still asks in skip-all-permissions mode",
  removing built-in danger rules or secret files, MCP servers, the `xj mcp` settings, keys and services, environment variables. Whitelist entries and commands outside the sandbox must look like what "always allow" writes
  (a command with at most two subcommands), `*` is not taken as a domain, and paths covering your home folder, credential folders or `~/.xing-janus` are refused.
  In untrusted folders xj does not take proposals that loosen anything (files read there could have asked for them); `xj -p`, `xj mcp` and subagents cannot use it.
- **Editing the config yourself**: open it in an editor such as VS Code for completion and a description of every field (`$schema`); `xj doctor` checks it. Restart xj after changing it.

## Project rules, skills, MCP

- **AGENTS.md** (CLAUDE.md is read too): rules for coding agents, such as the test command or code style. Only read in trusted folders.
- **Skills**: `~/.xing-janus/skills/<name>/SKILL.md`, or `.agents/skills/` and `.claude/skills/` in the project (project skills are only read in trusted folders; on a name clash, yours wins). Normally only the name and description are given to the model, and the content is read when needed.
  xj comes with `xj-settings` (see "Changing settings" above; not listed at startup): a skill of yours with the same name replaces it, a project's cannot.
- **MCP servers**: only the ones listed in the config; they run in the sandbox by default, and every call asks (except those listed in `autoApprove` in the config, and in skip-all-permissions mode).

## Flutter projects

Nothing to set up. With the sandbox on, `flutter` and `dart` cannot run inside it: the model asks to run that one command outside the sandbox and you are asked every time ("Tools that cannot run in the sandbox" in config.md; skip-all-permissions mode blocks it without asking, see "Skip-all-permissions mode" above), using your own environment (`JAVA_HOME` and `ANDROID_HOME` are passed through by default); for the same reason the Windows sandbox no longer opens the Flutter SDK or the Pub cache, so it starts faster. Without the sandbox the usual rules apply.

## Language

The interface (everything xj prints) is in English or Traditional Chinese. On the first start xj asks which one you want (it suggests the one your system uses), and saves it as `uiLanguage` in the config. Switch at any time with `/language ui en` or `/language ui zh-TW`.
The model always gets its instructions and tool results in English; the language it replies in is a separate setting: `language` in the config (by default the same as the interface), or `/language reply <language>` for this session.

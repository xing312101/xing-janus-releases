# Security: what is blocked and what is not

Xing Janus reads files, edits files and runs commands on your computer. Its gatekeeper layer is called Cerberus, and it has three heads:

| | What it does |
|---|---|
| Permission rules | Decide for every action: do it, ask you, or refuse. Secret files (`.env`, private keys, `credentials`…) are blocked for both reading and writing; dangerous commands ask every time |
| OS sandbox | Commands can only read the working folder and added folders (and the toolchains you allow) and only write the working folder, editable folders and temp folders; the network is blocked by default, and a new domain asks you. xj's own tools for reading, editing and searching files stay within the same scope |
| Trusted folders | Things in someone else's repo that would get executed (`package.json` scripts, git hooks) only run directly in folders you trust; a folder added as editable must be trusted too |

**The whitelist only saves you some questions; the safety boundary is the sandbox.** When the sandbox cannot start, every command is refused.
The sandbox and the mode are two separate settings: the sandbox decides what can be reached, the mode only whether xj asks you first, and switching modes (skip all permissions included) never turns the sandbox on or off. What your config loosens (sandbox off, full-auto or skip all permissions allowed…) is listed in the "Relaxed in the config" row of `xj doctor`.
For how each mode behaves with the sandbox on and off, see "Sandbox and modes" in [usage.md](usage.md) (the full tables are only there); for what to set in your situation, see [setup-guide.md](setup-guide.md).
Fetching web pages (`web_fetch`) is a connection xj makes itself, outside the sandbox, but it uses the same domain list and the same address checks as the sandbox library before connecting.

## What it cannot block, and you need to watch

- **Things in the working folder**: the sandbox allows changes to the working folder (and to folders you added as editable), so `rm -rf` and `git reset --hard` are not blocked (they are on the danger list and ask first; skip-all-permissions mode does not ask). Keep your changes in git.
  Secret files too: sandboxed commands cannot read them, but can change them indirectly (for example a program run by `npm test` appending a line to `.env`); only xj's own tools and commands that name the file are blocked from writing them.
- **Folders added as editable (`/add-dir`, `--add-dir`, `permissions.addDirs`)**: they follow the same rules as the working folder, and commands in the sandbox can write there too, so the previous point applies to them as well.
  They must be trusted at the computer (their `package.json` scripts and git hooks will run later); secret files and protected files (that folder's own `.git/hooks`, `.vscode/tasks.json`…) stay blocked.
  Your home folder, a folder above it, the root, folders that hold credentials and `~/.xing-janus` cannot be added at all; a folder containing programs xj runs outside the sandbox (git, rg…), folders on its PATH or xj's own program can only be added read-only
  (commands in the sandbox could replace those programs, which would then run outside the sandbox with your permissions). Add only the folders you need to change: adding a folder above the working folder puts every project in it in scope (see "Added folders" in usage.md).
- **Secret files are recognized by name**: the list is in "The built-in whitelist, danger list and secret files" in config.md. Keys with other names (such as `deploy_key` or `service-account.json`) are not blocked: add them to `permissions.secretFiles`.
- **Prompt injection**: files and web pages can hide instructions telling the AI to do harm. It is taught to treat what it reads as data, not instructions, but that is not guaranteed. Only use it on code you trust.
- **The model proposing settings**: the model cannot write xj's config, only propose changes (`settings_propose`), each marked as loosening, tightening or no effect on permissions, written by xj only if you approve (loosening ones in red, needing `y`). Files and web pages it reads can ask it to propose loosening (prompt injection):
  check that the items in red are what you want. In untrusted folders xj does not take proposals that loosen anything; skip all permissions, removing built-in protections, MCP servers, keys and the like cannot be proposed ("Changing settings" in usage.md).
- **Fetching web pages (`web_fetch`)**: anyone can write a web page, which makes them the most common source of prompt injection; the URL itself can also carry data out (file contents in the query string).
  So every new domain asks you, the full URL is shown when it asks, and URLs that look like they contain a key always ask. **Once a domain is allowed, later URLs are no longer shown to you**:
  choose "allow this session" or "always allow" only for sites you trust (official documentation, package sites); for unknown sites that anyone can set up (personal domains, free hosting), choose "allow once":
  the owner of such a site sees every URL.
  xj fetches pages itself, outside the sandbox; it blocks local and cloud metadata addresses on its own (see "Fetching web pages" in usage.md).
- **Subagents**: every action of a subagent still goes through Cerberus, and the searching kind (explore) is held read-only by Cerberus, not just by its prompt.
  But instructions hidden in the files and pages it reads can slip into the report it hands back: to the main agent the report is data (not something you said), and anything important it was asked to do should be checked.
  When the editing kind (general) finishes, xj lists for the main agent which files were actually changed (it does not rely on the subagent's word); edits ask you according to your mode, so any approval is yours.
- **Secret files tracked in git**: `git diff` and `git log -p` print their contents. Keep secret files out of git.
- **The keychain**: keychain items cannot be read inside the sandbox (tested on macOS 15, 2026-10-17). `security` is also on the danger list and asks first.
- **Turning the sandbox off (`--no-sandbox`, `sandbox.enabled: false` in the config, `/sandbox off`), `sandbox: false` for an MCP server, `sandbox.excludedCommands`**: all of these run programs outside the sandbox with your permissions
  (commands listed in `excludedCommands` ask every time, except in skip-all-permissions mode; see "Docker" in config.md). Only you can turn the sandbox off: the config, the command line, or `/sandbox` typed by you
  (the model can only propose a config change, written only if you approve; files in the project folder cannot turn it off). Without a sandbox, xj's own file tools can also reach outside the working folder:
  each read or edit outside asks (except in skip-all-permissions mode); secret files, places for credentials and xj's own folder stay blocked, and for edits also protected files and places that run things automatically later (shell startup files, services that start at login, scheduled jobs…) (see "The sandbox" in usage.md).
- **Full-auto mode (`--auto`, `allowFullAuto` in the config)**: with the sandbox on, it removes the "look at every command" step, leaving only the sandbox and the danger list (without a sandbox every command still asks; only edits do not).
  Changes in the working folder, and source code you later run outside the sandbox, are yours to review with git (see "Full-auto mode" in usage.md).
- **Skip-all-permissions mode (`--dangerously-skip-permissions`, `allowBypass` in the config)**: nothing asks you (you type a confirmation the first time in each run of xj). It only changes the mode, not the sandbox:
  with the sandbox on, commands stay in the sandbox and the network only reaches allowed domains (others are blocked without asking), but deletions and changes in the working folder and pushes to allowed domains are not stopped;
  without a sandbox, of the three heads only a small part of the permission rules is left (secret files and protected files whose name is written out in the command: in any kind of quotes, listed in braces, or inside `bash -c '…'`, `eval '…'`, `$( )` or a string handed to `cmd` or PowerShell (`cmd //c "type .env"`)), commands run on your computer with your permissions,
  and a name that has to be computed is not caught: `grep -r`, `cat *`, variables, code for another program (`node -e`, `python -c`) or a script piped into a shell (`echo 'cat .env' | sh`) can read `.env`: only use it in a container or virtual machine you can throw away.
  Either way these still ask: switching to root, deleting an important place (the root, a top-level folder, your home folder, the working folder or a folder above it, an added folder; `permissions.bypassAsksCriticalDelete` in the config can turn this off),
  your own entries in `permissions.danger`, and commands xj cannot check all the way through (nested too deeply, or braces that expand to too many words: what was not checked is not treated as fine). They are recognized from the words of the command, so a program written to delete things is not recognized (see "Skip-all-permissions mode" in usage.md).
- **Shared Gradle and pub caches**: Flutter/Android builds need to write `~/.gradle` and `~/.pub-cache`, so the sandbox can write them.
  The places that run code automatically (init scripts, `wrapper/dists`, `pub-cache/bin`) are blocked separately, but jars and package sources in the caches can still be changed, and your own next build runs them.
  If that worries you, empty `sandbox.writable` in the config and give each project its own `GRADLE_USER_HOME` and `PUB_CACHE`.
- **The npm cache**: your home folder is blocked in the sandbox, so npm in the sandbox does not use the machine-wide `~/.npm` but `~/.xing-janus-cache/npm` (Xing Janus's own).
  That means the first install downloads packages again, and programs in the sandbox cannot change the cache you use yourself. `~/.npmrc` cannot be read (it may contain tokens),
  so projects that need a private registry should put that in the project's own `.npmrc`.
- **It will not start in your home folder**: when the working folder is your home folder, above it, or `/`, Xing Janus refuses to start (that would hand over your whole home folder). cd into a project folder.
- **Windows** (2026-10-20; the sandbox is alpha): without the sandbox, every command asks you (the whitelist does not apply). The PowerShell whitelist only accepts very simple forms
  (2026-10-21): read-only cmdlets such as `Get-ChildItem`, `Get-Content` and `Select-String`, with literal parameter values (no `$` variables, parentheses, double quotes
  or redirection), and paths inside the project (or folders you opened for reading). `Select-String` after `Get-ChildItem` or `Get-Item` does not count (it opens those files to search them).
  For any other form xj cannot tell what will actually run, so it always asks, even in full-auto mode.
  The sandbox runs as another account that cannot read your home folder anyway; but creating protected files that do not exist yet cannot be blocked (xj warns about this). Other limits: see "Windows" in troubleshooting.md.
- **Windows: folders any logged-in account can change** (tested 2026-10-27): the sandbox blocks writes using the sandbox account's own permissions, so "it is yours" does not mean the sandbox cannot touch it —
  the root of a non-system drive (`D:\` lets any authenticated user modify it by default) and folders you created under `C:\` (such as `C:\tools`, and everything inside them, which inherit from `C:\`) are writable from inside the sandbox.
  When the sandbox starts, xj actually tries to write there: **if a place that runs outside the sandbox is writable, the whitelist stops applying and every command asks you first** (the screen and `xj doctor` list which folders and how to block them with `icacls`).
  It tries the folders on your PATH, the root of every toolchain the sandbox can read (such as `C:\src\flutter`), Git Bash's own folders, and the folder and SDK of each `excludedCommands` program.
  A writable drive root is only reported (that is the Windows default) and means programs in the sandbox can change **other projects** anywhere under that drive root (folders there inherit the root's permissions).
  If that worries you: install tools under `Program Files` (normal users cannot change it), or deny the sandbox account on those folders as administrator.
  The `icacls` line in the warning names **the highest folder above it that is writable all the way up** (for example `C:\src`, not just `bin`): blocking there covers the whole toolchain below.
  Blocking only `bin` would leave the rest of the toolchain changeable from the sandbox, and the changed program runs the next time it runs outside the sandbox (through `excludedCommands`, or when you run it yourself outside xj).
  It never tells you to block your home folder, the project, or a folder above them (that would also block what xj needs to write).
  **Programs in those places that run outside the sandbox** (xj's own git, the Git Bash that runs outside-sandbox commands, `excludedCommands`, tools such as flutter that cannot run in the sandbox, MCP servers, the rg used by search)
  can be replaced by commands in the sandbox, and what xj then runs outside the sandbox with your permissions is the replaced one. When the sandbox starts, xj asks you once whether to run them outside the sandbox anyway in this conversation
  (with "no" they do not run, and search runs inside the sandbox instead), and commands that need your approval still ask every time, marked red. `sandbox.writablePrograms` in the config can make it not ask and not run them (`"deny"`) or not ask and run them (`"allow"`); see config.md.
- **Windows: with the sandbox on, do not run another xj in an untrusted project at the same time**: every xj sandbox uses the same `srt-sandbox` account,
  so while this xj is running, commands in the other xj's sandbox get the same access this xj opened to the sandbox (such as the writable project folder and the toolchains), until this xj quits and the permissions are taken back.
  Secret files stay blocked: when xj starts the sandbox, it blocks secret files before it opens any folders. There are a few very short exceptions: at the moment xj re-applies the blocks after a secret file was replaced by another file (it takes them all off and puts them back),
  a command that happens to be running in another xj's sandbox can read secret files and change protected ones.
  The "cannot delete" on the folders that hold protected files (such as `.git`) is there twice, for the sandbox account and for the sandbox group, and goes on before any folder is opened:
  whenever srt-win changes permissions (also when another xj makes it do so), it removes the account's entry but leaves the group's alone, so another xj's sandbox cannot rename the folders inside the project either;
  and nobody can rename the working folder itself while xj is running in it.
  The exception is an editable `/add-dir` folder itself: after another xj touches it (preparing the sandbox again, turning its sandbox off), renaming it is not blocked until this xj re-applies its blocks
  (and only if its parent folder lets anyone create folders, such as folders you made directly under `C:\` or on a data drive).
  This xj's own background jobs cannot: while they are running, xj does not re-apply anything and asks you to stop them first (see "Differences of the Windows sandbox from macOS and Linux" in troubleshooting.md).
- **`flutter` and `dart` (and `fvm`) cannot run in the sandbox** (on every platform): they write into the Flutter SDK's own `bin/cache` on every run, which the sandbox does not allow.
  On Windows, for example (tested 2026-10-27), their `.bat` wrappers need to write `bin\cache` (a lock file and various `.stamp` files),
  the SDK is read-only in the sandbox, and `flutter.bat` then hangs. xj does not let it hang: the model asks to run that one command outside the sandbox, and you are asked every time ("Tools that cannot run in the sandbox" in config.md).
  Making the whole `bin\cache` writable is not an option: the `dart.exe`, snapshots and artifacts in there are later run outside the sandbox with your permissions.
- **Background processes that detach themselves**: when xj quits (including when it is force-quit), it stops the commands (background jobs included, `/jobs`) and MCP servers it started, as whole groups. But processes a command detaches on purpose (with `setsid` or `set -m`, or services that turn themselves into daemons) cannot be reached. In the Linux sandbox they end together with the command; on macOS they are left running, and you need to find and stop them with `ps`.
  Windows is stricter (tested 2026-10-22): every program a command starts is in the same job object and cannot leave it, so **everything stops as soon as the command ends**,
  including windowed programs (also those started with `start`) and long-running ones such as the Gradle daemon; the same with the sandbox on (it has its own job object).
  The exception is the Microsoft Store version of PowerShell 7 with the sandbox off (tested 2026-11-05): Store apps run in a job object Windows gives them, which lets child processes leave it,
  so the programs it starts are not in xj's job object. On Ctrl-C or timeout xj stops the process tree by parent (the running PowerShell and what it started); programs left behind after the command ends, or whose parent has already exited, are not stopped (as on macOS).
  To have everything stopped, install PowerShell 7 with the official MSI (into Program Files).
  Start programs you want to keep after xj quits (dev servers, emulators, editors) yourself, outside xj (a dev server you need while xj is open can be run as a background job by the model); programs that are already running (for example when a command only asks the browser to open a new tab) are not affected.

How to report a security issue: see [SECURITY.md](SECURITY.md).

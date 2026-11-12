# The config file

Location: `~/.xing-janus/config.json`. The full format is in [`config.schema.json`](../config.schema.json), and `xj doctor` checks against it; its "Relaxed in the config" row lists what is looser than the defaults (the sandbox turned off, `allowBypass`, a `"*"` domain, removed built-in rules…).
**Every setting has a default, so xj works with nothing written at all**: put only what you want to change in the file (`xj setup` writes the default model, the interface and reply languages, and the sound). The "Default" column is what is used when a field is left out, or how xj decides it.
`xj setup` writes `"$schema"` into the file: open it in an editor such as VS Code to get completion, a description for every field, and typos marked as you type.
You can also change it inside xj: the `/config` menu (the common settings), or ask the model to propose changes for your situation (`settings_propose`; xj writes them only if you approve); which settings each can change is under "Changing settings inside xj" below, and how to use them under "Changing settings" in usage.md.
Choosing "always allow" at a question writes to `permissions.allow`, `sandbox.allowedDomains` or an MCP server's `autoApprove`, and for the two questions the Windows sandbox start asks, to `sandbox.autoFixWritable` or `sandbox.writablePrograms` (only that field changes, the whole file is checked before writing, and nothing is written if the file itself has errors); see and remove them with `/permissions`.

| Field | Default | Description |
|---|---|---|
| `version` | `1` | The format version of the file: xj writes it and converts old files itself, so you never change it |
| `defaultModel` | `anthropic:claude-opus-5-5` | The default provider and model. A series name (`anthropic:opus`) always uses the latest one. The model part is the name the service gives it (for example a file path for llama.cpp), without control or invisible characters. `xj setup` rewrites this for the provider you pick, and so does switching with `/model` |
| `effort` | `{ "default": "high" }` | The default reasoning effort per model; keys are model IDs, series names or `default`. If a model does not support that level, the model's own default is used; models without reasoning effort get none |
| `defaultMode` | `normal` | `normal`, `auto-edit`, `full-auto` (the same as always adding `--auto`, see usage.md), `plan`, `read-only`, `bypass` (skip all permissions: needs `allowBypass`, without which xj starts in normal mode and says why; you still confirm by typing when xj starts, and cancelling also starts in normal mode). `xj -p` ignores `auto-edit`, `full-auto` and `bypass` (it uses normal mode) and needs the explicit flags |
| `allowFullAuto` | `false` | `true`: Shift-Tab and `/mode` can reach full-auto mode (the same as always adding `--auto`, without entering it). Without it, `--auto` at startup or `defaultMode` set to `full-auto` count too |
| `allowBypass` | `false` | `true`: Shift-Tab and `/mode` can reach skip-all-permissions mode; the first time you enter it after starting xj you confirm by typing. Running as root or administrator is still refused; `xj -p` and `xj mcp` ignore it. Without it, `--dangerously-skip-permissions` at startup counts too |
| `uiLanguage` | Your system's language | The interface language: `en` or `zh-TW`. If unset, xj looks at the first of the environment variables `LC_ALL`, `LC_MESSAGES` and `LANG` that is set (`C` and `POSIX` do not count), and otherwise at the operating system's region setting: anything starting with `zh` gives Traditional Chinese, everything else English. Asked on the first start (with that preselected); `/language ui` changes it |
| `language` | Follows the interface language (`English` for the English interface, Traditional Chinese for the Chinese one) | The language AI replies in, written as you would say it (for example `English`, `Deutsch`) |
| `notify` | `false` | Beep when a turn finishes or xj needs your answer |
| `screen` | `fullscreen` | The screen: `fullscreen` is full-screen mode (in wide windows the to-do list and current status are on the right; PageUp/PageDown and the wheel scroll); `classic` is classic mode (the terminal scrolls, searches and copies). When full-screen mode cannot start (output is not a terminal, the old Windows console), classic mode is used. `/tui` writes it when you switch |
| `enterToApprove` | `true` | Enter means yes at questions (on by default, as in Claude Code; set it to `false` to always press y) ("allow once" for domain questions). Dangerous commands, content you have not seen in full, the always-allow confirmation, loosening changes (`/sandbox off`, loosening settings in `/config` and in the model's proposals) and writing `/budget` to the config still need `y`; an Enter within 0.5 seconds of a question appearing does not count (you may have been typing). Yes/no questions (trusting a folder, whether an added folder is editable too) cannot be answered with Enter |
| `statusBar.fields` | `model`, `key`, `effort`, `mode`, `sandbox`, `folders`, `jobs`, `context`, `todo`, `turn-cost`, `day-cost`, `month-cost`, `progress` | Which items the status bar shows, in order of importance: when the window is too narrow, items are dropped from the end (the model is always shown). There is also `session-cost` (the cost of this session). `key` is which key is in use when a provider has several (it takes no space with only one key); `folders` is how many folders are in scope, counting the working folder ("folders 3": added folders and `extraReadable` ones count; it takes no space with only the working folder); `jobs` is how many background jobs are running ("jobs 2"; no space when none are); `todo` is the to-do list (no space when there is no list or everything is done); `turn-cost` is this turn's spending and limit (only while the model is working); `day-cost` and `month-cost` are today's and this month's spending and limits. If you wrote this field yourself, items added in later versions only show once you add them (0.1.1 added `key`; 0.1.5 added `folders`, `jobs` and `turn-cost`) |
| `budget` | $5 per turn, $30 per day, $100 per month, billing cycle starts on the 1st | `perTurnUsd`, `perDayUsd`, `perMonthUsd` (US dollars), `cycleStartDay` (the day of the month the billing cycle starts, 1–28), `unknownRequestsPerTurn` (after how many requests with no known price in one turn xj stops and asks, 10 by default: their cost cannot be worked out, so the dollar limits cannot catch them; set 0 if you use a local service and do not want to be asked, or put 0 for that model in the price table). A notice at 80%, a stop-and-ask at 100%. The limits are compared against the **estimated** cost (local price table × usage), not the provider's bill; 0 turns that one off. `xj setup` does not ask about this; in xj, `/budget` shows and changes them (optionally for this conversation only), and `/config` changes the per-turn, per-day and per-month limits too (this conversation uses the new limit as well). The model cannot change them itself (it can propose a change you approve). See "Cost and budget" in usage.md |
| `commands` | 2 minutes in the foreground, at most 10 minutes; 60 minutes in the background | Time limits for commands (bash, PowerShell): `timeoutSec` (how long a command runs when the model does not set a timeout, in seconds), `maxTimeoutSec` (the longest timeout the model can set, in seconds; not shorter than `timeoutSec`), `backgroundMaxMinutes` (the longest a background job runs, in minutes; 0 means no limit, for example a dev server). The model cannot change them itself (it can propose a change you approve); `/config` changes them. Changes apply from the next start. See "Command time limits and background jobs" in usage.md |
| `limits` | 100 tool calls per turn, at most 64000 output tokens per request | `toolCallsPerTurn`: the most tool calls per turn; at the limit it stops and asks you. `maxOutputTokens` (at least 1024): the output limit for each request, never above the model's own limit; reaching it does not stop and ask (a cut-off reply gets a notice, and cut-off tool calls are not run) |
| `permissions` | Every field empty (the built-in whitelist, danger list and secret files still apply) | **The built-in lists always apply** (see "The built-in whitelist, danger list and secret files" below); write only what you want to add or remove here: `allow`/`allowRemove` (whitelist), `danger`/`dangerRemove` (danger list; removals by name), `secretFiles`/`secretFilesRemove` (secret files). `extraReadable`: folders outside the working folder it may also read (your home folder itself, credential folders such as `~/.ssh` and `~/.aws`, xj's own `~/.xing-janus`, and the folders containing them are not used; xj says so at startup). `addDirs`: folders to add when xj starts, see "Folders added at startup" below. `bypassAsksCriticalDelete` (default `true`): in skip-all-permissions mode, deleting an important place (the root, a folder directly under it, your home folder, the working folder, an added folder, or a folder above them) still asks; `false` stops asking ("Skip-all-permissions mode" in usage.md). If you do not want anything looser or stricter, leave it out |
| `sandbox.toolchainReadable` | `~/.gitconfig`, `~/.config/git`, `~/.cargo`, `~/.rustup`, `~/.pyenv`, `~/.nvm`, `~/.volta`, `~/development/flutter`, `~/fvm`, `~/.pub-cache`, `~/.gradle`, `~/.android`, `~/Library/Android/sdk` (missing ones are skipped) | Toolchains commands in the sandbox can read (outside the working folder; the rest of your home folder is not readable). `~/.gitconfig` and `~/.config/git` are only opened when `sandbox.gitconfig` is `"full"` (with the default `"filtered"`, the sandbox gets a filtered copy). Windows: opening a folder means changing the permissions of every file in it, so with the default value, folders are opened by project type (npm's global folder only with a `package.json`, the Android SDK and `~/.gradle` only with Gradle files such as `build.gradle`…; when the project type cannot be told yet (a brand-new empty folder), the Node and Python ones are opened anyway; Flutter projects (`pubspec.yaml`) do not open the Flutter SDK or the Pub cache, and their `android/` folder does not count either — flutter and dart always run outside the sandbox, so opening them would only slow down the start), and large toolchains only partly; if you write this field or `writable` yourself, it is used as written and xj adds nothing |
| `sandbox.writable` | `~/.pub-cache`, `~/.gradle` | Where commands in the sandbox can write (the working folder and temp folders are always writable; no need to list them) |
| `sandbox.enabled` | `true` | Whether xj starts the sandbox. `--sandbox` and `--no-sandbox` override it; skip-all-permissions mode does not change it (the sandbox and the mode are two separate settings). With `false`, commands run with your permissions and the whitelist does not apply (every command asks, except in skip-all-permissions mode), and the read and edit tools can reach outside the working folder too (each one asks, except in skip-all-permissions mode; what is always blocked stays blocked, see "The sandbox" in usage.md). `xj mcp` always starts it. In a container where the sandbox cannot start, write `false` in that machine's config. Inside xj, `/sandbox off` and `/sandbox on` switch it (that conversation only; this setting is not changed) |
| `sandbox.allowedDomains` | Empty | Domains commands in the sandbox and `web_fetch` may reach without asking. Empty means xj asks the first time a command connects to each new domain (allow once, for this conversation, or always; "always" writes it here); in skip-all-permissions mode with the sandbox on nothing asks, and domains not allowed before are blocked. `example.com` covers every port, `example.com:8080` only that port; `*.github.com` covers its subdomains (not `github.com` itself); `"*"` is any domain, in every mode (the sandbox then does not restrict the network, so skip-all-permissions mode blocks nothing either; addresses on this machine, private network addresses and cloud metadata are still blocked) |
| `sandbox.allowLocalBinding` | `false` | Let commands in the sandbox listen on local ports (set `true` if tools such as Flutter or Gradle need it). **On macOS this also lets them connect to every local port on this computer** (services that only listen locally, such as Ollama, databases, Docker), without the domain prompt; turn it on only while you need it |
| `sandbox.gitconfig` | `filtered` | Which global config git reads in the sandbox: `"filtered"` keeps only harmless settings such as `user.name` and `user.email` (your aliases and `insteadOf` do not apply in the sandbox), `"full"` is the whole `~/.gitconfig` (commands in the sandbox can then read any URLs and tokens in it) |
| `sandbox.excludedCommands` | Empty | Commands that run outside the sandbox (asked every time), see "Docker" below |
| `sandbox.allowUnsandboxedCommands` | `true` | Whether tools that cannot run in the sandbox (such as flutter) may ask to run that one command outside it (asked every time), see "Tools that cannot run in the sandbox" below |
| `sandbox.autoFixWritable` | `false` | Windows: when the sandbox start finds places on your PATH the sandbox account can write to, block them as administrator without asking (UAC still asks). `false` means xj asks you once, and "always allow" writes `true`; remove it with `/permissions`; see "Skip-all-permissions mode" in usage.md |
| `sandbox.writablePrograms` | `ask` | Windows: what to do when programs that run outside the sandbox (xj's git, Git Bash, `excludedCommands`, tools such as flutter, MCP servers, the rg used by search) are in folders the sandbox can write to — `"ask"` asks once when the sandbox starts, for that conversation; `"deny"` does not ask and does not run them outside the sandbox (search runs inside the sandbox instead); `"allow"` does not ask and runs them (still marked red every time). "Always allow" writes `"allow"`; remove it with `/permissions`; see cerberus.md |
| `windows` | `{}` | Windows only: `gitBash` is the full path of Git Bash's `bash.exe`, `powershell` the full path of `pwsh.exe` or `powershell.exe`. Without them xj looks for them (Git Bash via git.exe on PATH and the default install locations; PowerShell 7 first, then the built-in one; the Microsoft Store 7 counts too, but not while the sandbox is on). They cannot be files in the project, nor `C:\Windows\System32\bash.exe` (that is WSL) |
| `env.passthrough` | `JAVA_HOME`, `ANDROID_HOME` | Commands run in a clean environment: only `PATH`, `HOME`, `LANG`, `TERM` and `TMPDIR` are kept (on Windows also the few the system needs, such as `SystemRoot`), plus the variables listed here; keys, tokens and the like are not passed on |
| `keyFiles` | Empty (keys are looked up in environment variables, then the keychain or Credential Manager) | Without a keychain: provider → key file. Paths must be absolute or start with `~` (on Windows with a drive letter, for example `C:\keys\anthropic.txt`): a relative path would be read from the working folder, so the config rejects it. These paths automatically count as secret files. For several keys of one provider, write the key as `provider:name` (for example `anthropic:work`) |
| `keys` | Empty (one key per provider) | Several keys for one provider: which one each folder uses, what to do when one is rejected, a limit for one key. See "Several keys for one provider" below |
| `providers` | Built in: `openai` (`https://api.openai.com/v1`), `ollama` (`http://localhost:11434/v1`), `gemini` (Google's Gemini API, see "Gemini" below) | Change `baseURL`, turn one off with `enabled: false`, or add your own service (`kind: "chat-completions"`, `baseURL`, `keyName`, `allowPlainHttpKey`; see "Connecting other model services" below). OpenAI and Gemini are not connected without a key; your own services are connected even without one (local services often need none); Ollama needs no key. Giving `gemini` a `baseURL` makes it use that address's OpenAI-compatible endpoint instead |
| `capabilities` | Empty (what each provider's model list reports; for models it does not know, conservative defaults: 32k context, 8k output, with `?` in the status bar) | Override a model's capabilities (context, maximum output, toolset…); keys are "provider:model": `contextTokens`, `maxOutputTokens`, `compactAtTokens` (the context size in tokens at which it summarizes automatically; by default 150,000, but no more than 80% of the context), `imageInput` (whether it accepts images), `tools` (`"none"` sends no tools; `"reliable"` and `"unreliable"` currently make no difference; Anthropic models ignore this field), `parallelToolCalls` (currently has no effect), `toolset` (`"full"`, or `"reduced"`: only read, search and edit). `todoTool`: whether this model gets the to-do list tool (`todo_write`); `webFetch`: whether it gets the web-fetch tool (`web_fetch`). Every model gets them by default; write `false` to keep a model from using one. `subagents`: whether it gets the subagent tool (`agent`); also on by default, write `false` to turn it off for a model |
| `subagents` | 4 at once, 30 tool calls, 10 minutes | Subagents (see "Subagents" in usage.md): `maxConcurrent` how many run at once (only read-only explore subagents run in parallel), `maxToolCalls` and `timeoutMinutes` the limits for each (at the limit it is told to report), `model` the default model (without it, the main conversation's) |
| `mcp.servers` | Empty (no MCP servers) | MCP servers: `command`, `args`, `env`, `description`, `sandbox`, `autoApprove`, `readOnly`, `timeoutSec` (how many seconds one call may take, 600 by default). Values in `env` whose names look like keys (ending in `_KEY`, `_TOKEN`, `_SECRET`, `_PASSWORD` and so on) are treated as keys: masked wherever they appear in the server's output, error messages and session logs. The Windows sandbox cannot run MCP servers yet (stdin cannot be connected): with the sandbox on, set `sandbox: false` (it then runs outside the sandbox with your permissions) |
| `mcpServer` | Read-only, nothing outside the sandbox, only the folder `xj mcp` was started in, spending $2 a turn / $10 a day | What other programs may do when they call xj through MCP (`xj mcp`): `mode`, `outsideSandbox`, `projects`, `anyProject`, `budget`. Read the risks in [mcp-server.md](mcp-server.md) before relaxing anything |
| `anthropic` | `fallback: default`, `prefixMismatch: drop_block` | Anthropic-specific settings. `fallback`: `"default"` lets Anthropic's server retry with another model when the request is blocked by the safety classifier, and that model answers instead (the status bar shows "→ the model that answered", and that model's price is used); `"off"` means no switch and no retry. `prefixMismatch`: when the earlier conversation changed (for example after summarizing) and no longer matches the model's reasoning, `"drop_block"` drops that reasoning and sends the request anyway, `"error"` fails instead (for testing) |
| `evals.budgetUsd` | `2` | For development: the most each eval run may spend (US dollars). Evals (`node evals/run.ts`) only exist in the source repo, not in the released version |
| `sessions.retentionDays` | `30` | How many days session logs are kept: older ones are removed at every start (`xj sessions prune` does it by hand too) |

## Changing settings inside xj

Both ways have xj write the config itself (only those fields change, the whole file is checked before writing, and nothing is written if the file itself has errors); loosening changes need `y`. How to use them: "Changing settings" in usage.md.

- **`/config`** can change: `sandbox.enabled`, `defaultMode` (not to or from `bypass`; `full-auto` needs `allowFullAuto` first), `allowFullAuto`, the per-turn, per-day and per-month `budget`, `commands`,
  `uiLanguage`, `notify`, `enterToApprove`, `screen`, `permissions.addDirs` (the entry for the folder you opened xj in); `allowBypass` is only shown.
- **The model can propose** (`settings_propose`, written only if you approve): `sandbox.enabled`, `defaultMode` (except `bypass`), `allowFullAuto`;
  in `permissions`: `allow`, `allowRemove`, `danger`, `secretFiles`, `extraReadable`, `addDirs` (the entry for the folder you opened xj in), `bypassAsksCriticalDelete` (only `true`);
  in `sandbox`: `allowedDomains` (not `"*"`), `writable`, `toolchainReadable`, `excludedCommands`, `allowLocalBinding`, `allowUnsandboxedCommands`;
  every field of `budget`, `commands` and `limits`, and `maxConcurrent`, `maxToolCalls` and `timeoutMinutes` in `subagents`; `defaultModel`, `effort.default`, `language`, `uiLanguage`, `notify`, `enterToApprove`, `screen`, `statusBar.fields`.
  In untrusted folders nothing that loosens is accepted; `xj -p`, `xj mcp` and subagents cannot propose.
- **The model cannot propose these; you change them yourself**: `allowBypass`, `defaultMode: "bypass"`, `permissions.bypassAsksCriticalDelete: false`, `permissions.dangerRemove`, `permissions.secretFilesRemove`,
  `mcp`, `mcpServer`, `providers`, `keys`, `keyFiles`, `env`, and everything not in the previous item (`sandbox.gitconfig`, `windows`, `capabilities`, `anthropic`, `sessions`…).

## The built-in whitelist, danger list and secret files

These apply without any setting; `permissions` only holds what you add or remove.

- **Whitelist** (runs without asking while the sandbox is on; without a sandbox — `--no-sandbox`, `sandbox.enabled` set to `false`, `/sandbox off` in the conversation, or Windows without the sandbox installed — it does not apply and every command asks): the read-only `git status`, `git diff`, `git log`, `git show`, `git branch`, `git rev-parse`, `git ls-files`, `git blame`, `ls`, `pwd`, `cat`, `head`, `tail`, `wc`, `file`, `stat`, `which`, `echo`, `date`, `tree`, `du`, `df`, `rg`, `grep`, `find`;
  and for tests and checks `npm test`, `npm run test`, `npm run lint`, `npm run typecheck`, `npm run build`, `npx tsc`, `npx vitest`, `npx eslint`, `npx prettier`, `pytest`, `cargo test`, `go test`, `flutter test`, `flutter analyze`, `dart analyze`, `dart test`, `dart format`.
  Some also check their arguments (for example `find -delete` and `git branch -D` do not count). To add one: `permissions.allow` (for example `"make test"`); to remove one: `permissions.allowRemove` (for example `"npm run build"`).
- **Danger list** (asked every time and marked in red with the reason; only `y` counts; full-auto mode asks too; read-only and plan modes refuse them; skip-all-permissions mode does not ask, except for switching to root or administrator and what you added to `permissions.danger`; deleting an important place also still asks there, see "Skip-all-permissions mode" in usage.md).
  To remove an entry, put its name in `permissions.dangerRemove`; to add one, put the start of the command in `permissions.danger` (for example `"terraform apply"`).
  Matching goes by the command's words: every part of a pipeline, `&&`, `bash -c`, `env` and `xargs` is checked; done through a program (`python -c "…"`, `node -e "…"`) it is not recognized, and then the sandbox is what limits it:

  | Name | What it catches |
  |---|---|
  | `rm-recursive-force` | Deleting files (`rm` with `-r`, `-R`, `-f`, `-d`, or `--recursive`, `--force`, `--dir`) |
  | `find-delete` | Deleting files (`find -delete`) |
  | `shred` | Destroying files (`shred`) |
  | `git-reset-hard` | Throwing away changes (`git reset --hard`, `--merge`, `--keep`) |
  | `git-checkout-discard` | Throwing away changes (`git checkout -- …`, `git checkout .`, `git checkout -f`) |
  | `git-restore` | Throwing away changes (`git restore`) |
  | `git-clean` | Deleting untracked files (`git clean -f`) |
  | `git-stash-drop` | Throwing away stashed changes (`git stash drop`, `git stash clear`) |
  | `git-branch-delete` | Deleting a branch (`git branch -d`, `-D`, `--delete`) |
  | `git-push` | Pushing to a remote (`git push`, including `--force`) |
  | `publish` | Publishing a package (`publish` of `npm`, `yarn`, `pnpm`, `bun`, `cargo`; `dart pub publish`, `flutter pub publish`, `twine upload`, `gem push`) |
  | `remote-shell` | Connecting to another machine (`ssh`, `scp`, `rsync`, `sftp`) |
  | `upload` | Sending data out: `curl` with data (`-d`, `-F`, `-T`, `--data…`, `--form`, `--upload-file`, `--json`) or a method other than GET and HEAD (such as `-X POST`); `wget` with `--post-data`, `--post-file`, `--body-data`, `--body-file`, or a `--method` other than GET and HEAD |
  | `sudo` | Running as root (`sudo`, `su`, `doas`, `sudoedit`, `pkexec`, `run0`) |
  | `chmod-chown-recursive` | Changing permissions recursively (`chmod`, `chown`, `chgrp` with `-R` or `--recursive`) |
  | `disk` | Writing to a disk directly (`mkfs`, `dd`, `diskutil`) |
  | `power` | Shutting down or restarting the machine (`shutdown`, `reboot`, `halt`, `poweroff`) |
  | `kill` | Force-killing programs (`kill -9`, `-KILL`, `-SIGKILL`, `-s`; `killall`, `pkill`) |
  | `system-launch` | Asking the system to run something, possibly outside the sandbox (`open`, `osascript`, `launchctl`, `crontab`, `at`, `xdg-open`, `systemctl`, `systemd-run`) |
  | `windows-launch` | Asking Windows to run something, possibly outside the sandbox (`explorer`, `rundll32`, `mshta`, `wscript`, `cscript`, `msiexec`) |
  | `windows-elevate` | Running as administrator (`gsudo`, `runas`; handing a command to `cmd`, `powershell` or `pwsh` with `runas`, or a PowerShell command whose content cannot be read, such as `-EncodedCommand`) |
  | `windows-shell` | Handing the command to cmd or PowerShell (`cmd`, `powershell`, `pwsh`: the commands inside are not checked one by one) |
  | `windows-system` | Changing Windows system settings: registry, scheduled tasks, services, permissions, environment variables, firewall (`reg`, `regedit`, `schtasks`, `sc`, `setx`, `icacls`, `takeown`, `cacls`, `bcdedit`, `netsh`, `wmic`, `vssadmin`, `diskpart`, `cipher`, `wevtutil`, `certutil`, `bitsadmin`) |
  | `keychain` | Accessing the keychain or Credential Manager (`security`, `cmdkey`, `vaultcmd`) |
  | `system-dirs` | Writing to system folders with `cp`, `mv`, `tee`, `install`, `ln`, `touch`, `mkdir`, `rm`, `rmdir`, `chmod`, `chown`, `truncate`: `/etc`, `/usr`, `/bin`, `/sbin`, `/lib`, `/opt`, `/boot`, `/var`, `/private`, `/System`, `/Library`, `/Applications` (except macOS's temporary folders), and on Windows `C:\Windows`, `Program Files`, `ProgramData` |

  Writing to those system folders with a `>` redirect counts too (that one cannot be removed with `dangerRemove`).
  PowerShell commands are checked against a list of their own (no names; `dangerRemove` cannot remove them): recursive deletion (`Remove-Item -Recurse` and aliases such as `rm`, `del`, `rd`),
  `Invoke-Expression` (`iex`), network access (`Invoke-WebRequest`, `Invoke-RestMethod`, `Start-BitsTransfer`), starting other programs (`Start-Process`),
  registry changes (`Set-ItemProperty` and similar on `HKLM:`, `HKCU:`), system settings (`Set-ExecutionPolicy`, Defender settings, scheduled tasks, services),
  reading or writing Credential Manager (`CredRead` and similar, `PasswordVault`, `Get-StoredCredential`, `cmdkey`, `vaultcmd`), commands whose content cannot be read (`-EncodedCommand`),
  switching to administrator (`-Verb RunAs`, `runas`, `sudo`, `gsudo`), and writing to system folders.

- **Secret files** (reading, writing and searching are all blocked, though sandboxed commands can still change them indirectly: see cerberus.md; answering yes in the conversation does not help, only the config file can change it): `.env`, `.env.*`, `*.pem`, `*.key`, `*.p12`, `*.pfx`, `*.keystore`, `*.jks`, `*.p8`, `*.ppk` (PuTTY),
  SSH keys (`id_rsa`, `id_ed25519`, `id_ecdsa`, `id_dsa`, `id_ed25519_sk`, `id_ecdsa_sk` and their `.pub`), `credentials`, `credentials.json`, `credentials.toml`, `.npmrc`, `.netrc`, `.pypirc`, `.git-credentials`, `.envrc`,
  Android/Flutter's `local.properties`, `key.properties`, `keystore.properties`, `secrets.properties`, `adbkey`, and in your home folder `~/.gradle/gradle.properties`, `~/.docker/config.json`, `~/.kube/config`, `~/.config/gh/hosts.yml`, `~/.m2/settings.xml`, and the `npmrc` files inside nvm and npm,
  plus the key files named in `keyFiles`. Not secret: `.env.example`, `.env.sample`, `.env.template`, `~/.android/debug.keystore`. To add: `permissions.secretFiles` (for example `"*.secret"`; patterns ignore case, `*` is any characters (within one folder; `**` crosses folders), `?` is one character, `[abc]` is one of those, `[!abc]` or `[^abc]` is anything but those); to remove: `permissions.secretFilesRemove` (for example `".npmrc"`).

## Tools that cannot run in the sandbox (Flutter)

Some tools simply cannot run inside the sandbox: `flutter` and `dart` (and `fvm`) write into the Flutter SDK's own folder (`bin/cache`) on every run; the sandbox does not allow that, so they hang.
Opening that folder is not an option either: the `dart.exe` and friends in there are later run outside the sandbox with your permissions, so programs in the sandbox could change them (tested on Windows).

So (the way Claude Code does it): when the model calls one of these with the sandbox on, xj does not let it hang; it tells the model why, the model asks to **run that one command outside the sandbox**,
and xj asks you once (in red: "runs outside the sandbox with your permissions", and which program actually runs). It runs only if you answer yes. No config change, no turning the sandbox off; other commands stay in the sandbox.

- It asks every time, also in full-auto mode, and there is no "allow permanently"; like `excludedCommands`, only simple forms are allowed (it must run on its own, not chained after other commands).
- In skip-all-permissions mode with the sandbox on: it is neither run outside the sandbox nor asked about; it is blocked, with a note on what to do (switch to a mode that asks; tools such as flutter can also be listed in `excludedCommands`; turning the sandbox off is the last resort).
- Only tools on this list can ask to run outside the sandbox (currently flutter, dart, fvm, and installing, below); any other command asking for it is refused. Similar tools will be added to the same list.

**Installing tools**: when a tool is missing on this computer, the model offers to install it the same way: you are asked every time, and it runs outside the sandbox only if you answer yes.
- Only `winget install --scope user …` and `scoop install …` on Windows (plus lookups such as their search), and `brew install …` on macOS. Other installers (choco, `npm -g`, pip…) are refused.
- `winget install` must use `--scope user` (installs under your account, no administrator rights).
- **No administrator rights by default**: if they are needed, the model gives you the full command to run in an administrator terminal yourself. Only when you have explicitly said in the conversation that it may run it directly does the model ask to run it as administrator
  (`winget install` only): xj still asks you (in red), and Windows shows a UAC prompt too; this also applies in skip-all-permissions mode without a sandbox (with the sandbox on it is blocked, as above).
- **Restart xj** afterwards: PATH and what the sandbox can read are worked out at start, so a newly installed command is only found after a restart.
- What winget installs under your account (`--scope user`) can be read in the sandbox: with `sandbox.toolchainReadable` left at its default, xj opens `%LOCALAPPDATA%\Microsoft\WinGet\Packages` and `Links` to the sandbox for reading (as it does for Scoop; if you write that field yourself, list them in it).
  **What is installed for the whole computer as administrator cannot be used inside the sandbox**: winget gives those files no access for ordinary users, and the sandbox runs as another account (tested). It works outside the sandbox; to use it inside, install with `--scope user`.
- If you do not want this option: set `"sandbox": { "allowUnsandboxedCommands": false }` in the config, and these commands are always refused; to use them, turn the sandbox off with `/sandbox off` in xj (or restart with `xj --no-sandbox`), or list them in `excludedCommands`.
- Commands you listed in `excludedCommands` yourself work as before (asked, run outside the sandbox) without the model asking.

## Docker: commands that run outside the sandbox

The sandbox blocks the connection to Docker (`docker.sock`: whoever can tell Docker what to do can mount the whole computer's files into a container), so `docker` always fails inside the sandbox.
To let it use Docker without turning the whole sandbox off, list the command in `sandbox.excludedCommands`:

```json
{ "sandbox": { "excludedCommands": ["docker"] } }
```

- `"docker"` covers all of docker; `"docker compose"` covers only that subcommand. Write the command name, not a path.
- Listed commands **ask every time**, marked in red with "runs outside the sandbox with your permissions" and which program actually runs; nothing runs until you say yes. Full-auto mode asks too, and there is no allow-all; skip-all-permissions mode does not ask and runs them outside the sandbox directly.
- Only simple forms are allowed: you can chain several listed commands with `&&` and add `2>&1` or `>/dev/null`; you cannot pipe into other commands, write to files,
  use `$( )`, or put environment variables in front (`PATH=` and `LD_PRELOAD=` would swap the program that actually runs). Forms that are not allowed are refused directly, with a note on how to rewrite them.
- The program is looked up on the PATH outside the sandbox: files of the same name in the project or temp folders (which commands in the sandbox could put there) do not count.
- Nothing runs outside the sandbox in untrusted folders or in read-only and plan modes. Secret files are still always blocked.
- Docker acts on the project's `Dockerfile` and `docker-compose.yml`, which commands in the sandbox can change: look at `git diff` before saying yes.
- To reach a remote Docker (`DOCKER_HOST`), add the variable to `env.passthrough`.

Skip-all-permissions mode (`--dangerously-skip-permissions`) cannot start as root by default (the same goes for a terminal opened with "Run as administrator" on Windows); if you are in a throwaway container or VM and must use root (administrator),
set the environment variable `XING_JANUS_ALLOW_ROOT_BYPASS=1` (only `1` counts). Like the other `XING_JANUS_` variables, it is not passed to the commands xj runs.

Keys are not written in the config: put them in the keychain (`xing-janus-<provider>`), the environment variable `XING_JANUS_<PROVIDER>_API_KEY`, or a file named in `keyFiles`. Do not use generic names such as `ANTHROPIC_API_KEY` or `OPENAI_API_KEY`: other tools read them too.

## Folders added at startup

If you always open xj in A and also edit B and read C, write them in `permissions.addDirs` (only in your own config: files in a project cannot add folders):

```json
{
  "permissions": {
    "addDirs": {
      "~/work/app": [
        { "path": "~/work/api", "write": true },
        { "path": "~/work/docs" }
      ]
    }
  }
}
```

- The key is the folder you open xj in: opening xj there or in any folder under it counts (compared by real path, so symlinks resolve the same way); when several keys match, all of them are added, in the order written.
- The values are the folders to add: `write: true` means editable (like `/add-dir` with "editable": the folder must be trusted at the computer, xj asks at startup if it is not, and if you say no it is read-only); without it, read-only.
- Every path must be absolute or start with `~` (a relative path would change with where you open xj).
- The same rules as `/add-dir` apply ("Added folders" in usage.md): your home folder, a parent of it, the root and credential folders are not added; secret files and protected files stay blocked.
  Folders added at startup stay for new conversations (`/clear`, `/resume`); only `/add-dir remove` takes them out (for that run of xj).
- For a folder you need just this once, use `xj --add-dir <path>` on the command line (editable; repeat for more). `xj -p` adds both too, but cannot ask about trust, so untrusted ones are read-only.
  `xj mcp` does not take `--add-dir` and ignores `permissions.addDirs` (the folders it can reach are `mcpServer.projects`).
- Inside xj: "Folders added at start" in `/config` remembers the folders added with `/add-dir` in this conversation in the entry for this folder, or removes remembered ones (entries for parent folders are only listed);
  the model can also propose changes to this entry (written only if you approve).

## Several keys for one provider

A work key and a personal one, one per client: give each key a name (lowercase letters, digits and `-`, for example `work`). The unnamed key is called `default`, and it is the key you already have: **with only one key, nothing changes**, and the screen looks the same as before.

Named keys are kept the same way, with the name added after the usual one:

| Where | The unnamed key | The key called work |
|---|---|---|
| Environment variable | `XING_JANUS_ANTHROPIC_API_KEY` | `XING_JANUS_ANTHROPIC_API_KEY_WORK` (`-` in the name becomes `_`) |
| macOS keychain, Windows Credential Manager | `xing-janus-anthropic` | `xing-janus-anthropic--work` |
| Key file | `"keyFiles": { "anthropic": "…" }` | `"keyFiles": { "anthropic:work": "…" }` |

`xj setup` asks whether to add another key. The name has to appear in the config file (`byFolder`, `failover` or `names` under `keys`, or `provider:name` in `keyFiles`) for xj to know about the key: it does not go looking through environment variables or the keychain. Keys added by `xj setup` are written there for you.

```json
{
  "keys": {
    "anthropic": {
      "default": "personal",
      "byFolder": { "~/work": "work", "~/clients/acme": "acme" },
      "onReject": "ask",
      "failover": ["backup"],
      "budget": { "work": { "perDayUsd": 10 } }
    }
  }
}
```

- **Which key is used** (in this order): `--key work` on the command line (`--key openai:work` for another provider) → the one picked with `/key` in this conversation → the `byFolder` match (that folder and its subfolders; start with `~` or use an absolute path, globs work too; when several match, the longest wins; real paths are compared, so folders reached through a symlink (such as `~/Dropbox` on macOS) match too) → `default` → the unnamed key. **Only your own config file counts**: files in a project cannot decide whose money is spent. If the chosen key cannot be found, xj does not fall back to another one (it may be billed to someone else); it says where to put it.
- **`/key`**: lists this provider's keys (names and where each is kept, never the contents) and switches to one (for this conversation only). Switching keys often means switching accounts: the cache breaks (the next request costs more), and the available models may differ (the model list is cached per key).
- **On screen**: the opening line, the status bar (the `key` item) and `/cost` show which key is in use. The transcript records only the name, never the key. `/resume` uses the key this folder's setting picks now, and says so if that differs from the original conversation.
- **Costs**: recorded per provider and name, and listed separately by `/cost`. `budget` still covers everything; `keys.<provider>.budget` adds a daily (`perDayUsd`) or monthly (`perMonthUsd`) limit for one key, with the same warning first and then a question.
- **When a key is rejected** (quota used up, invalid key, still rate-limited after retries): `onReject` is `"ask"` (the default: asks whether to switch to the next key and carry on with the turn), `"auto"` (switches and says so) or `"off"` (stops; switch yourself with `/key`). xj only switches to keys **this folder may use**: the `byFolder` matches for this folder plus those in `failover` (in order). It never switches to another bill just to carry on. The switch lasts for this conversation only; with `xj -p` nobody can answer, so `"ask"` never switches.
- Before choosing `"auto"`, think about two things: rotating several keys automatically to get around rate limits **may break the provider's terms** (check them first), and after a switch, costs go on that key's bill. `xj doctor` reminds you too.
- Subagents use the same key as the main conversation. `xj -p` and evals (for development, `node evals/run.ts --key work`) take `--key` as well. `xj doctor` shows one row per key (found or not, where, whether it connects).

## Gemini

Built in, connecting to Google's Gemini API (a native adapter: thought signatures are sent back unchanged, so long runs of tool calls keep working).

- **Key**: create a key restricted to the Gemini API in Google AI Studio (or the Google Cloud console). Store it with `xj setup` (choose 3, Gemini, in the first question), or set the
  environment variable `XING_JANUS_GEMINI_API_KEY`, or the keychain (Credential Manager) item `xing-janus-gemini`. Do not use `GEMINI_API_KEY` or
  `GOOGLE_API_KEY`: other tools read them too (xj does not). New keys starting with `AQ.` and older ones starting with `AIza` both work.
- **Models**: `gemini:gemini-flash-latest` (Google's alias that always points at the newest Flash), `gemini:gemini-3.5-flash-lite`, `gemini:gemini-3.1-pro-preview`…;
  `/model` lists the ones your key can use (speech, image and music models, which are not for conversations, are left out).
- **Reasoning effort**: from Gemini 3 on, `low` and `high` (`/effort`, `high` by default); 2.5 uses the server's default. While the model thinks, Google sometimes sends a summary of its thoughts; xj shows its beginning as progress.
- **What is sent**: if the Cloud project that owns the key has billing turned on (paid), Google does not use what you send to improve its products;
  **with free-tier keys it does**, and people may review it (the Gemini API additional terms). Use a paid key for code you do not want used to improve products or reviewed by people; code that must not leave your company should not go to any cloud model (use a local one).
- **Prices**: the default price table has the main Gemini models (copied from Google's pricing page). Prices discounted for a limited time say until which day
  they apply (`until`) and the regular price after that (`after`); xj switches by date (see "Time-limited prices" below).
  If your `~/.xing-janus/prices.json` was created by an older xj it has no Gemini prices and costs show as "unknown": `xj doctor` says how to get the new table.
- **If you connected the compatible endpoint before** (`providers.gemini` with a `baseURL`, as this page used to say): that keeps using the compatible endpoint.
  To use the native one, remove `providers.gemini`. People on Google's forum report that keys AI Studio issues now, starting with `AQ.`,
  fail on the compatible endpoint (`Multiple authentication credentials received`).

## Connecting other model services

Any OpenAI-compatible API works by adding a `providers` entry, with no code changes. Put the key in the environment variable
`XING_JANUS_<NAME IN CAPITALS>_API_KEY`, the keychain or Credential Manager (the item is named `xing-janus-<name>`), or a file named in `keyFiles`.
`xj setup` asks about Anthropic, OpenAI, Gemini, or a local or other OpenAI-compatible service (choice 4: it lists Ollama and the local servers found on this computer, the last item lets you enter an address; then a name and whether it needs a key); for services you add later, set the key right inside xj (paste it when you pick or add the service in `/model`),
or store it yourself (macOS: `security add-generic-password -s xing-janus-<name> -a "$USER" -w`; Windows: `cmdkey /generic:xing-janus-<name> /user:xj /pass`).
You can also add one inside xj: the last item of the `/model` menu, "Add an OpenAI-compatible service", or a server "found on this computer"
("Switching platforms and models" in usage.md). It is written to `providers` here (only `kind` and `baseURL`) once its model list answers;
a key set there is stored the same way as with `xj setup`.
A name may use lowercase letters, digits and `-` (starting with a lowercase letter, at most 32 characters), but not `--` (it separates named keys, so the name would share another provider's key);
reserved names such as `prototype`, `constructor`, `con` and `nul` are refused (they cannot be used in the config file or as Windows file names).

```json
{
  "providers": {
    "openrouter": { "kind": "chat-completions", "baseURL": "https://openrouter.ai/api/v1" },
    "local": { "kind": "chat-completions", "baseURL": "http://localhost:8080/v1" }
  },
  "capabilities": {
    "openrouter:anthropic/claude-sonnet-4.5": { "contextTokens": 200000, "maxOutputTokens": 32000 },
    "local:your-model-id": { "contextTokens": 65536, "maxOutputTokens": 8192, "toolset": "reduced" }
  },
  "defaultModel": "openrouter:anthropic/claude-sonnet-4.5"
}
```

Three things to watch:

1. **Fill in the capabilities yourself.** When a model's capabilities cannot be found, conservative defaults are used (**32k context**, 8k output). So a model with a 1M context
   is treated as 32k unless you write `capabilities`. Model IDs with slashes are fine.
2. **Costs show as "unknown".** Costs come from `~/.xing-janus/prices.json` (see the example under "Time-limited prices" below), so add an entry for your own service
   (for local models, 0 everywhere; otherwise once 10 requests in one turn have no known price, it stops and asks whether to continue).
3. **Scale local models down**: `"toolset": "reduced"` (only read, search and edit).
   Ollama gets this automatically (small models often cannot call tools correctly), but "your own service" (LM Studio, llama.cpp, vLLM…) is treated as a cloud API, so write it yourself.

Local services (llama.cpp's `llama-server`, vLLM, LM Studio) just need `http://localhost:<port>/v1`,
without a key; `http://` only escapes the warning for localhost.
Unencrypted `http://` to another machine (a vLLM on another computer on your network, say): the key would go in plain text, so xj does not send it
(at startup, when switching to it inside xj, when setting a key there, and in `xj doctor`); services that need no key work as usual.
To send it anyway, add `"allowPlainHttpKey": true` to that provider (anyone on the same network can see the key).
Without `--alias`, llama.cpp's `llama-server` names the model after the file given to `-m` (for example `C:\models\qwen2.5-7b-instruct-q4_k_m.gguf`):
that still works and is remembered, but for a shorter name start it with `--alias qwen` and pick it with `/model llamacpp:qwen`
(`llamacpp` is the name `/model` gives it when it finds the server; if you added it yourself, use your name).

## Time-limited prices

A model's entry in the price table (`~/.xing-janus/prices.json`) can say until which day a discount applies: the entry's own prices (`inputPerMTok` and so on)
apply through the `until` day, and the prices in `after` from the next day. Days are counted in UTC (official pages usually give US time: UTC switches to the
later price a few hours early, which overestimates rather than underestimates). Write both together; if either has a mistake the entry is not used (as if it had
no price), and xj says so when it starts and in `xj doctor`. For example, this entry from the default price table (under `providers.gemini`):

```json
"gemini-4-argon": {
  "inputPerMTok": 2, "outputPerMTok": 10, "cacheReadPerMTok": 0.1,
  "until": "2026-11-30",
  "after": { "inputPerMTok": 4, "outputPerMTok": 20, "cacheReadPerMTok": 0.2 },
  "source": "https://aistudio.google.com/docs/pricing", "asOf": "2026-11-04"
}
```

`after` can have its own `longContext` (the prices above a prompt-length threshold); without it, there is no separate long-context price after that day.

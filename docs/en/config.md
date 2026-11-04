# The config file

Location: `~/.xing-janus/config.json`. The full format is in [`config.schema.json`](../config.schema.json), and `xj doctor` checks against it.
**Every setting has a default, so xj works with nothing written at all**: put only what you want to change in the file (`xj setup` writes the default model, the interface and reply languages, and the sound). The "Default" column is what is used when a field is left out, or how xj decides it.
`xj setup` writes `"$schema"` into the file: open it in an editor such as VS Code to get completion, a description for every field, and typos marked as you type.
Choosing "always allow" at a question writes to `permissions.allow`, `sandbox.allowedDomains` or an MCP server's `autoApprove`, and for the two questions the Windows sandbox start asks, to `sandbox.autoFixWritable` or `sandbox.writablePrograms` (only that field changes, the whole file is checked before writing, and nothing is written if the file itself has errors); see and remove them with `/permissions`.

| Field | Default | Description |
|---|---|---|
| `version` | `1` | The format version of the file: xj writes it and converts old files itself, so you never change it |
| `defaultModel` | `anthropic:claude-opus-5-5` | The default provider and model. A series name (`anthropic:opus`) always uses the latest one. The model part is the name the service gives it (for example a file path for llama.cpp), without control or invisible characters. `xj setup` rewrites this for the provider you pick, and so does switching with `/model` |
| `effort` | `{ "default": "high" }` | The default reasoning effort per model; keys are model IDs, series names or `default`. If a model does not support that level, the model's own default is used; models without reasoning effort get none |
| `defaultMode` | `normal` | `normal`, `auto-edit`, `full-auto` (the same as always adding `--auto`, see usage.md), `plan`, `read-only`. `xj -p` ignores `auto-edit` and `full-auto` (it uses normal mode) and needs an explicit `--auto` for full-auto. Skip-all-permissions mode cannot be the default: it needs `--dangerously-skip-permissions` every time |
| `uiLanguage` | Your system's language | The interface language: `en` or `zh-TW`. If unset, xj looks at the first of the environment variables `LC_ALL`, `LC_MESSAGES` and `LANG` that is set (`C` and `POSIX` do not count), and otherwise at the operating system's region setting: anything starting with `zh` gives Traditional Chinese, everything else English. Asked on the first start (with that preselected); `/language ui` changes it |
| `language` | Follows the interface language (`English` for the English interface, Traditional Chinese for the Chinese one) | The language AI replies in, written as you would say it (for example `English`, `Deutsch`) |
| `notify` | `false` | Beep when a turn finishes or xj needs your answer |
| `screen` | `fullscreen` | The screen: `fullscreen` is full-screen mode (in wide windows the to-do list and current status are on the right; PageUp/PageDown and the wheel scroll); `classic` is classic mode (the terminal scrolls, searches and copies). When full-screen mode cannot start (output is not a terminal, the old Windows console), classic mode is used. `/tui` writes it when you switch |
| `enterToApprove` | `true` | Enter means yes at questions (on by default, as in Claude Code; set it to `false` to always press y) ("allow once" for domain questions). Dangerous commands, content you have not seen in full and the always-allow confirmation still need `y`; an Enter within 0.5 seconds of a question appearing does not count (you may have been typing). The trust-this-folder question cannot be answered with Enter |
| `statusBar.fields` | `model`, `key`, `effort`, `mode`, `sandbox`, `context`, `todo`, `day-cost`, `month-cost`, `progress` | Which items the status bar shows, in order of importance: when the window is too narrow, items are dropped from the end. There is also `session-cost` (the cost of this session). `todo` is the to-do list (it takes no space when there is no list or everything is done; added 2026-10-22, so if you wrote this field yourself before that, add `todo` to see it). `key` is which key is in use when a provider has several (it takes no space with only one key; added 2026-10-29, so add it yourself if you wrote this field before) |
| `budget` | $5 per turn, $30 per day, $100 per month, billing cycle starts on the 1st | `perTurnUsd`, `perDayUsd`, `perMonthUsd` (US dollars), `cycleStartDay` (the day of the month the billing cycle starts, 1–28). A notice at 80%, a stop-and-ask at 100%. The limits are compared against the **estimated** cost (local price table × usage), not the provider's bill; 0 turns that one off. `xj setup` does not ask about this |
| `limits` | 50 tool calls per turn, at most 64000 output tokens per request | `toolCallsPerTurn`: the most tool calls per turn; at the limit it stops and asks you. `maxOutputTokens` (at least 1024): the output limit for each request, never above the model's own limit; reaching it does not stop and ask (a cut-off reply gets a notice, and cut-off tool calls are not run) |
| `permissions` | Every field empty (the built-in whitelist, danger list and secret files still apply) | **The built-in lists always apply** (see "The built-in whitelist, danger list and secret files" below); write only what you want to add or remove here: `allow`/`allowRemove` (whitelist), `danger`/`dangerRemove` (danger list; removals by name), `secretFiles`/`secretFilesRemove` (secret files). `extraReadable`: folders outside the working folder it may also read (your home folder itself, credential folders such as `~/.ssh` and `~/.aws`, and the folders containing them are not used; xj says so at startup). If you do not want anything looser or stricter, leave it out |
| `sandbox.toolchainReadable` | `~/.gitconfig`, `~/.config/git`, `~/.cargo`, `~/.rustup`, `~/.pyenv`, `~/.nvm`, `~/.volta`, `~/development/flutter`, `~/fvm`, `~/.pub-cache`, `~/.gradle`, `~/.android`, `~/Library/Android/sdk` (missing ones are skipped) | Toolchains commands in the sandbox can read (outside the working folder; the rest of your home folder is not readable). `~/.gitconfig` and `~/.config/git` are only opened when `sandbox.gitconfig` is `"full"` (with the default `"filtered"`, the sandbox gets a filtered copy). Windows (2026-10-21): opening a folder means changing the permissions of every file in it, so with the default value, folders are opened by project type (npm's global folder only with a `package.json`, the Android SDK and `~/.gradle` only with Gradle files such as `build.gradle`…; when the project type cannot be told yet (a brand-new empty folder), the Node and Python ones are opened anyway; Flutter projects (`pubspec.yaml`) do not open the Flutter SDK or the Pub cache, and their `android/` folder does not count either — flutter and dart always run outside the sandbox, so opening them would only slow down the start), and large toolchains only partly; if you write this field or `writable` yourself, it is used as written and xj adds nothing |
| `sandbox.writable` | `~/.pub-cache`, `~/.gradle` | Where commands in the sandbox can write (the working folder and temp folders are always writable; no need to list them) |
| `sandbox.allowedDomains` | Empty | Domains commands in the sandbox may reach without asking. Empty means xj asks the first time a command connects to each new domain (allow once, for this conversation, or always; "always" writes it here) |
| `sandbox.allowLocalBinding` | `false` | Let commands in the sandbox listen on local ports (set `true` if tools such as Flutter or Gradle need it). **On macOS this also lets them connect to every local port on this computer** (services that only listen locally, such as Ollama, databases, Docker), without the domain prompt; turn it on only while you need it |
| `sandbox.gitconfig` | `filtered` | Which global config git reads in the sandbox: `"filtered"` keeps only harmless settings such as `user.name` and `user.email` (your aliases and `insteadOf` do not apply in the sandbox), `"full"` is the whole `~/.gitconfig` (commands in the sandbox can then read any URLs and tokens in it) |
| `sandbox.excludedCommands` | Empty | Commands that run outside the sandbox (asked every time), see "Docker" below |
| `sandbox.allowUnsandboxedCommands` | `true` | Whether tools that cannot run in the sandbox (such as flutter) may ask to run that one command outside it (asked every time), see "Tools that cannot run in the sandbox" below |
| `sandbox.autoFixWritable` | `false` | Windows: when the sandbox start finds places on your PATH the sandbox account can write to, block them as administrator without asking (UAC still asks). `false` means xj asks you once, and "always allow" writes `true`; remove it with `/permissions`; see "Skip-all-permissions mode" in usage.md |
| `sandbox.writablePrograms` | `ask` | Windows: what to do when programs that run outside the sandbox (xj's git, Git Bash, `excludedCommands`, tools such as flutter, MCP servers, the rg used by search) are in folders the sandbox can write to — `"ask"` asks once when the sandbox starts, for that conversation; `"deny"` does not ask and does not run them outside the sandbox (search runs inside the sandbox instead); `"allow"` does not ask and runs them (still marked red every time). "Always allow" writes `"allow"`; remove it with `/permissions`; see cerberus.md |
| `windows` | `{}` | Windows only (2026-10-20): `gitBash` is the full path of Git Bash's `bash.exe`, `powershell` the full path of `pwsh.exe` or `powershell.exe`. Without them xj looks for them (Git Bash via git.exe on PATH and the default install locations; PowerShell 7 first, then the built-in one). They cannot be files in the project, nor `C:\Windows\System32\bash.exe` (that is WSL) |
| `env.passthrough` | `JAVA_HOME`, `ANDROID_HOME` | Commands run in a clean environment: only `PATH`, `HOME`, `LANG`, `TERM` and `TMPDIR` are kept (on Windows also the few the system needs, such as `SystemRoot`), plus the variables listed here; keys, tokens and the like are not passed on |
| `keyFiles` | Empty (keys are looked up in environment variables, then the keychain or Credential Manager) | Without a keychain: provider → key file. Paths must be absolute or start with `~` (on Windows with a drive letter, for example `C:\keys\anthropic.txt`): a relative path would be read from the working folder, so the config rejects it. These paths automatically count as secret files. For several keys of one provider, write the key as `provider:name` (for example `anthropic:work`) |
| `keys` | Empty (one key per provider) | Several keys for one provider: which one each folder uses, what to do when one is rejected, a limit for one key. See "Several keys for one provider" below |
| `providers` | Built in: `openai` (`https://api.openai.com/v1`), `ollama` (`http://localhost:11434/v1`) | Change `baseURL`, turn one off with `enabled: false`, or add your own service (`kind: "chat-completions"`, `baseURL`, `keyName`). OpenAI is not connected without a key; your own services are connected even without one (local services often need none); Ollama needs no key |
| `capabilities` | Empty (what each provider's model list reports; for models it does not know, conservative defaults: 32k context, 8k output, with `?` in the status bar) | Override a model's capabilities (context, maximum output, toolset…); keys are "provider:model": `contextTokens`, `maxOutputTokens`, `compactAtTokens` (the context size in tokens at which it summarizes automatically; by default 150,000, but no more than 80% of the context), `imageInput` (whether it accepts images), `tools` (`"none"` sends no tools; `"reliable"` and `"unreliable"` currently make no difference; Anthropic models ignore this field), `parallelToolCalls` (currently has no effect), `toolset` (`"full"`, or `"reduced"`: only read, search and edit). `todoTool`: whether this model gets the to-do list tool (`todo_write`); `webFetch`: whether it gets the web-fetch tool (`web_fetch`). Every model gets them by default; write `false` to keep a model from using one. `subagents`: whether it gets the subagent tool (`agent`); also on by default, write `false` to turn it off for a model |
| `subagents` | 4 at once, 30 tool calls, 10 minutes | Subagents (see "Subagents" in usage.md): `maxConcurrent` how many run at once (only read-only explore subagents run in parallel), `maxToolCalls` and `timeoutMinutes` the limits for each (at the limit it is told to report), `model` the default model (without it, the main conversation's) |
| `mcp.servers` | Empty (no MCP servers) | MCP servers: `command`, `args`, `env`, `description`, `sandbox`, `autoApprove`, `readOnly`, `timeoutSec` (how many seconds one call may take, 600 by default). Values in `env` whose names look like keys (ending in `_KEY`, `_TOKEN`, `_SECRET`, `_PASSWORD` and so on) are treated as keys: masked wherever they appear in the server's output, error messages and session logs. The Windows sandbox cannot run MCP servers yet (stdin cannot be connected): with the sandbox on, set `sandbox: false` (it then runs outside the sandbox with your permissions) |
| `anthropic` | `fallback: default`, `prefixMismatch: drop_block` | Anthropic-specific settings. `fallback`: `"default"` lets Anthropic's server retry with another model when the request is blocked by the safety classifier, and that model answers instead (the status bar shows "→ the model that answered", and that model's price is used); `"off"` means no switch and no retry. `prefixMismatch`: when the earlier conversation changed (for example after summarizing) and no longer matches the model's reasoning, `"drop_block"` drops that reasoning and sends the request anyway, `"error"` fails instead (for testing) |
| `evals.budgetUsd` | `2` | For development: the most each eval run may spend (US dollars). Evals (`node evals/run.ts`) only exist in the source repo, not in the released version |
| `sessions.retentionDays` | `30` | How many days session logs are kept: older ones are removed at every start (`xj sessions prune` does it by hand too) |

## The built-in whitelist, danger list and secret files

These apply without any setting; `permissions` only holds what you add or remove.

- **Whitelist** (runs without asking while the sandbox is on; without a sandbox — `--no-sandbox`, or Windows without the sandbox installed — it does not apply and every command asks): the read-only `git status`, `git diff`, `git log`, `git show`, `git branch`, `git rev-parse`, `git ls-files`, `git blame`, `ls`, `pwd`, `cat`, `head`, `tail`, `wc`, `file`, `stat`, `which`, `echo`, `date`, `tree`, `du`, `df`, `rg`, `grep`, `find`;
  and for tests and checks `npm test`, `npm run test`, `npm run lint`, `npm run typecheck`, `npm run build`, `npx tsc`, `npx vitest`, `npx eslint`, `npx prettier`, `pytest`, `cargo test`, `go test`, `flutter test`, `flutter analyze`, `dart analyze`, `dart test`, `dart format`.
  Some also check their arguments (for example `find -delete` and `git branch -D` do not count). To add one: `permissions.allow` (for example `"make test"`); to remove one: `permissions.allowRemove` (for example `"npm run build"`).
- **Danger list** (asked every time, in full-auto mode too). To remove an entry, put its name in `permissions.dangerRemove`; to add one, put the start of the command in `permissions.danger` (for example `"terraform apply"`):

  | Name | What it catches |
  |---|---|
  | `rm-recursive-force` | Deleting files (`rm -r`/`-f`/`-d`) |
  | `find-delete` | Deleting files (`find -delete`) |
  | `shred` | Destroying files |
  | `git-reset-hard`, `git-checkout-discard`, `git-restore` | Throwing away changes (`git reset --hard`, `git checkout -- …`, `git restore`) |
  | `git-clean` | Deleting untracked files (`git clean -f`) |
  | `git-stash-drop` | Throwing away stashed changes (`git stash drop`/`clear`) |
  | `git-branch-delete` | Deleting a branch (`git branch -d`/`-D`) |
  | `git-push` | Pushing to a remote |
  | `publish` | Publishing a package |
  | `remote-shell` | Connecting to another machine (`ssh`, `scp`, `rsync`, `sftp`) |
  | `upload` | Sending data out (`curl`/`wget` uploads) |
  | `sudo` | Running as root (`sudo`, `su`, `doas`, `sudoedit`, `pkexec`, `run0`) |
  | `chmod-chown-recursive` | Changing permissions recursively (`chmod`/`chown -R`) |
  | `disk` | Writing to a disk directly (`mkfs`, `dd`) |
  | `power` | Shutting down or restarting the machine |
  | `kill` | Force-killing programs (`kill -9`, `killall`) |
  | `system-launch` | Asking the system to run something, possibly outside the sandbox (`open`, `osascript`, `launchctl`, `crontab`) |
  | `windows-launch` | Asking Windows to run something, possibly outside the sandbox (`explorer`, `rundll32`, `mshta`, `wscript`) |
  | `windows-elevate` | Running as administrator (`gsudo`, `runas`, `-Verb RunAs`, or a PowerShell command whose content cannot be read) |
  | `windows-shell` | Handing the command to cmd or PowerShell (the commands inside are not checked one by one) |
  | `windows-system` | Changing Windows system settings (registry, scheduled tasks, services, permissions, environment variables, firewall) |
  | `keychain` | Accessing the keychain or Credential Manager (`security`, `cmdkey`, `vaultcmd`) |
  | `system-dirs` | Writing to system folders |

- **Secret files** (reading, writing and searching are all blocked, though sandboxed commands can still change them indirectly: see cerberus.md; answering yes in the conversation does not help, only the config file can change it): `.env`, `.env.*`, `*.pem`, `*.key`, `*.p12`, `*.pfx`, `*.keystore`, `*.jks`, `*.p8`, `*.ppk` (PuTTY),
  SSH keys (`id_rsa`, `id_ed25519`, `id_ecdsa`, `id_dsa`, `id_ed25519_sk`, `id_ecdsa_sk` and their `.pub`), `credentials`, `credentials.json`, `credentials.toml`, `.npmrc`, `.netrc`, `.pypirc`, `.git-credentials`, `.envrc`,
  Android/Flutter's `local.properties`, `key.properties`, `keystore.properties`, `secrets.properties`, `adbkey`, and in your home folder `~/.gradle/gradle.properties`, `~/.docker/config.json`, `~/.kube/config`, `~/.config/gh/hosts.yml`, `~/.m2/settings.xml`, and the `npmrc` files inside nvm and npm,
  plus the key files named in `keyFiles`. Not secret: `.env.example`, `.env.sample`, `.env.template`, `~/.android/debug.keystore`. To add: `permissions.secretFiles` (for example `"*.secret"`); to remove: `permissions.secretFilesRemove` (for example `".npmrc"`).

## Tools that cannot run in the sandbox (Flutter)

Some tools simply cannot run inside the sandbox: `flutter` and `dart` (and `fvm`) write into the Flutter SDK's own folder (`bin/cache`) on every run; the sandbox does not allow that, so they hang.
Opening that folder is not an option either: the `dart.exe` and friends in there are later run outside the sandbox with your permissions, so programs in the sandbox could change them (tested on Windows 2026-10-27).

So (the way Claude Code does it): when the model calls one of these with the sandbox on, xj does not let it hang; it tells the model why, the model asks to **run that one command outside the sandbox**,
and xj asks you once (in red: "runs outside the sandbox with your permissions", and which program actually runs). It runs only if you answer yes. No config change, no turning the sandbox off; other commands stay in the sandbox.

- It asks every time, also in full-auto mode, and there is no "allow permanently"; like `excludedCommands`, only simple forms are allowed (it must run on its own, not chained after other commands).
- Only tools on this list can ask to run outside the sandbox (currently flutter, dart, fvm, and installing, below); any other command asking for it is refused. Similar tools will be added to the same list.

**Installing tools**: when a tool is missing on this computer, the model offers to install it the same way: you are asked every time, and it runs outside the sandbox only if you answer yes.
- Only `winget install --scope user …` and `scoop install …` on Windows (plus lookups such as their search), and `brew install …` on macOS. Other installers (choco, `npm -g`, pip…) are refused.
- `winget install` must use `--scope user` (installs under your account, no administrator rights).
- **No administrator rights by default**: if they are needed, the model gives you the full command to run in an administrator terminal yourself. Only when you have explicitly said in the conversation that it may run it directly does the model ask to run it as administrator
  (`winget install` only): xj still asks you (in red), and Windows shows a UAC prompt too; this also applies in bypass mode.
- **Restart xj** afterwards: PATH and what the sandbox can read are worked out at start, so a newly installed command is only found after a restart.
- What winget installs under your account (`--scope user`) can be read in the sandbox: with `sandbox.toolchainReadable` left at its default, xj opens `%LOCALAPPDATA%\Microsoft\WinGet\Packages` and `Links` to the sandbox for reading (as it does for Scoop; if you write that field yourself, list them in it).
  **What is installed for the whole computer as administrator cannot be used inside the sandbox**: winget gives those files no access for ordinary users, and the sandbox runs as another account (tested 2026-10-28). It works outside the sandbox; to use it inside, install with `--scope user`.
- If you do not want this option: set `"sandbox": { "allowUnsandboxedCommands": false }` in the config, and these commands are always refused; to use them, restart with `xj --no-sandbox` or list them in `excludedCommands`.
- Commands you listed in `excludedCommands` yourself work as before (asked, run outside the sandbox) without the model asking.

## Docker: commands that run outside the sandbox

The sandbox blocks the connection to Docker (`docker.sock`: whoever can tell Docker what to do can mount the whole computer's files into a container), so `docker` always fails inside the sandbox.
To let it use Docker without turning the whole sandbox off, list the command in `sandbox.excludedCommands`:

```json
{ "sandbox": { "excludedCommands": ["docker"] } }
```

- `"docker"` covers all of docker; `"docker compose"` covers only that subcommand. Write the command name, not a path.
- Listed commands **ask every time**, marked in red with "runs outside the sandbox with your permissions" and which program actually runs; nothing runs until you say yes. Full-auto mode asks too, and there is no allow-all.
- Only simple forms are allowed: you can chain several listed commands with `&&` and add `2>&1` or `>/dev/null`; you cannot pipe into other commands, write to files,
  use `$( )`, or put environment variables in front (`PATH=` and `LD_PRELOAD=` would swap the program that actually runs). Forms that are not allowed are refused directly, with a note on how to rewrite them.
- The program is looked up on the PATH outside the sandbox: files of the same name in the project or temp folders (which commands in the sandbox could put there) do not count.
- Nothing runs outside the sandbox in untrusted folders or in read-only and plan modes. Secret files are still always blocked.
- Docker acts on the project's `Dockerfile` and `docker-compose.yml`, which commands in the sandbox can change: look at `git diff` before saying yes.
- To reach a remote Docker (`DOCKER_HOST`), add the variable to `env.passthrough`.

Skip-all-permissions mode (`--dangerously-skip-permissions`) cannot start as root by default (the same goes for a terminal opened with "Run as administrator" on Windows); if you are in a throwaway container or VM and must use root (administrator),
set the environment variable `XING_JANUS_ALLOW_ROOT_BYPASS=1` (only `1` counts). Like the other `XING_JANUS_` variables, it is not passed to the commands xj runs.

Keys are not written in the config: put them in the keychain (`xing-janus-<provider>`), the environment variable `XING_JANUS_<PROVIDER>_API_KEY`, or a file named in `keyFiles`. Do not use generic names such as `ANTHROPIC_API_KEY` or `OPENAI_API_KEY`: other tools read them too.

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

## Connecting other model services

Any OpenAI-compatible API works by adding a `providers` entry, with no code changes. Put the key in the environment variable
`XING_JANUS_<NAME IN CAPITALS>_API_KEY`, the keychain or Credential Manager (the item is named `xing-janus-<name>`), or a file named in `keyFiles`.
`xj setup` only asks about Anthropic, OpenAI and Ollama: set your own service's key right inside xj (paste it when you pick or add the service in `/model`),
or store it yourself (macOS: `security add-generic-password -s xing-janus-<name> -a "$USER" -w`; Windows: `cmdkey /generic:xing-janus-<name> /user:xj /pass`).
You can also add one inside xj: the last item of the `/model` menu, "Add an OpenAI-compatible service", or a server "found on this computer"
("Switching platforms and models" in usage.md). It is written to `providers` here (only `kind` and `baseURL`) once its model list answers;
a key set there is stored the same way as with `xj setup`.

```json
{
  "providers": {
    "openrouter": { "kind": "chat-completions", "baseURL": "https://openrouter.ai/api/v1" },
    "gemini": { "kind": "chat-completions", "baseURL": "https://generativelanguage.googleapis.com/v1beta/openai" },
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
2. **Costs show as "unknown".** Costs come from `~/.xing-janus/prices.json`, so add an entry for your own service
   (for local models, 0 everywhere; otherwise once 10 requests in one turn have no known price, it stops and asks whether to continue).
3. **Scale local models down**: `"toolset": "reduced"` (only read, search and edit).
   Ollama gets this automatically (small models often cannot call tools correctly), but "your own service" (LM Studio, llama.cpp, vLLM…) is treated as a cloud API, so write it yourself.

Local services (llama.cpp's `llama-server`, vLLM, LM Studio) just need `http://localhost:<port>/v1`,
without a key; `http://` only escapes the warning for localhost.
Without `--alias`, llama.cpp's `llama-server` names the model after the file given to `-m` (for example `C:\models\qwen2.5-7b-instruct-q4_k_m.gguf`):
that still works and is remembered, but for a shorter name start it with `--alias qwen` and pick it with `/model llamacpp:qwen`
(`llamacpp` is the name `/model` gives it when it finds the server; if you added it yourself, use your name).

# The config file

Location: `~/.xing-janus/config.json`. The full format is in [`config.schema.json`](../config.schema.json), and `xj doctor` checks against it. Fields you leave out use the defaults.
`xj setup` writes `"$schema"` at the top of the file: open it in an editor such as VS Code to get completion, a description for every field, and typos marked as you type.
Choosing "always allow" at a question writes to `permissions.allow`, `sandbox.allowedDomains` or an MCP server's `autoApprove` (only that field changes, the whole file is checked before writing, and nothing is written if the file itself has errors); see and remove them with `/permissions`.

| Field | Default | Description |
|---|---|---|
| `defaultModel` | `anthropic:claude-opus-5-5` | The default provider and model. A series name (`anthropic:opus`) always uses the latest one |
| `effort` | `{ "default": "high" }` | The default reasoning effort per model; keys are model IDs, series names or `default` |
| `defaultMode` | `normal` | `normal`, `auto-edit`, `full-auto` (the same as always adding `--auto`, see usage.md; `xj -p` ignores it and needs an explicit `--auto`), `plan`, `read-only`. Skip-all-permissions mode cannot be the default: it needs `--dangerously-skip-permissions` every time |
| `uiLanguage` | Your system's language | The interface language: `en` or `zh-TW`. Asked on the first start; `/language ui` changes it |
| `language` | Follows the interface language (`English` for the English interface) | The language AI replies in, written as you would say it (for example `English`, `Deutsch`) |
| `notify` | `false` | Beep when a turn finishes or xj needs your answer |
| `screen` | `fullscreen` | The screen: `fullscreen` is full-screen mode (in wide windows the to-do list and current status are on the right; PageUp/PageDown and the wheel scroll); `classic` is classic mode (the terminal scrolls, searches and copies). When full-screen mode cannot start (output is not a terminal, the old Windows console), classic mode is used. `/tui` writes it when you switch |
| `enterToApprove` | `true` | Enter means yes at questions (on by default, as in Claude Code; set it to `false` to always press y) ("allow once" for domain questions). Dangerous commands, content you have not seen in full and the always-allow confirmation still need `y`; an Enter within 0.5 seconds of a question appearing does not count (you may have been typing). The trust-this-folder question cannot be answered with Enter |
| `statusBar.fields` | `model`, `effort`, `mode`, `sandbox`, `context`, `todo`, `day-cost`, `month-cost`, `progress` | Which items the status bar shows, in order of importance: when the window is too narrow, items are dropped from the end. There is also `session-cost` (the cost of this session). `todo` is the to-do list (it takes no space when there is no list or everything is done; added 2026-10-22, so if you wrote this field yourself before that, add `todo` to see it) |
| `budget` | $5 per turn, $30 per day, $100 per month, billing cycle starts on the 1st | A notice at 80%, a stop-and-ask at 100%. The limits are compared against the **estimated** cost (local price table × usage), not the provider's bill; 0 turns that one off. `xj setup` does not ask about this |
| `limits` | 50 tool calls per turn, 64000 output tokens | Stops and asks at the limit |
| `permissions` | Empty | Additions to and removals from the whitelist (`allow`), the danger list (`danger`) and secret files (`secretFiles`); extra readable paths (`extraReadable`: your home folder itself, credential folders such as `~/.ssh` and `~/.aws`, and the folders containing them are not used; xj says so at startup) |
| `sandbox` | See the schema | Readable toolchains, writable caches, allowed domains, local connections; `gitconfig`: which global config git reads in the sandbox — `"filtered"` (default) keeps only harmless settings such as `user.name` and `user.email` (your aliases and `insteadOf` do not apply in the sandbox), `"full"` is the whole `~/.gitconfig` (commands in the sandbox can then read any URLs and tokens in it); `excludedCommands`: commands that run outside the sandbox (empty by default), see "Docker" below. `autoFixWritable` (Windows): when the sandbox start finds places on your PATH the sandbox account can write to, block them as administrator without asking (UAC still asks; default `false`: xj asks you once, and "always allow" writes `true`; remove it with `/permissions`), see "Skip-all-permissions mode" in usage.md. `writablePrograms` (Windows): what to do when programs that run outside the sandbox (xj's git, Git Bash, `excludedCommands`, tools such as flutter, MCP servers, the rg used by search) are in folders the sandbox can write to — `"ask"` (default) asks once when the sandbox starts, for that conversation; `"deny"` does not ask and does not run them outside the sandbox (search runs inside the sandbox instead); `"allow"` does not ask and runs them (still marked red every time). "Always allow" writes `"allow"`; remove it with `/permissions`; see cerberus.md. Windows (2026-10-21): opening a folder means changing the permissions of every file in it, so with the default `toolchainReadable` and `writable`, folders are opened by project type (npm's global folder only with a `package.json`, the Android SDK and `~/.gradle` only with Gradle files such as `build.gradle`…; Flutter projects (`pubspec.yaml`) do not open the Flutter SDK or the Pub cache, and their `android/` folder does not count either — flutter and dart always run outside the sandbox, so opening them would only slow down the start), and large toolchains only partly; if you write these two fields yourself, they are used as written and xj adds nothing |
| `windows` | `{}` | Windows only (2026-10-20): `gitBash` is the full path of Git Bash's `bash.exe`, `powershell` the full path of `pwsh.exe` or `powershell.exe`. Without them xj looks for them (Git Bash via git.exe on PATH and the default install locations; PowerShell 7 first, then the built-in one). They cannot be files in the project, nor `C:\Windows\System32\bash.exe` (that is WSL) |
| `env.passthrough` | `JAVA_HOME`, `ANDROID_HOME` | Extra variables kept in the clean environment commands get |
| `keyFiles` | Empty | Without a keychain: provider → key file. Paths must be absolute or start with `~` (on Windows with a drive letter, for example `C:\keys\anthropic.txt`): a relative path would be read from the working folder, so the config rejects it. These paths automatically count as secret files |
| `providers` | Built in: `openai`, `ollama` | Change `baseURL`, turn one off with `enabled: false`, or add your own service (`kind: "chat-completions"`, `baseURL`, `keyName`) |
| `capabilities` | Empty | Override a model's capabilities (context, maximum output, toolset…); keys are "provider:model". `todoTool`: whether this model gets the to-do list tool (`todo_write`); `webFetch`: whether it gets the web-fetch tool (`web_fetch`). Every model gets them by default; write `false` to keep a model from using one. `subagents`: whether it gets the subagent tool (`agent`); also on by default, write `false` to turn it off for a model |
| `subagents` | 4 at once, 30 tool calls, 10 minutes | Subagents (see "Subagents" in usage.md): `maxConcurrent` how many run at once (only read-only explore subagents run in parallel), `maxToolCalls` and `timeoutMinutes` the limits for each (at the limit it is told to report), `model` the default model (without it, the main conversation's) |
| `mcp.servers` | Empty | MCP servers: `command`, `args`, `env`, `description`, `sandbox`, `autoApprove`, `readOnly`. Values in `env` whose names look like keys (ending in `_KEY`, `_TOKEN`, `_SECRET`, `_PASSWORD` and so on) are treated as keys: masked wherever they appear in the server's output, error messages and session logs. The Windows sandbox cannot run MCP servers yet (stdin cannot be connected): with the sandbox on, set `sandbox: false` (it then runs outside the sandbox with your permissions) |
| `anthropic` | `fallback: default`, `prefixMismatch: drop_block` | Anthropic-specific settings |
| `evals.budgetUsd` | `2` | The most each eval run may spend |
| `sessions.retentionDays` | `30` | How many days session logs are kept |

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
- What winget installs under your account (`--scope user`) can be read in the sandbox: xj always opens `%LOCALAPPDATA%\Microsoft\WinGet\Packages` and `Links` to the sandbox for reading (as it does for Scoop).
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

Skip-all-permissions mode (`--dangerously-skip-permissions`) cannot start as root by default; if you are in a throwaway container and must use root,
set the environment variable `XING_JANUS_ALLOW_ROOT_BYPASS=1` (only `1` counts). Like the other `XING_JANUS_` variables, it is not passed to the commands xj runs.

Keys are not written in the config: put them in the keychain (`xing-janus-<provider>`), the environment variable `XING_JANUS_<PROVIDER>_API_KEY`, or a file named in `keyFiles`. Do not use generic names such as `ANTHROPIC_API_KEY` or `OPENAI_API_KEY`: other tools read them too.

## Connecting other model services

Any OpenAI-compatible API works by adding a `providers` entry, with no code changes. Put the key in the environment variable
`XING_JANUS_<NAME IN CAPITALS>_API_KEY`, or store it in the keychain with `xj setup` (the item is named `xing-janus-<name>`).

```json
{
  "providers": {
    "openrouter": { "kind": "chat-completions", "baseURL": "https://openrouter.ai/api/v1" },
    "gemini": { "kind": "chat-completions", "baseURL": "https://generativelanguage.googleapis.com/v1beta/openai" },
    "local": { "kind": "chat-completions", "baseURL": "http://localhost:8080/v1" }
  },
  "capabilities": {
    "openrouter:anthropic/claude-sonnet-4.5": { "contextTokens": 200000, "maxOutputTokens": 32000 },
    "local:your-model-id": { "contextTokens": 65536, "maxOutputTokens": 8192, "tools": "unreliable", "toolset": "reduced", "parallelToolCalls": false }
  },
  "defaultModel": "openrouter:anthropic/claude-sonnet-4.5"
}
```

Three things to watch:

1. **Fill in the capabilities yourself.** When a model's capabilities cannot be found, conservative defaults are used (**32k context**, 8k output). So a model with a 1M context
   is treated as 32k unless you write `capabilities`. Model IDs with slashes are fine.
2. **Costs show as "unknown".** Costs come from `~/.xing-janus/prices.json`, so add an entry for your own service
   (for local models, 0 everywhere; otherwise each turn asks "some requests have no known price; continue?").
3. **Scale local models down**: `"tools": "unreliable"`, `"toolset": "reduced"`, `"parallelToolCalls": false`.
   Ollama gets this automatically (small models often cannot call tools correctly), but "your own service" is treated as a cloud API.

Local services (llama.cpp's `llama-server`, vLLM, LM Studio) just need `http://localhost:<port>/v1`,
without a key; `http://` only escapes the warning for localhost.

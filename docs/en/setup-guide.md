# What to set: by situation

The default is **the sandbox on and normal mode**: edits show a diff and ask, whitelisted commands run directly, and the rest ask. Most people do not need to change anything.
For what each setting decides and how each mode behaves with the sandbox on and off, see "Sandbox and modes" in [usage.md](usage.md).

Each situation below gives the lines to add to the config (`~/.xing-janus/config.json`) and what the status bar shows. Write only the fields you change; the rest keep their defaults. Restart xj after changing the config.
The common ones (sandbox at start, default mode, allow full-auto…) can also be changed inside xj with `/config`, or just ask xj what to set (see "Changing settings" in usage.md).

| Your situation | Sandbox | Mode |
|---|---|---|
| Everyday development | On | Normal (or auto-edit) |
| You trust the code and want fewer questions | On | Full-auto |
| Let it run for a while without stopping for you | On | Skip all permissions |
| You need docker or flutter | On: list docker in `excludedCommands`; flutter runs outside the sandbox by itself (asking each time) | As above |
| A throwaway container or virtual machine | Off (if the sandbox cannot start there, write `sandbox.enabled: false` in that machine's config) | Skip all permissions |

## Everyday development

Nothing to change. Status bar: "Normal mode · sandbox on".

- The first time you open a folder, xj asks whether to trust it: say yes for your own projects (only then are the whitelist, checks and AGENTS.md used), and no for other people's repos at first (only read-only and plan modes; change it later with `/trust`).
- For commands you run often (`make lint` and the like), press `p` (always allow) when asked to add them to the whitelist, and they run directly from then on; see and remove them with `/permissions`.
- If you do not want to be asked about every edit: switch to auto-edit (Shift-Tab, or `"defaultMode": "auto-edit"` in the config). What changed is still shown, and `/undo` can restore it.

## Docker and other things that cannot run in the sandbox

No need to turn the whole sandbox off: list those commands in `excludedCommands`, and only they run outside the sandbox, asking every time.

```json
{ "sandbox": { "excludedCommands": ["docker"] } }
```

- flutter, dart, fvm and installers (winget, scoop, brew) already work this way (asking each time they need to run outside the sandbox); no need to list them.
- Details: "Docker: commands that run outside the sandbox" and "Tools that cannot run in the sandbox (Flutter)" in [config.md](config.md).

## Reading or editing another folder

You open xj in A and also need to read or edit a related folder B: type `/add-dir ../B` in xj (it asks whether B is read-only or also editable; editable folders must be trusted).
For folders to add every time you open xj in A, write them in the config:

```json
{ "permissions": { "addDirs": { "~/work/A": [{ "path": "~/work/B", "write": true }, { "path": "~/work/docs" }] } } }
```

Opening xj in A then adds B (editable) and docs (read-only), and the status bar gains "folders 3" (counting the working folder). For just this once, start with `xj --add-dir ../B` (editable).
See "Added folders" in [usage.md](usage.md) and "Folders added at startup" in [config.md](config.md). Do not open xj in the parent folder (the one with all your projects): that puts every project in scope.

## Fewer questions

Full-auto: neither edits nor commands in the sandbox ask, and the sandbox is the safety boundary (commands only stop asking while the sandbox is on). Enable it in the config first, then switch to it with Shift-Tab:

```json
{ "allowFullAuto": true }
```

- For just this run, start with `xj --auto`. It only works in trusted folders.
- Still asks: dangerous commands, new domains, commands that run outside the sandbox, MCP tools, PowerShell commands outside its whitelist (Windows).
- **Commit with git before you start**: commands in the sandbox can still change the working folder, and `/undo` cannot restore what commands changed.
- Status bar: "Full-auto mode · sandbox on".

## Let it run without stopping to ask

Skip all permissions with the sandbox on: nothing asks, but commands stay confined in the sandbox, and domains not allowed before are blocked without asking.

```json
{ "allowBypass": true, "sandbox": { "allowedDomains": ["registry.npmjs.org"] } }
```

- Shift-Tab can then switch to it; the first time in each run of xj you type "I understand the risks". For just this run, start with `xj --dangerously-skip-permissions`.
- List the domains it needs in `allowedDomains` first: in this mode, domains not allowed before are blocked without asking.
- Commands in `excludedCommands` (docker and the like) run outside the sandbox without asking; the built-in flutter, dart, fvm and installers are blocked without asking:
  to use flutter and the like, list them in `excludedCommands`; for installers, switch to a mode that asks, or install the tool yourself in a terminal.
- Still asks: deleting an important place (the root, your home folder, the working folder or a folder above it…), switching to root, your own entries in `permissions.danger`.
- **Commit with git before you start.** The budget limits still apply (`/budget`).
- Status bar: "Skip-all-permissions mode · sandbox on".

## In a throwaway container or virtual machine

Skip all permissions without a sandbox: nothing asks, and commands run with your permissions. **Use it only in an environment you can throw away.**

```json
{ "allowBypass": true, "defaultMode": "bypass", "sandbox": { "enabled": false } }
```

- `defaultMode: "bypass"` enters skip-all-permissions mode as xj starts (you still type "I understand the risks" first; cancelling starts in normal mode); without it, switch with Shift-Tab.
- In containers the sandbox often cannot start (and then every command is refused): write `sandbox.enabled: false` in that machine's config, or start with `--no-sandbox`.
- Skip all permissions does not start as root; if you must, set the environment variable `XING_JANUS_ALLOW_ROOT_BYPASS=1`.
- Secret files and protected files are only blocked when a command names them directly (indirect reads such as `grep -r` are not): do not keep secrets in this folder.
- The whole status bar is red (both at their loosest).

## Windows without the sandbox installed

It still works: every command asks first, and runs outside the sandbox with your permissions only after you say yes (the whitelist does not apply). Status bar: "sandbox not installed".
To install it, see "Installing on Windows" in [install.md](install.md); `xj doctor` says whether it is installed and what is missing.

## The sandbox cannot start on Linux

When the sandbox cannot start, every command is refused (status bar: "sandbox failed"). Run `xj doctor` for the reason first; common ones are in [troubleshooting.md](troubleshooting.md).
To use xj for now anyway: type `/sandbox off` in xj, or start with `xj --no-sandbox` (every command asks).

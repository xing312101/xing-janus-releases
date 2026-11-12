# Where your data is, where it goes, how to delete it

## Where it goes

- What you type, the file contents it reads and the output of commands are sent to the model service you chose (Anthropic, OpenAI, Gemini, a local Ollama, or another OpenAI-compatible service you connected), under that service's terms.
  With a free-tier Gemini key, Google uses what you send to improve its products ("Gemini" in config.md).
- Xing Janus itself collects nothing: no telemetry, no crash reports, no automatic update checks. Its only outside connections are the model APIs, the domains you allow, and GitHub when you run `xj update` (it only looks up the version and downloads the release files; it sends no data).
- Fetching web pages (`web_fetch`): the website sees the full URL, your IP address and the User-Agent `xing-janus/<version>`; no cookies and no login details are sent.
  What it fetches is treated like a file it read: it is sent to the model and saved in the session log. URLs are not sent anywhere else to be checked.
- Switching platforms with `/model`: the conversation so far (including file contents it read and command output) is handed over as text to that platform, and everything after it goes there too. If the conversation has content, xj asks first and says which machine it goes to (it does not ask for a platform on this computer: the conversation stays on it).
- Subagents: they use the same provider as the main conversation; if you name another provider's model, the instructions and whatever it reads go to that provider.
- Asking xj how to set it up (the built-in skill `xj-settings`): when the model reads this skill, xj adds your current settings, which go to the model service like a file it read:
  the config as written (with MCP servers' environment variable values and arguments, and user names, passwords and query strings in service URLs, taken out), and this conversation's state (working folder, mode, sandbox, added folders, budget, how many background jobs are running, the platform and the xj version).
  `xj mcp` does not offer this skill. If the model proposes setting changes (`settings_propose`), xj writes them to `config.json` only if you approve.
- When other programs call xj through `xj mcp`: replies, what it did, and the questions that need your approval (the full command or diff) are handed to the program calling it (for example Claude Desktop or a chat-app bridge you wrote),
  and where they go next is up to that program (a bridge sends them to the chat app). Keys in known formats are masked before they are handed over ([mcp-server.md](mcp-server.md)).
- If a model service answers with a redirect (3xx), xj does not follow it: neither the key nor the conversation goes along to another place. Keys are not sent over unencrypted http to another machine either (unless you set `allowPlainHttpKey`; "Connecting other model services" in config.md).
- Before anything is sent, keys in known formats (`sk-ant-`, `sk-proj-`, `AIza`, `AQ.`, `ghp_`, `AKIA`…) are masked. That is a last line of defense; secret files cannot be read in the first place.
  Multi-line private keys (PEM, OpenSSH, PGP) are masked completely, also when they sit inside an ordinary file: when a file is read each line becomes `[REDACTED:private-key]` and the line numbers stay the same; in search results a single line that looks like the body of a key becomes `[REDACTED:possible-private-key]`.
  This goes by format, so what is not recognized is still sent: private keys in other formats (for example PuTTY's `.ppk`), a command that prints only a middle part of a key, a key written in source code as one string per line,
  output with a marker at the end of every line (`cat -A`). Keep private keys in files with a secret-file name, or add them to `permissions.secretFiles`.

## Where it is stored

Everything is in `~/.xing-janus/` (or in the folder the environment variable `XING_JANUS_HOME` names, if set; folder permissions 700, file permissions 600):

| File | Contents | Kept |
|---|---|---|
| `config.json`, `prices.json`, `checks.json` | Settings, price table, checks after edits | Always |
| `trusted.json`, `accepted.json` | Trusted folders, the notice you accepted | Always |
| `spend.jsonl` | The cost of each request (no content) | Always |
| `keys/` | Key files: keys that `xj setup` or `/model` could not store in the keychain or Credential Manager (Linux and WSL have no keychain) | Always |
| `skills/` | Your own skills | Always |
| `models-*.json` | The cached model list of each provider (of each key, when a provider has several) | Always (looked up again after a day) |
| `acl-fix.json` | Windows: which places on PATH the sandbox account can write to, and what to block (see "Skip-all-permissions mode" in usage.md) | Updated at every sandbox start |
| `sessions/` | Session logs (keys masked), the full run of each subagent (`subagents/`, keys masked), snapshots before edits (unchanged, for `/undo`), full outputs (`outputs/`: output too long to show in full, and background job output in `job-<number>.log`, keys masked), images you attached, `--debug` logs | 30 days (`sessions.retentionDays`) |

Session logs contain the file contents it read, so do not put `~/.xing-janus/` in a folder that syncs to the cloud (`xj doctor` checks this).

Next to it, `~/.xing-janus-cache/` (with `XING_JANUS_HOME` set, that path plus `-cache`) is for commands in the sandbox: the npm cache and the git config used in the sandbox. The sandbox can write there, so no keys or logs are kept in it.
Windows also has `%LOCALAPPDATA%\xing-janus\`: a record of the permissions added to parent folders, and to the folders that hold protected files, when the sandbox starts (used to take them back; see "Windows" in troubleshooting.md).

## How to delete it

- Old session logs: `xj sessions prune`
- Everything: `xj reset` (lists what will go and asks first; `~/.xing-janus-cache/` goes too)
- The key in the keychain: `security delete-generic-password -s xing-janus-<provider>`
- The key in Windows Credential Manager: `cmdkey /delete:xing-janus-<provider>`
- With several keys for one provider, the named ones are separate items (`xing-janus-<provider>--<name>`): the two lines above do not delete them, so delete each one by its name

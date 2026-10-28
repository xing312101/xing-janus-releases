# Where your data is, where it goes, how to delete it

## Where it goes

- What you type, the file contents it reads and the output of commands are sent to the model service you chose (Anthropic, OpenAI, Gemini, or a local Ollama), under that service's terms.
- Xing Janus itself collects nothing: no telemetry, no crash reports, no automatic update checks. Its only outside connections are the model APIs and the domains you allow.
- Fetching web pages (`web_fetch`): the website sees the full URL, your IP address and the User-Agent `xing-janus/<version>`; no cookies and no login details are sent.
  What it fetches is treated like a file it read: it is sent to the model and saved in the session log. URLs are not sent anywhere else to be checked.
- Subagents: they use the same provider as the main conversation; if you name another provider's model, the instructions and whatever it reads go to that provider.
- Before anything is sent, keys in known formats (`sk-ant-`, `sk-proj-`, `AIza`, `ghp_`, `AKIA`…) are masked. That is a last line of defense; secret files cannot be read in the first place.

## Where it is stored

Everything is in `~/.xing-janus/` (folder permissions 700, file permissions 600):

| File | Contents | Kept |
|---|---|---|
| `config.json`, `prices.json`, `checks.json` | Settings, price table, checks after edits | Always |
| `trusted.json`, `accepted.json` | Trusted folders, the notice you accepted | Always |
| `spend.jsonl` | The cost of each request (no content) | Always |
| `sessions/` | Session logs (keys masked), the full run of each subagent (`subagents/`, keys masked), snapshots before edits (unchanged, for `/undo`), full outputs, images you attached, `--debug` logs | 30 days (`sessions.retentionDays`) |

Session logs contain the file contents it read, so do not put `~/.xing-janus/` in a folder that syncs to the cloud (`xj doctor` checks this).

## How to delete it

- Old session logs: `xj sessions prune`
- Everything: `xj reset` (lists what will go and asks first)
- The key in the keychain: `security delete-generic-password -s xing-janus-<provider>`
- The key in Windows Credential Manager: `cmdkey /delete:xing-janus-<provider>`

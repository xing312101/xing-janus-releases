# Letting other programs call xj through MCP (xj mcp)

`xj mcp` lets other programs use Xing Janus as a tool: Claude Desktop, IDEs, other agents, or a chat-app bridge you write yourself (Telegram, Discord and the like).
It is an MCP server over stdio: the MCP client starts it, and it opens no ports for others to connect to. Its network use is the same as xj in a terminal: the model's API, the web tool, and the local proxy the sandbox gives the commands inside it.

xj itself does not connect to chat apps. To ask xj for something from your phone, write (or find) a "bridge" program: it keeps the bot token, receives your messages and calls xj as an MCP client;
when xj needs your approval it asks through MCP elicitation, and the bridge turns the question into buttons and passes your answer back. The second half of this page is the interface for people writing bridges.

## Before you start

- Run `xj` once in a terminal: accept the notice and set up a key.
- Open each folder you want xj to work in once with `xj` at the computer and choose to trust it. **A new folder cannot be trusted through MCP**.
- The sandbox must be on and able to block everything (the "OS sandbox" row of `xj doctor`; on Windows, a sandbox that "cannot block everything" does not count). Without it `xj_task` is always refused; `--no-sandbox`, bypass mode and `--auto` cannot be used here.
- After changing `mcpServer` in the config file, restart the MCP client: `xj mcp` reads its settings only when it starts, and once they have changed it takes no new tasks and asks you to restart (whether you relaxed or tightened them).

## Connecting a client

Claude Desktop (`claude_desktop_config.json`), Claude Code (`.mcp.json` in a project) and similar clients use the same form:

```json
{
  "mcpServers": {
    "xing-janus": { "command": "xj", "args": ["mcp"] }
  }
}
```

On Windows, use `xj.cmd` as the `command` (or the full path `C:\Users\<you>\AppData\Local\Programs\xj\xj.cmd`).
The folder the client starts `xj mcp` in is the default project; for other projects see `mcpServer.projects` below.
**Claude Desktop ignores `cwd` in its config and starts `xj mcp` in a system folder such as `C:\Windows\System32`** (tested in November 2026): that folder is not trusted, so `xj_task` refuses.
With Desktop, always list projects in `mcpServer.projects` and ask it to pass `project` to `xj_task` (for example "use xj_task with project web to …").

## Settings (`mcpServer` in the config file)

You decide what xj may do through MCP. Anything you leave out takes the safe side:

| Field | Default | Meaning |
|---|---|---|
| `mode` | `read-only` | The mode used through MCP: `read-only`, `plan`, `normal`, `auto-edit`, `full-auto` (the same modes as in a conversation). `bypass` is never allowed |
| `outsideSandbox` | `never` | Actions outside the sandbox (`excludedCommands`, tools that cannot run in the sandbox, installing tools, administrator rights, MCP servers with `sandbox: false` in the config): `never` refuses them (do them at the computer); `ask` asks through elicitation (even for those servers' `autoApprove` tools) |
| `projects` | `{}` | Name → folder (a full path or starting with `~`); the client picks one by name. Without it, only the folder `xj mcp` was started in |
| `anyProject` | `false` | The client may pass a folder path directly (it still has to be a trusted folder; folders inside a trusted folder count too, including ones created later) |
| `budget` | $2 a turn, $10 a day | Spending limits through MCP: `perTurnUsd`, `perDayUsd` (US dollars, more than 0). They count together with `budget`, and the lower one applies; the daily figure is all spending (including the terminal). At the limit xj stops, without asking whether to go on |

For example, to let your own bridge write code in two projects:

```json
{
  "mcpServer": {
    "mode": "normal",
    "projects": { "web": "~/code/web", "api": "~/code/api" },
    "budget": { "perDayUsd": 10 }
  }
}
```

**Before you relax anything**:

- `mode` at `normal` or above: the program calling xj can change your project and run commands in the sandbox. "The program" also means whoever is behind it and the messages it receives
  (for example a message someone else sent to your bridge, or a web page another agent read). With `normal`, edits and commands outside the allowlist are still asked, but the person pressing the button is not at the computer;
  `auto-edit` does not ask about edits; `full-auto` asks about neither edits nor commands in the sandbox (only dangerous ones, new domains and actions outside the sandbox are asked).
- `outsideSandbox` set to `ask`: outside the sandbox means your permissions. If the approver's account or phone is in someone else's hands, the button stops nothing.
- `anyProject`: the client can pick any folder you have trusted, including folders inside it.
- Every relaxed setting is printed to stderr each time `xj mcp` starts (visible in the client's logs), and `xj doctor` mentions it too.

## Tools (for plugin authors: changes will stay compatible)

xj's tool descriptions, results and errors are always in English (for the client's model); elicitation questions follow your interface language (for a person).

### `xj_task`

Give it one task and wait for the turn to finish.

| Argument | Type | Meaning |
|---|---|---|
| `prompt` | string (required) | What to do (at most 100,000 characters). `@path` mentions and file paths in it do not attach files (that only works when typing at the computer): ask it to read the file instead |
| `project` | string | A name from `mcpServer.projects`; with `anyProject` on it can also be a path. Omit it for the folder `xj mcp` was started in |
| `conversation` | `"continue"` (default) or `"new"` | Continue the previous conversation in that project, or start a new one |

Result: `content[0].text` is a summary for people (reply, what it did, what was not done, notices, cost); `structuredContent` is:

```json
{
  "project": "web",
  "reply": "the final reply (what it said before working and its final answer, separated by a blank line)",
  "actions": ["Read package.json", "Run npm test"],
  "blocked": ["Create x.txt (user)"],
  "notices": ["…"],
  "ended": "done",
  "costUsd": 0.0123
}
```

`ended` is `done`, `interrupted`, `stopped`, `error` or `refused`; with `error` and `refused`, `isError` is true. `costUsd` is `"unknown"` when no price is known.
The part in brackets in each `blocked` entry says what stopped it: `user` (you refused), `rules` (the mode or permission rules), `sandbox`, `trust`, `interrupt`, `input`.
When it cannot run at all (a folder that is not trusted, no sandbox, another task running…), the result has `isError: true` and the reason. When `project` is a path, a folder that does not exist and one that is not trusted get the same answer (a client cannot probe which paths exist), and `project` in the result is the path as given.

One task at a time; calling it while a task runs returns "already running". At a spending limit (`mcpServer.budget`, `budget`) it stops (`ended` is `stopped` and `notices` says why) without asking whether to go on. If the request carries `_meta.progressToken`, every tool call and change of the to-do list sends `notifications/progress`
(`message` describes the step): clients can reset their timeout on it.

### `xj_status`

Argument: `project` (as above). Returns the model, mode, sandbox, today's and this month's spending, the to-do list, and whether a task is running (the same fields in `structuredContent`).

### `xj_stop`

Interrupts the running task. A client sending `notifications/cancelled` for the `xj_task` request does the same.

## When approval is needed (elicitation)

If the client declares `capabilities.elicitation` in `initialize` (protocol version `2025-06-18`), xj sends `elicitation/create` when it needs to ask:

- `message`: what it wants to do, why it asks, what to watch for, and the full command or diff. Keys are masked before it is sent (the question passes through the bridge and leaves the computer), control characters are shown in a visible form,
  and line breaks in one-line parts (what, why, the address) become spaces.
- `requestedSchema`: one field, `answer`, a string whose `enum` lists the allowed values and `enumNames` the text for people.
  - Editing a file, running a command: `yes` (allow once), `no` (deny). There is no "always" or "for this conversation": those can only be given at the computer.
  - Connecting to a domain: `once`, `no`. "For this conversation" would add the domain to the sandbox allowlist, so like "always" it can only be given at the computer (put domains you use often in `sandbox.allowedDomains` in the config file).
  - Stopping to ask whether to go on (the tool-call limit and the like): `yes`, `no`. A spending limit is not asked about: xj just stops.
- Answer with `{ "action": "accept", "content": { "answer": "yes" } }`. `decline`, `cancel`, or a value that was not offered all count as a refusal.
- One question at a time: the next is sent after the previous one is answered. No answer in 10 minutes counts as a refusal (xj sends `notifications/cancelled` to withdraw the question); when the task is interrupted, unanswered ones are withdrawn too.
  When no task is running (for example a program left running in the sandbox wants to connect somewhere), xj does not ask and refuses.

If the client does not support elicitation, everything that needs approval is refused (as with `xj -p`). Tested in November 2026: Claude Code supports it (the question appears in its screen, choices shown from `enumNames`);
Claude Desktop does not yet (its `initialize` declares no elicitation), so through it only things that need no approval work (for example the default read-only mode). Claude Code's `-p` (non-interactive) declares elicitation
but nobody can answer, so anything that needs approval is refused. A bridge should show people the whole `message`: attach it as a file if it is long, rather than giving only a summary.

## Protocol details

One JSON-RPC 2.0 message per line on stdio. Protocol versions `2025-06-18`, `2025-03-26` and `2024-11-05` are accepted (only `2025-06-18` has elicitation). Batches (arrays) are handled as JSON-RPC says, with one array in reply;
sending a request with the id of one that is still running gets an error (`-32600`); a request you sent `notifications/cancelled` for gets no result. A line can be at most 8 MB.

## Records

Conversations through MCP are kept in `~/.xing-janus/sessions` like any other: the client's name and version, the project, every question asked and how it was answered (including timeouts and no answer), and every refusal reason are recorded.

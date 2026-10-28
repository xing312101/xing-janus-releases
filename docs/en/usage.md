# Usage

## Modes

| Mode | Edits | Commands |
|---|---|---|
| Normal (default) | Shows the diff, then asks: yes / no / allow all edits this session | Whitelisted ones run directly; others ask |
| Auto-edit | No asking, edits directly (the screen still shows what changed) | Same as above |
| Full-auto (start with `--auto`) | No asking | Nothing that runs in the sandbox asks; commands on the danger list, new domains, PowerShell outside the whitelist and MCP still ask |
| Skip all permissions (start with `--dangerously-skip-permissions`) | No asking | Nothing asks, and **there is no sandbox**; only secret files named directly and protected files stay blocked (indirect reads such as `grep -r` are not) |
| Plan | Not allowed | Only whitelisted read-only commands; it proposes an approach, and after you approve it switches to normal mode and starts |
| Read-only | Not allowed | Only whitelisted read-only commands |

- Dangerous commands (`rm -rf`, `git push`, `sudo`…) always ask, marked in red with the reason. The only exception is skip-all-permissions mode.
- Switch with `/mode` or Shift-Tab. In a folder you do not trust, only read-only and plan modes are available. Each switch adds a line to the conversation, such as "Mode: normal → auto-edit" (yellow when it loosens permissions);
  a Shift-Tab pressed in normal mode while you are typing is recorded when you send that line (otherwise it would mess up the input line).

### Full-auto mode

Start with `xj --auto` (or set `defaultMode` to `"full-auto"` in the config); only then can Shift-Tab reach it. Edits and commands in the sandbox no longer ask first,
and the sandbox is the safety boundary. It only works in a **trusted folder** with **the sandbox on**: with `--no-sandbox`, without the Windows sandbox installed, or when part of the sandbox cannot block
(a Windows project too large to finish scanning for secret files), it cannot be entered; xj starts in normal mode and says why.

Still asks: commands on the danger list, new domains, PowerShell commands outside the whitelist (PowerShell strings can build any path or command), MCP calls, and cases where it cannot tell whether a file is a secret
(for example a file with several names). Still always blocked: secret files, and files that later run outside the sandbox (`.git/hooks`, `.vscode/tasks.json`…). The budget limits still apply.

What the sandbox cannot block is up to you:

- **Things in the working folder**: the sandbox lets commands change the working folder, so deletes and overwrites not on the danger list do not ask, and `/undo` cannot restore files changed by commands. **git commit before you start.**
- **Things you later run outside the sandbox**: source code and `package.json` scripts are its work area. **Look at `git diff` before running build or `npm start` outside the sandbox.**
- **Allowed domains**: programs in the sandbox can send things to domains you already allowed without asking again.
- **Instructions hidden in files**: if it is led astray, anything not on the danger list happens directly. Only use it on code you trust.

Trusting the folder again (`/trust`) does not bring full-auto back by itself; switch again. `xj -p` only uses full-auto with an explicit `--auto` (the config setting does not count).

### Skip-all-permissions mode

Start with `xj --dangerously-skip-permissions`. **There is no sandbox**: edits, commands (including those on the danger list), PowerShell and MCP calls never ask first;
MCP servers also run outside the sandbox. Commands can do anything your account can: delete files, read the SSH keys and cloud credentials in your home folder, connect to any site, use Docker.
Instructions hidden in files and web pages (prompt injection) can tell it to do exactly that. **Only use it in an environment you can throw away (a container or VM), and git commit before you start.**

- **Still always blocked**: secret files named directly (`cat .env`, reading a key file…) and protected files (`.git/hooks`, `.vscode/tasks.json`…). When it cannot tell whether a file is a secret (a file with several names), it still asks.
- **Indirect reads are not blocked**: without a sandbox, xj only sees the text of the command. `grep -r`, `cat *`, `find -exec cat`, `tar`, PowerShell's `Get-ChildItem | Get-Content`,
  or a program the model writes can all read a `.env` in the folder, and what they read goes to the model service (tested 2026-10-21). **Keep secrets out of this folder.**
- **Switching to root still asks**: `sudo`, `su`, `doas`, `pkexec` (also inside `env` or `bash -c`), and PowerShell's `-Verb RunAs`, `runas` and `sudo`.
  To stop `sudo` from asking, add `"sudo"` to `permissions.dangerRemove` in the config (it then is not marked red in normal mode either). This only covers bash's `sudo`: PowerShell's `-Verb RunAs` and `runas` still ask.
- **It cannot start as root**: `sudo xj --dangerously-skip-permissions`, or the default root user in a container, is refused; use a normal account instead.
  The same goes for a terminal opened with "Run as administrator" on Windows (when xj cannot tell, it assumes it is).
  If you are in a throwaway container and must use root, set the environment variable `XING_JANUS_ALLOW_ROOT_BYPASS=1` and start again.
- **Still in effect**: the budget limits and the per-turn tool-call limit.
- **Confirm every time**: at startup it shows a warning and the notice, and you must type "I understand the risks" yourself and press Enter (pasting does not count; Esc cancels). It cannot be made the default in the config.
- **Only in a trusted folder**: otherwise it starts in the usual mode (with the sandbox).
- The whole status bar turns red and says "Skip-all-permissions mode". Shift-Tab can switch to other modes, but this session has no sandbox, so after switching every command asks.
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
  Dangerous commands, content you have not seen in full (press `v` first) and the confirmation for always-allow still need `y`; an Enter within 0.5 seconds of a question appearing does not count.
- **Always allow (`p`)**: written into the config, **for every project**. After you press it, it asks once more, listing the file, the field and what will be written; nothing is written until you press `y` (`n` goes back to the original question).
  Where it is offered: domains in the sandbox (written to `sandbox.allowedDomains`), simple commands (written to the whitelist `permissions.allow` as the command plus its subcommand,
  for example `npm test`: `npm test --watch` then counts too), MCP tools (written to that server's `autoApprove`).
  Where it is not: commands on the danger list, commands that run outside the sandbox (`excludedCommands`), commands with `&&`, pipes or redirection, a bare shell or interpreter (`node`, `python`, `bash`),
  PowerShell commands outside the whitelist, and edits (for those, make auto-edit the default mode). Secret files can still only be changed in the config by hand. Always-allow for commands is only offered with the sandbox on and in a trusted folder (only then does the whitelist run them directly).
  See and remove always-allowed items with `/permissions`.
- A real question has `▍` at the start of its title and of the line that wants your answer, and the label on the left is "xj". Text from the model and file contents cannot print that mark: something that looks like a question without it was written by the model, not asked by Xing Janus.

## Slash commands

`/help` lists them all. The common ones:

| Command | What it does |
|---|---|
| `/model [provider:model]` | Switch models; without an argument, pick from the models this key can use (↑↓ to move, Enter to pick, a digit picks directly, Esc cancels). Switching to another provider mid-conversation hands the conversation over as text |
| `/effort low…max` | Reasoning effort; only the levels the current model supports are listed |
| `/undo` | Restore the files `edit` and `write` changed in the last turn; can be repeated (changes made by `bash` cannot be restored) |
| `/diff` | What `edit` and `write` changed in this session |
| `/todo` | Show the model's to-do list again (how far it got); view only, tell it if you want changes |
| `/compact` | Condense the earlier conversation into a summary (it also happens automatically when the context reaches the threshold) |
| `/clear`, `/resume` | Start a new conversation; resume an earlier one in this folder |
| `/checks` | Checks that run automatically after edits (tests, lint); failures go back to the model, up to 3 times |
| `/image <path>` | Attach an image (you can also drag an image into the terminal, or type `@path` in your message) |
| `/trust`, `/add-dir` | Trust this folder; add a readable folder for this session |
| `/permissions` | Show the permissions in the config (always-allowed whitelist entries, domains, MCP tools); `/permissions remove <number>` removes one, effective in this session too |
| `/cost` | The cost of this turn, this session, today and this month |
| `/tui` | Full-screen mode (default) and classic mode: `/tui fullscreen`, `/tui classic` (saved in the config for next time) |
| `/language` | The interface language (`/language ui en`, `/language ui zh-TW`, saved in the config) and the language of AI replies for this session (`/language reply …`) |

## Typing while it works

While the model is working, the input line stays open (at the very bottom in full-screen mode; on the status-bar line in classic mode):

- Type and press Enter: the line is **queued** (drawn above the input line in full-screen mode) and handed to the model after its current step (before its next tool call); if it has already finished talking, it answers you in the same turn.
  It is printed into the conversation at the moment it is handed over, so the order matches what the model saw.
- Use it to add details, correct something or stop it: "skip item 2", "use TypeScript instead", "do not touch the tests yet".
- Slash commands typed then wait until the turn ends (it never switches models or conversations while the model is halfway through).
- With an empty input line, Ctrl-C interrupts the turn (if there is text, it clears the text first). Lines not yet handed over when you interrupt are not sent automatically; they are listed so you can send them again.
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
- The model cannot pretend to be xj: the first columns of every line (5 in the English interface, 4 in Chinese) are always drawn by Xing Janus, and the model's text starts after them; the `▏` bar, like the `▍` of questions, cannot be printed by the model's text, file names or command output.

## Full-screen mode

Full-screen mode is the default: like vim or htop, Xing Janus draws the whole screen. To use the terminal's own scrolling, search, and select-and-copy, type `/tui classic` to switch to classic mode (saved as `screen` in the config for next time); `/tui fullscreen` switches back:

- When the window is wide enough (100 columns or more), there is a column on the right: the to-do list at the top, and below it the **current status** and common commands. When it is narrower, the to-do list sits above the input line (up to 5 items), and the status is in the status bar at the bottom.
- Each status item has a colored block in front and says what it currently is: mode, sandbox, trusted folder, edits ok (you pressed `a` at a question), Enter=yes, beep, checks, always ok.
  Blocks: **green** on, **gray** off, **yellow** on but looser (asks less), **red** needs attention (no sandbox, skip-all-permissions mode). The words say it all; the colors only help (no colors with `NO_COLOR`).
- **Click** (press and release on the same line): the gray text after each item says what a click does (flip, off, list); for items a click cannot change (sandbox, trusted folder, edits ok when off), the next line says how to change them. Beep and Enter=yes flip on/off (for this session only); mode moves to the next one (like Shift-Tab);
  edits ok is taken back with a click when it is on (a click does not turn it on: press `a` at an edit question); trusted folder and sandbox only say how to change them;
  checks and always ok list their contents; common commands are filled into the input line (read it, then press Enter). After a click, the bottom line says what changed. Mouse codes inside pasted text do not count as clicks.
- Scrolling: PageUp/PageDown and the mouse wheel. While scrolled up, new output does not pull the view down (the bottom says how many lines you have not seen); sending a new message goes back to the bottom.
- **Select and copy**: drag in the conversation column to highlight, release to copy to the clipboard (only the conversation text, without the labels on the left or the column on the right; long lines that were wrapped are joined back). A click or any key clears the highlight.
  While the highlight is shown, Ctrl-C copies instead of interrupting (as in Windows Terminal: Ctrl-C copies when there is a selection and interrupts when there is none); if a turn is running, the status line says to press Ctrl-C again to interrupt it.
  Over SSH, or on Linux without wl-copy/xclip, it is handed to the terminal (OSC 52: supported by iTerm2, tmux and Windows Terminal, not by Terminal.app).
  To use the terminal's own selection: hold Option while dragging in iTerm2, or type `/tui classic` for classic mode.
- The terminal's own search (Cmd+F) cannot find the conversation inside full-screen mode. When you leave full-screen mode (`/tui classic`, or quitting xj), the whole conversation is printed back to the normal screen, and then search finds it.
- It cannot start when the output is not a terminal, with `TERM=dumb`, in a window that is too small, or in the old Windows console; classic mode is used instead.

## Non-interactive mode (for scripts)

```bash
xj -p "What is this project's test command?"
git diff | xj -p "Write a one-line commit message for this change"
```

The reply goes to stdout; tool actions and the cost go to stderr. Anything that would ask you (edits, commands outside the whitelist, new domains) is refused; a folder you have never been asked about counts as untrusted.

What comes in through a pipe is data, not your question: `@path` inside it does not attach files. To attach one, put it in the `-p` question (`git diff | xj -p "check this change against @docs/style.md"`).

## The status bar

The bottom line: model · effort · mode · sandbox · context (used / summarizing threshold) · to-do (when a to-do list has open items, for example "to-do 2/5: Writing tests") · today's cost · this month's cost (against the limit). While working, it shows the latest progress. When the window is too narrow, items are dropped from the end. Without a sandbox, the whole bar is red.
Which items and in what order: `statusBar.fields` in the config.

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
- It works in read-only and plan modes and in untrusted folders too (it only reads; domains still ask). Full-auto mode still asks about domains; skip-all-permissions mode does not, but the "blocked without asking" list above still applies.
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
- Subagents cannot start subagents, and cannot use the to-do list, MCP or skills. The full run is saved in `subagents/` next to the session log (`/resume` only resumes the main conversation).

## Cost and budget

Amounts on screen are marked "about": they are **estimates** (local price table × usage), not a bill — providers change their prices, and their bill is what counts.

There are limits per turn, per day and per month (`budget` in the config; by default $5 per turn, $30 per day, $100 per month; 0 turns that one off). At 80% you get a notice, and at 100% it stops and asks you. `xj setup` does not ask about the budget; change it in the config. The price table is `~/.xing-janus/prices.json`, and every entry records its source and date; `xj doctor` warns if it has not been updated for three months. Local models (Ollama) count as $0. Requests for which the service reports no token usage show the cost as "unknown" (not $0), and if too many pile up in one turn it stops and asks you.

## Project rules, skills, MCP

- **AGENTS.md** (CLAUDE.md is read too): rules for coding agents, such as the test command or code style. Only read in trusted folders.
- **Skills**: `~/.xing-janus/skills/<name>/SKILL.md`, or `.agents/skills/` in the project. Normally only the name and description are given to the model, and the content is read when needed.
- **MCP servers**: only the ones listed in the config; they run in the sandbox by default, and every call asks (except those listed in `autoApprove` in the config).

## Flutter projects

Nothing to set up. With the sandbox on, `flutter` and `dart` cannot run inside it: the model asks to run that one command outside the sandbox and you are asked every time ("Tools that cannot run in the sandbox" in config.md), using your own environment (`JAVA_HOME` and `ANDROID_HOME` are passed through by default); for the same reason the sandbox no longer opens the Flutter SDK or the Pub cache, so it starts faster. Without the sandbox the usual rules apply.

## Language

The interface (everything xj prints) is in English or Traditional Chinese. On the first start xj asks which one you want (it suggests the one your system uses), and saves it as `uiLanguage` in the config. Switch at any time with `/language ui en` or `/language ui zh-TW`.
The model always gets its instructions and tool results in English; the language it replies in is a separate setting: `language` in the config (by default the same as the interface), or `/language reply <language>` for this session.

# Install, update, uninstall

## What you need first

- **macOS** (fully supported). On Linux, also install bubblewrap, socat, ripgrep and sandbox-runtime (see "The Linux sandbox" below).
- **Windows**: the native version needs Git for Windows and Node 24 or later; the sandbox is alpha and optional (see "Windows" and "Installing on Windows" below).
  WSL2 also works (it is Linux inside, and the sandbox is bubblewrap). With WSL2, four things to know:
  1. Inside WSL, install `bubblewrap`, `socat`, `ripgrep`, Node 24 or later, and sandbox-runtime (as in "The Linux sandbox" below).
  2. **Keep the project on WSL's own file system** (for example `~/code/project`), not under `/mnt/c/...`: crossing file systems is slow,
     and hard links and inodes behave differently — the secret-file checks rely on inodes (`xj doctor` warns about this).
  3. There is no keychain: keys go in a key file (permissions 600) or an environment variable. **Do not put key files or `~/.xing-janus` under `/mnt/c`**:
     `chmod 600` does nothing there, so there is no permission protection at all (`xj doctor` marks it as failed).
  4. Creating user namespaces must be allowed (bubblewrap needs it); recent WSL2 allows it by default.
- **Node 24 or later**: check with `node -v` (24 is the active LTS; 26 also works; 23 and 25 are out of support, do not use them).
- **ripgrep**: `brew install ripgrep` (the sandbox and search both use it). Windows: `winget install BurntSushi.ripgrep.MSVC` (only search uses it).

### Windows

- **Node 24 or later**: download the installer from nodejs.org and install it for the whole computer (the default `C:\Program Files\nodejs`).
  nvm-windows also works; the sandbox account can read its folder (xj opens it up).
- **Git for Windows** (includes Git Bash): https://git-scm.com/download/win . The bash tool runs commands with it, and permission checks work the same as on macOS and Linux.
  Without it only the powershell tool is left, and in PowerShell only whitelisted read-only forms (`Get-ChildItem`, `Get-Content`…) run without asking; everything else asks every time.
  If it is installed somewhere else, set `windows.gitBash` in the config (see config.md).
- **Terminal**: use Windows Terminal (built into Windows 11). In the old console, colors and pasting several lines may not work; `xj doctor` warns about it.
- **Sandbox (alpha, optional)**: commands run as a dedicated `srt-sandbox` account; the network is blocked by Windows Firewall and files by permissions (ACLs).
  It needs a one-time setup with administrator rights: in the Start menu, right-click Windows PowerShell → "Run as administrator", then:

  ```powershell
  npm install -g --prefix "$env:ProgramFiles\sandbox-runtime" --ignore-scripts @anthropic-ai/sandbox-runtime@0.0.79
  & "$env:ProgramFiles\sandbox-runtime\srt.cmd" windows-install
  ```

  **It must be installed for the whole computer** (the `--prefix` above), not in npm's default location (`%APPDATA%\npm` under your account):
  the sandbox account cannot read files under your account, so the sandbox would not start; and only administrators can change Program Files, so commands in the sandbox cannot replace it.
  xj finds this location by itself (`xj doctor` shows which copy it uses). If you followed older steps and installed it in `%APPDATA%\npm`, remove that copy with
  `npm uninstall -g @anthropic-ai/sandbox-runtime` in a normal PowerShell (xj does not use it either way).

  **Without the sandbox, every command asks you first** (the same as `--no-sandbox`: the whitelist does not apply); with it installed, whitelisted commands run directly.
  For the sandbox's limits, see "Windows" in troubleshooting.md.

### The Linux sandbox

The Linux sandbox also needs `apply-seccomp` (it blocks Unix sockets, for example borrowing ssh-agent or docker.sock). It is statically linked against
the GNU C Library (LGPL), so the xj archive does not include it. Install it from npm yourself (the version must match the one xj uses):

```bash
npm install -g --ignore-scripts @anthropic-ai/sandbox-runtime@0.0.79
```

- `--ignore-scripts`: do not run the package's install scripts (it does not need them).
- With nvm, install it with the Node version that runs xj. A changed npm prefix (such as `~/.npm-global`) is found too.
- If the install fails with a permission error (`EACCES`): switch to nvm, or set npm's prefix to your home folder (`npm config set prefix ~/.npm-global`) and install again; xj finds either.
- A copy installed where the sandbox can write (the project folder, a temp folder) is never used: commands in the sandbox could replace it.
- `xj doctor` shows which copy it uses. If none is found or the version is wrong, the sandbox row gives the exact command to run;
  when an xj update needs a different version, reinstall with that line too.
- Without it the sandbox cannot start, and xj refuses to run any command. To run commands without the sandbox, start with `--no-sandbox` (every command asks you first).

## Install

1. From the releases page, download your platform's `xj-<version>-<platform>.tar.gz`, `.sha256` and `.minisig`. `<platform>`:
   - macOS: `darwin` (one file for both Apple silicon and Intel)
   - Linux (including WSL): pick `linux-x64` or `linux-arm64` by CPU. If `uname -m` prints `x86_64`, pick x64; if it prints `aarch64`, pick arm64
   - Windows: `xj-<version>-windows.zip` (one file for x64 and arm64; steps in "Installing on Windows" below). With WSL2, pick the Linux file as above
2. Check that the file is intact and really from the publisher:

   ```bash
   shasum -a 256 -c xj-<version>-<platform>.tar.gz.sha256      # on Linux without shasum, use sha256sum -c
   minisign -Vm xj-<version>-<platform>.tar.gz -P <public key>
   ```

   **Keep your own copy of the public key the first time you get it** (for example in `~/.xing-janus/minisign.pub`), and verify every later update with your copy.
   The public key lives in the same repo as the program, so if you only ever take it from there, someone who tampers with the repo can swap the key too, and the signature check would not catch it.

   After verifying, read the line minisign prints (the trusted comment); it contains the version and file name:

   ```
   Trusted comment: xing-janus v0.1.0 xj-0.1.0-darwin.tar.gz
   ```

   The version must be the one you mean to install. A valid signature only means "the publisher signed some version", not that it is the latest:
   someone could put an old version (with its signature) in place, so never install a version older than the one you use now.

3. Unpack it wherever you like and link `xj` into your PATH:

   ```bash
   mkdir -p ~/.local/share ~/.local/bin
   tar -xzf xj-<version>-<platform>.tar.gz -C ~/.local/share
   ln -sf ~/.local/share/xj-<version>-<platform>/xj ~/.local/bin/xj
   ```

   macOS's default PATH does not include `~/.local/bin` (some Linux distributions do). If `command -v xj` prints nothing, add it in your shell's settings (check with `echo $SHELL`),
   then open a new terminal window:

   | shell | command |
   |---|---|
   | zsh (macOS default) | `echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc` |
   | bash (Linux) | `echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc` |
   | bash (macOS: the terminal opens a login shell, which reads a different file) | `echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bash_profile` |
   | fish | `fish_add_path ~/.local/bin` |

   For other shells, add `~/.local/bin` to PATH their own way. You are done when `xj --version` prints the version in a new window.

4. Linux: install sandbox-runtime as in "The Linux sandbox" above.
5. Run `xj doctor`. Every row should be ✓ or - (features you do not use).

The archive already contains the bundled program (`xj.mjs`) and everything it needs: no `npm install`, and no package install scripts run.

### Installing on Windows

First install what the "Windows" list above needs. In PowerShell:

1. Check that the file is intact:

   ```powershell
   (Get-FileHash xj-<version>-windows.zip -Algorithm SHA256).Hash.ToLower()
   Get-Content xj-<version>-windows.zip.sha256      # the first part must match
   ```

2. Verify the signature: download the Windows build of minisign from its GitHub releases (or `scoop install minisign`), then

   ```powershell
   minisign -Vm xj-<version>-windows.zip -P <public key>
   ```

   Again, keep your own copy of the public key, and check the version in the trusted comment (see step 2 above).
3. Unpack it into a folder without the version number (`%LOCALAPPDATA%\Programs\xj`) and add that to your PATH (for your account only):

   ```powershell
   $programs = "$env:LOCALAPPDATA\Programs"
   $dir = "$programs\xj"
   Expand-Archive xj-<version>-windows.zip -DestinationPath $programs -Force
   if (Test-Path $dir) { Remove-Item -Recurse -Force $dir }     # when updating: remove the old version
   Rename-Item "$programs\xj-<version>-windows" xj
   $path = [Environment]::GetEnvironmentVariable('Path', 'User')
   if (($path -split ';') -notcontains $dir) {                  # add to PATH only once
     [Environment]::SetEnvironmentVariable('Path', ((@($path -split ';') + $dir) | Where-Object { $_ }) -join ';', 'User')
   }
   ```

   The folder has no version number, so for an update you repeat this step and PATH stays the same (with one folder per version, the old one comes first and `xj` runs the old version).
   Windows that are already open do not see the new PATH: open a new terminal window (if a new Windows Terminal tab still cannot find it, close Windows Terminal completely and reopen it);
   you are done when `xj --version` prints the version. cmd and PowerShell use `xj.cmd`; Git Bash uses `xj` in the same folder.
   This snippet replaces forms like `%USERPROFILE%` in your PATH with the expanded paths (they work the same). To keep the original forms, search the Start menu for
   "Edit environment variables for your account" and add `%LOCALAPPDATA%\Programs\xj` to Path instead.
4. For the sandbox, install sandbox-runtime for the whole computer with administrator rights and run `windows-install`, as in "Windows" above.
5. Run `xj doctor`. Store the key with `xj setup` (it goes into Windows Credential Manager; if that fails, into a key file only you can read), or set the environment variable `XING_JANUS_ANTHROPIC_API_KEY`.

## Update

Xing Janus never goes online by itself to check for updates (its only outside connections are the model APIs). Check the releases page for a new version, then download, verify and unpack it as above, and point the `xj` link at the new folder.
Windows: quit any running xj first, then repeat step 3 of "Installing on Windows" (it replaces the files in `%LOCALAPPDATA%\Programs\xj`; PATH is not added again if it is already there). The config and session logs live in `~/.xing-janus/` and are not affected.

## Uninstall

```bash
xj reset                                           # delete ~/.xing-janus (lists what will go and asks first)
rm ~/.local/bin/xj && rm -r ~/.local/share/xj-*    # delete the program
security delete-generic-password -s xing-janus-anthropic   # the key in the keychain (same for openai and others)
```

Windows: after `xj reset`, in a normal PowerShell, delete the program folder and remove it from PATH (the PATH is your own account's; do not use administrator rights for this):

```powershell
$dir = "$env:LOCALAPPDATA\Programs\xj"
Remove-Item -Recurse -Force $dir
$path = [Environment]::GetEnvironmentVariable('Path', 'User')
[Environment]::SetEnvironmentVariable('Path', (($path -split ';') | Where-Object { $_ -and $_ -ne $dir }) -join ';', 'User')
```

If you installed the sandbox, then in PowerShell with administrator rights:

```powershell
& "$env:ProgramFiles\sandbox-runtime\srt.cmd" windows-uninstall    # removes the srt-sandbox account and the firewall rules
Remove-Item -Recurse "$env:ProgramFiles\sandbox-runtime"
```

`%ProgramData%\sandbox-runtime` is a folder the library leaves behind; delete it too for a full cleanup.

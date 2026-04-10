# dotfiles

Personal dotfiles managed with [chezmoi](https://www.chezmoi.io/).

## What's in here

| Path | Purpose |
| --- | --- |
| `dot_bash_profile`, `dot_bashrc` | Bash shell setup (PATH, aliases, conda/cargo/bun, starship, yazi wrapper). `dot_bash_profile` is a one-line stub that sources `dot_bashrc` |
| `dot_gitconfig` | Git user + git-lfs filters |
| `dot_tmux.conf` | tmux config (vim-style panes, tpm, resurrect, continuum) |
| `dot_vimrc` | Vim config |
| `dot_config/ghostty/` | Ghostty terminal |
| `dot_config/starship.toml` | Starship prompt |
| `dot_config/yazi/` | Yazi file manager (+ `glow` and `piper` plugins) |
| `dot_config/zellij/` | Zellij terminal multiplexer |
| `private_Documents/Keyboard Maestro macros/` | Keyboard Maestro macros (deployed with restricted perms) |

## chezmoi naming conventions

- `dot_foo` → `~/.foo`
- `private_foo` → deployed with `0600` / `0700` perms
- `readonly_foo` → deployed read-only

## Usage

Install chezmoi and initialize from this repo:

```bash
brew install chezmoi
chezmoi init <this-repo-url>
chezmoi diff      # preview changes
chezmoi apply     # write files to $HOME
```

Day-to-day:

```bash
chezmoi edit ~/.bashrc   # edit the source copy
chezmoi cd               # jump into the source directory
chezmoi apply            # re-apply after edits
```

## Upgrading bash on macOS

macOS ships `/bin/bash` as version **3.2.57 from 2014** — Apple froze it at the last
GPLv2 release and won't ship anything newer because bash switched to GPLv3. This old
bash lacks many modern features, including `enable-bracketed-paste` (added in bash 5.1).

**Symptom of still being on bash 3.2:** pasting text into the terminal shows garbage
characters like `00~` and `01~` wrapping the pasted content — these are the visible
remains of bracketed-paste escape sequences (`\e[200~` / `\e[201~`) that the ancient
readline doesn't know how to consume.

### Steps

1. **Install modern bash via Homebrew**

   ```bash
   brew install bash
   ```

   This installs to `/opt/homebrew/bin/bash` (Apple Silicon) or `/usr/local/bin/bash`
   (Intel). Verify:

   ```bash
   /opt/homebrew/bin/bash --version   # should print 5.x
   ```

2. **Whitelist it as a valid login shell**

   `chsh` refuses to switch to any shell that isn't listed in `/etc/shells`:

   ```bash
   sudo sh -c 'echo /opt/homebrew/bin/bash >> /etc/shells'
   ```

3. **Change your login shell**

   ```bash
   chsh -s /opt/homebrew/bin/bash
   ```

4. **Fully restart your terminal**

   Close **all** terminal windows (Ghostty, Terminal.app, iTerm2, etc.) and open a
   fresh one. `$SHELL` is captured at process start, so existing windows keep running
   the old bash until they're killed.

5. **Verify**

   ```bash
   echo "$BASH_VERSION"       # 5.x.y(...)-release
   echo "$SHELL"              # /opt/homebrew/bin/bash
   bind -V | grep paste       # enable-bracketed-paste is set to 'on'
   ```

   Try pasting something — no more `00~` / `01~` garbage.

### Why `.inputrc` tweaks don't help on bash 3.2

Setting `enable-bracketed-paste on` in `~/.inputrc` or via `bind` in `.bashrc` is a
**no-op on bash 3.2** — the variable doesn't exist in that version and readline
silently ignores it. The only real fix is actually running a bash that supports the
feature. Once you're on bash 5.1+, bracketed paste is **on by default**, so no
`.inputrc` entry is needed at all — which is why this repo doesn't ship one.

### Gotcha: `/bin/bash` is still 3.2 even after upgrading

Homebrew installs alongside the system bash — it does **not** replace `/bin/bash`.
Anything that hardcodes `#!/bin/bash` or explicitly invokes `/bin/bash` will still run
the old version. `chsh` only changes your *login* shell (the one your terminal starts
for you); it's `$SHELL`, not `/bin/bash`, that picks up the new version.

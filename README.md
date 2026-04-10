# dotfiles

Personal dotfiles managed with [chezmoi](https://www.chezmoi.io/).

## What's in here

| Path | Purpose |
| --- | --- |
| `dot_bash_profile`, `dot_bashrc`, `dot_inputrc` | Bash shell setup (PATH, aliases, conda/cargo/bun, starship, yazi wrapper) |
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

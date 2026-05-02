# CLAUDE.md

## What This Is

Personal dotfiles managed with [chezmoi](https://www.chezmoi.io/). Manages bash, git, ghostty, starship, yazi, zellij, Keyboard Maestro macros, and other configs.

## Key Paths

- chezmoi source: `~/.local/share/chezmoi/`
- chezmoi target: `~/` (files deployed to home directory)
- KM macros plist: `~/Library/Application Support/Keyboard Maestro/Keyboard Maestro Macros.plist`

## Common Workflows

```bash
# Preview what chezmoi would change on the live system
chezmoi diff

# Apply chezmoi source → live files
chezmoi apply

# Sync live file changes back to chezmoi source
chezmoi re-add <target-path>

# Sync KM macros after editing in Keyboard Maestro
chezmoi re-add ~/Library/Application\ Support/Keyboard\ Maestro/Keyboard\ Maestro\ Macros.plist
```

## Keyboard Maestro

- All macros live in a single binary plist: `Keyboard Maestro Macros.plist`
- KM always writes binary plist, even if you feed it XML — don't bother converting
- `.gitattributes` has `*.plist diff=plist` textconv so `git diff` shows XML
- New machine setup requires: `git config diff.plist.textconv 'plutil -convert xml1 -o -'`

## chezmoi Naming Conventions

- `dot_foo` → `~/.foo`
- `private_foo` → deployed with `0600`/`0700` perms
- `readonly_foo` → deployed read-only

## Notes

- `chezmoi diff` shows "what apply would do to live files" (source → live direction)
- `chezmoi re-add` goes the other way (live → source)
- `.chezmoiignore` prevents `README.md` and `CLAUDE.md` from being deployed to `$HOME`

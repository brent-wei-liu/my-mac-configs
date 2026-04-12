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

如果你直接改了 live 文件（目标位置的文件）而不是 chezmoi 源目录里的文件：

- `chezmoi re-add` — 把 live 文件的改动拉回源目录，让源状态追上你的改动
- `chezmoi apply` — 反过来，用源状态覆盖 live 文件，丢弃你的改动

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

## 常用 CLI 工具速查(中文)

这套 dotfiles 默认假设以下工具都已经装好。所有 shell 集成(快捷键、补全、环境变量)
都已经在 `dot_bashrc` 里配好了,带 `command -v` 或目录探测保护,**没装的工具不会报错**。

### 一键安装全部

```bash
brew install fzf fd lsd starship yazi zellij bat ripgrep
```

### fzf —— 模糊查找(必装)

**安装:**

```bash
brew install fzf
```

**注意:** 不要跑 `$(brew --prefix)/opt/fzf/install` —— 那个脚本会生成 `~/.fzf.bash`
并直接改你的 `~/.bashrc`,跟 chezmoi 的 source-of-truth 冲突。这个仓库的 `dot_bashrc`
里已经直接 source 了 brew 自带的 `key-bindings.bash` 和 `completion.bash`,效果完全
一样。

**常用快捷键(在任何 bash 提示符下):**

| 快捷键 | 作用 |
|---|---|
| `Ctrl+R` | **模糊搜索命令历史**(替代 bash 默认的反向搜索,大幅好用) |
| `Ctrl+T` | 在当前命令光标处插入一个/多个**模糊选择的文件路径** |
| `Alt+C` | **模糊选择目录**并 `cd` 进去 |
| `**<Tab>` | 模糊补全:`vim **<Tab>` / `cd **<Tab>` / `kill **<Tab>` 等 |

**fzf 界面内的操作:**

| 按键 | 作用 |
|---|---|
| `↑` / `↓` 或 `Ctrl+P` / `Ctrl+N` | 上下移动 |
| `Enter` | 选中并退出 |
| `Esc` 或 `Ctrl+C` | 取消 |
| `Tab` / `Shift+Tab` | 多选(只在支持多选的命令里) |
| `Ctrl+J` / `Ctrl+K` | 同上下移动(vim 风格) |

### fd —— `find` 的现代替代品

**安装:**

```bash
brew install fd
```

**为什么用 fd:**

- 比 `find` 快,默认并行
- **自动忽略 `.gitignore`** 里的内容和 `.git/` 目录
- 语法直观:`fd pattern` 而不是 `find . -name '*pattern*'`
- 默认彩色输出

**常用命令:**

```bash
fd readme                    # 当前目录递归找名字含 readme 的文件
fd -e md                     # 找所有 .md 文件
fd -H pattern                # 包括隐藏文件(默认跳过 . 开头的文件)
fd -t d node_modules         # 只找目录
fd -x rm {} \;               # 对每个匹配执行命令(类似 find -exec)
fd pattern /some/dir         # 在指定目录里找
```

**与 fzf 集成:** `dot_bashrc` 里设置了 `FZF_DEFAULT_COMMAND=fd ...`,所以 `Ctrl+T`
和 `Alt+C` 都会自动用 fd 做底层搜索 —— 在大目录里速度差异非常明显。

### lsd —— `ls` 替代,带图标和颜色

**安装:**

```bash
brew install lsd
```

**用法:** `dot_bashrc` 里有别名 `ll = lsd -lh`,所以直接:

```bash
ll                # 长格式 + 人类可读大小
ll -A             # 包括隐藏文件
lsd --tree        # 树形显示
lsd --tree -L 2   # 限制深度
```

需要 [Nerd Font](https://www.nerdfonts.com/) 才能正确显示图标(Ghostty 配置里已经
设好了)。

### starship —— 跨 shell 的 prompt

**安装:**

```bash
brew install starship
```

`dot_bashrc` 末尾有 `eval "$(starship init bash)"`,启动 bash 时自动接管 prompt。
配置文件在 `~/.config/starship.toml`,这个仓库里有 `dot_config/starship.toml`。
修改方式:`chezmoi edit ~/.config/starship.toml`,然后 `chezmoi apply`。

### yazi —— TUI 文件管理器(类 ranger)

**安装:**

```bash
brew install yazi
```

**用法:** `dot_bashrc` 里定义了 `y` 函数(不是简单 alias):

```bash
y                 # 打开 yazi
y /some/path      # 在指定路径打开
```

跟普通 `yazi` 命令的**关键区别**:`y` 在你退出 yazi 时,会把 yazi 里最后所在的
目录写回当前 shell 的 `cwd` —— 也就是说你可以在 yazi 里浏览/进出目录,退出后
shell 自动 `cd` 到最后那个目录。直接跑 `yazi` 没有这个效果。

**yazi 内部常用按键:**

| 按键 | 作用 |
|---|---|
| `h j k l` | vim 风格移动(`l` 进入,`h` 返回上级) |
| `/` | 当前目录搜索 |
| `Space` | 选中(可多选) |
| `y` `p` `d` | 复制 / 粘贴 / 剪切 |
| `a` | 新建文件/目录(末尾加 `/` 是目录) |
| `r` | 重命名 |
| `q` | 退出 |
| `Tab` | 在多个 tab 之间切换 |

### zellij —— 终端复用器(类 tmux)

**安装:**

```bash
brew install zellij
```

**用法:** `dot_bashrc` 里有别名 `zj = zellij`。

```bash
zj                          # 启动一个新会话
zj attach <session-name>    # 重新连接已有会话
zj a <session-name>         # 同上(简写)
zj list-sessions            # 列出所有会话
zj kill-session <name>      # 杀掉指定会话
```

**默认按键(zellij 自带,不是 vim/tmux 风格):**

| 按键 | 作用 |
|---|---|
| `Ctrl+p` | 进入 **pane** 模式(然后按字母操作 pane) |
| `Ctrl+t` | 进入 **tab** 模式 |
| `Ctrl+s` | 进入 **scroll** 模式(滚动历史) |
| `Ctrl+o` | 进入 **session** 模式,可以 detach |
| `Ctrl+o` 然后 `d` | **detach**(回到外层 shell,会话在后台保留) |
| `Ctrl+q` | 退出整个 zellij 会话 |

zellij 会在屏幕底部一直显示当前模式提示,新手不容易迷路。

### 其他建议工具(可选,装了用着舒服)

| 工具 | 用途 | 安装 |
|---|---|---|
| `bat` | `cat` 替代,带语法高亮和分页 | `brew install bat` |
| `ripgrep` (`rg`) | `grep` 替代,极快,默认尊重 `.gitignore` | `brew install ripgrep` |
| `delta` | git diff 的彩色高亮 pager | `brew install git-delta` |
| `eza` | `ls` 替代(`lsd` 的另一个流派,二选一即可) | `brew install eza` |
| `zoxide` | `cd` 替代,根据访问频率智能跳转 | `brew install zoxide` |
| `tldr` | 命令速查(简化版 man) | `brew install tldr` |
| `jq` | JSON 命令行处理器 | `brew install jq` |
| `gh` | GitHub 官方 CLI | `brew install gh` |

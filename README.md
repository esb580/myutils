# macOS Python Dev Environment Setup

A clean, repeatable setup for Python development on macOS using pyenv and Claude Code.

---

## Prerequisites

Apple Silicon Mac, macOS, zsh shell (default on modern macOS).

---

## 1. Homebrew

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

After install, add to `~/.zprofile` (Apple Silicon path):

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
source ~/.zprofile
```

Verify: `brew --version`

---

## 2. Core Tools

```bash
brew install git pyenv xz tmux vim mc
```

- `git` — newer than macOS system git
- `pyenv` — Python version manager
- `xz` — required for Python lzma support (must be installed before Python)
- `tmux` — terminal multiplexer
- `vim` — newer than macOS system vim
- `mc` — Midnight Commander file browser

---

## 3. Configure pyenv

Add to `~/.zshrc`:

```bash
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
source ~/.zshrc
```

---

## 4. Install Python

```bash
pyenv install 3        # installs latest Python 3.x
pyenv global 3.x.x    # set as default (use the version number printed above)
python --version       # verify
```

---

## 5. Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

> Requires Node.js. If not installed: `brew install node`

Configure global context file (auto-loaded every session):

```
~/.claude/CLAUDE.md
```

---

## 6. VS Code (optional but recommended)

Install from https://code.visualstudio.com

Extensions to install:
- Python (Microsoft)
- Claude Code (Anthropic)
- GitLens (GitKraken)
- Git Graph

---

## Starting a New Project

```bash
mkdir ~/git-repos/my-project && cd ~/git-repos/my-project
pyenv local 3.x.x                  # pins Python version, creates .python-version
python -m venv .venv               # create virtual environment
source .venv/bin/activate          # activate it
pip install <packages>             # install dependencies
```

Add a `CLAUDE.md` to the project root with context for Claude Code.

---

## dev Script — One-Command Layout

`~/bin/dev` launches a 3-pane tmux session for any project:

```
+------------------+------------------+
|                  |                  |
|   vim .          |   claude         |
|                  |                  |
+------------------------------------------+
|   terminal (shell)                        |
+------------------------------------------+
```

**One-time setup** — add `~/bin` to PATH in `~/.zshrc`:

```bash
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

**Usage:**

```bash
cd ~/git-repos/my-project
dev
```

That's it. The session is named after the folder.

**Tearing down:** Run `bye` from any pane inside the session to kill all panes and exit in one shot:

```bash
bye
```

`bye` uses the same `basename "$PWD"` naming as `dev` so they always match. It will warn you if no matching session is found.

To switch between project sessions:

```bash
tmux ls                        # list sessions
tmux attach -t my-project      # reattach to a session
```

---

## tmux Quick Reference

| Action | Keys |
|--------|------|
| Split panes side by side | `Ctrl+B` then `%` |
| Split panes top/bottom | `Ctrl+B` then `"` |
| Navigate panes | `Ctrl+B` then arrow keys |
| New window | `Ctrl+B` then `c` |
| Switch windows | `Ctrl+B` then `0-9` |
| Close pane | `exit` or `Ctrl+D` |

Typical layout: Claude Code in one pane, shell in another.

**Enable mouse support** (click to switch panes and windows):

Add to `~/.tmux.conf`:

```bash
echo 'set -g mouse on' >> ~/.tmux.conf
```

Reload in a running session:

```
Ctrl+B then :
source-file ~/.tmux.conf
```

---

## Claude Code Quick Reference

```bash
claude          # start interactive session
/clear          # clear conversation history
/exit           # end session
```

- `~/.claude/CLAUDE.md` — global context, auto-loaded every session
- `<project>/CLAUDE.md` — project context, auto-loaded when in that directory

---

## Tool Primers

### vim Basics

```
vim <file>       # open a file
i                # enter insert mode (start typing)
Esc              # return to normal mode
:w               # save
:q               # quit
:wq              # save and quit
:q!              # quit without saving
dd               # delete current line
u                # undo
Ctrl+R           # redo
/word            # search for "word"  (n = next match)
:set number      # show line numbers
```

---

### mc (Midnight Commander) Basics

```
mc               # launch mc
Arrow keys       # navigate files
Enter            # open directory or file
Tab              # switch between left/right panels
F3               # view file contents
F4               # edit file (opens in $EDITOR, see below)
F5               # copy file
F6               # move/rename file
F8               # delete file
F10              # quit mc
Ctrl+O           # toggle panels off/on (shows shell underneath)
```

---

### vim + mc Integration

The key is setting `$EDITOR` so mc knows to use vim when you press F4.

Add to `~/.zshrc`:

```bash
export EDITOR=vim
```

Reload: `source ~/.zshrc`

Now pressing **F4** on any file in mc opens it directly in vim. Save and quit (`:wq`) to return to mc.

**Opening mc from vim:**

- Press `Ctrl+Z` in vim to suspend it and drop to your shell
- Run `mc` normally
- When done, type `fg` to return to your vim session

**Tip:** In mc, `Ctrl+O` hides the panels and gives you a plain shell prompt. Type `exit` to bring panels back. Handy for running commands without fully leaving mc.

---

### htop

```
htop             # launch (better top — shows CPU, memory, processes)
F6               # sort by column (CPU% and MEM% are most useful)
F9               # kill a selected process
F10 or q         # quit
```

Arrow keys navigate the process list. Useful for watching Python scripts eating CPU or spotting runaway processes.

---

### iTerm2 (recommended terminal replacement)

Better than Terminal.app for tmux users. Key advantages:
- "Copy to clipboard on selection" — highlighted text is instantly in your clipboard, no Cmd+C needed
- Handles tmux mouse mode cleanly
- Better font rendering, split pane support, profiles

```bash
brew install --cask iterm2
```

**Enable copy on select:** iTerm2 menu → Settings → General → Selection → check "Copy to clipboard on selection"

**Text selection in tmux:** Hold `Option` while clicking and dragging. Text is copied automatically if the above setting is on.

---

## Shell Configuration (~/.zshrc additions)

### ls Colors (Night Owl iTerm2 theme)

Makes `ls` output colored using the terminal's ANSI palette. Night Owl remaps those colors to its purple/blue/cyan scheme automatically.

Add to `~/.zshrc`:

```bash
# ls aliases
alias ll='ls -lAh'

# ls colors (Night Owl iTerm2 theme)
export CLICOLOR=1
export LSCOLORS=FxfxFxDxfxegedabagacad
```

Reload: `source ~/.zshrc`

**LSCOLORS key** (each pair = foreground + background, `x` = default):

| Code | Meaning |
|------|---------|
| `F` | bold magenta (light purple) |
| `f` | regular magenta (purple) |
| `D` | bold yellow |
| `x` | default background |

**What each item shows as:**

| Item | Color |
|------|-------|
| Directories | bold light purple |
| Symlinks | regular purple |
| Sockets | bold light purple |
| Pipes | bold yellow |
| Executables | regular purple |

**`ll` alias flags:**
- `-l` — long format (permissions, size, date)
- `-A` — show hidden files (`.dotfiles`), skip `.` and `..`
- `-h` — human-readable sizes (`1.2M` instead of `1234567`)

---

## SSH Key Setup for GitHub

One-time setup to authenticate with GitHub over SSH.

**1. Generate a key:**

```bash
ssh-keygen -t ed25519 -C "your@email.com"
# accept default path (~/.ssh/id_ed25519), set a passphrase or leave blank
```

**2. Add to ssh-agent:**

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

**3. Copy your public key:**

```bash
cat ~/.ssh/id_ed25519.pub
# copy the entire output
```

**4. Add to GitHub:**

GitHub → Settings → SSH and GPG keys → New SSH key → paste and save

**5. Test it:**

```bash
ssh -T git@github.com
# should say: Hi username! You've successfully authenticated...
```

---

## Syncing this README to GitHub (myutils repo)

This guide lives at `~/git-repos/README.md` and is tracked in the `myutils` repo on GitHub.

**First-time setup:**

```bash
cd ~/git-repos
git init
git remote add origin git@github.com:esb580/myutils.git
git fetch origin
git checkout -b main
git add README.md
git commit -m "Add macOS dev environment setup guide"
git push -u origin main
```

> If the remote already has a `main` branch with content, use `git pull origin main --allow-unrelated-histories` before pushing.

**Pushing future updates:**

```bash
cd ~/git-repos
git add README.md
git commit -m "Update setup guide"
git push
```

---

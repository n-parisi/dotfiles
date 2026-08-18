# dotfiles

Personal, machine-agnostic dotfiles managed with [chezmoi](https://chezmoi.io)
and [mise](https://mise.jdx.dev). This repo contains only generic, safe-for-any-machine
configuration. Work-specific config lives in a separate private overlay repo
and is layered on top automatically on machines flagged as work machines (see
[Work machines](#work-machines-overlay)).

## Quick start

```sh
sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply n-parisi/dotfiles
```

`chezmoi init` prompts for the following before the first apply:

| Prompt | Purpose | Example / default |
|---|---|---|
| `email` | Used for `~/.gitconfig` | `you@example.com` |
| `region` | Cosmetic accent theme applied to tmux, starship, and fastfetch | `kanto` (default), `johto`, `hoenn` |
| `work machine?` | Gates whether the private work overlay is fetched | `n` on personal machines |

On a personal machine (`work = n`), nothing further is required. On first
apply, mise installs the pinned toolchain and chezmoi clones its two git-repo
dependencies (prezto, tpm). This can take a few minutes the first time.

## Work machines (overlay)

Answering `y` to "work machine?" prompts for the private overlay repo URL.
chezmoi then clones that repo to `~/.local/share/dotfiles-work` and layers it
on top of the base config:

| File | What the overlay adds |
|---|---|
| `~/.zshrc` | Sources `zshrc.work` (work environment variables, aliases, account IDs) |
| `~/.gitconfig` | Includes `gitconfig.work` (work git identity/signing config) |

Personal machines never fetch the overlay — the `.chezmoiexternal` entry for
it is gated on `work = true` — so no internal URLs or work-specific config
ever reach this public repo.

## What's managed

### Shell — zsh + prezto

| Area | Detail |
|---|---|
| Framework | [prezto](https://github.com/sorin-ionescu/prezto), modules: environment, terminal, editor, history, directory, completion, history-substring-search, git, prompt, syntax-highlighting |
| Editor | Helix (`hx`) is `$EDITOR`/`$VISUAL` |
| Git aliases | `gst`, `gd`, `gsw`, `grb`, `gl`, `ga`, `gc`, `gco`, `glol` |
| Tool aliases | `l`/`ls` → `lsd`, `cat` → `bat` |
| Tmux shortcut | `t` → `tmux new-session -A -s <region>` |
| SSH wrapper | Sets the terminal title to the target host for the duration of the connection |
| Navigation | zoxide replaces `cd` (initialized last so nothing overrides its hook) |
| Other CLI tools wired in | fzf, bat, lsd, glow |

### Prompt & startup — starship, fastfetch, pokefetch

| Component | Purpose |
|---|---|
| starship | Minimal one-line prompt (`user on ◓<region> in <context>`); accent color keyed to `region` (kanto = blue, johto = gold, hoenn = green) |
| fastfetch | System-info panel shown on every new shell via the `pokefetch` wrapper |
| pokefetch | Runs fastfetch with a random Pokémon sprite (via `pokeget`) in place of the OS logo, sized to match the info panel. Falls back to plain fastfetch if `pokeget` hasn't been built yet (fresh machine, before bootstrap completes) |

### tmux

| Area | Detail |
|---|---|
| Prefix | `C-space` (not the tmux default `C-b`) |
| Window navigation (no prefix) | `M-t` new window · `M-[`/`M-]` prev/next · `M-{`/`M-}` swap · `M-.` rename · `M-w` kill pane |
| Splits | `\|` / `-` |
| Pane navigation | Arrow keys, no prefix |
| Resize | prefix + `M-arrow` |
| Status bar | Region-accented to match starship |
| Claude Code integration | Each window tab can show a glyph (`●` / `⏳` / `✅`) driven by Claude Code hooks via `~/.local/bin/claude-tmux-state`, indicating at a glance which window Claude is working in, waiting on input in, or has finished in |
| Copy mode | `Enter` = "smart" copy (collapses soft-wrapped lines, keeps paragraph/list breaks — good for prose); `Y` = literal copy (preserves every newline — good for code); both go out over OSC 52 |
| URL grab | `prefix u` greps the pane for URLs, fzf-picks one, and copies it |
| Session persistence | tmux-resurrect / tmux-continuum autosave every 15 min and restore layout/cwd on server start; manual save/restore via `prefix Ctrl-s` / `Ctrl-r` |

### Git & code review

| Tool | Role |
|---|---|
| `delta` | Pager for `git diff` and interactive staging — side-by-side view, line numbers, Dracula theme, `zdiff3` conflict markers |
| `gh` | Wired as the credential helper for github.com / gist.github.com over HTTPS — run `gh auth login` once and `git push` works from then on |
| `tig` | Tuned for relative dates, abbreviated authors, mouse support, and first-parent diffs |

### Editor

| Editor | Role |
|---|---|
| Helix (`hx`) | Daily driver — kanagawa theme, relative line numbers, auto-save, multi-buffer bufferline, hidden files visible in the file picker, indent guides, soft wrap, `jdtls` wired up as the Java language server |
| vim | Lightweight fallback — line numbers, sane search/indent defaults, mouse off |

### Toolchain — mise

`~/.config/mise/config.toml` is the source of truth for installed tool
versions. `update` (see [Everyday use](#everyday-use)) keeps everything
current.

**CLI tools** (pinned to `latest`, mise resolves the correct arch/libc build per machine):

| Tool | Purpose |
|---|---|
| `bat` | `cat` replacement with syntax highlighting |
| `delta` | Git diff pager |
| `lsd` | `ls` replacement with icons/color |
| `fzf` | Fuzzy finder |
| `zoxide` | Smarter `cd` |
| `starship` | Shell prompt |
| `glow` | Markdown renderer |
| `gh` | GitHub CLI — auth + push for this repo |
| `bun` | JS runtime and package manager |
| `fastfetch` | System-info banner (polyfilled build on AL2 for glibc compatibility) |
| `helix` | Editor (mise-managed everywhere except AL2 — see below) |

**Language runtimes** (pinned for reproducibility):

| Runtime | Version |
|---|---|
| Node | `20` (mise upgrade moves within the pin) |
| Python | `3.12`, `3.11`, `3.10` |
| Rust | `latest` (also used to build helix/pokeget from source on AL2) |
| Java | Corretto 21 (mise sets `JAVA_HOME` automatically when active) |

**Bootstrapped outside mise** — a couple of tools mise can't cover:

| Tool | Why it's not mise-managed | How it's installed |
|---|---|---|
| `jdtls` (Java language server) | Eclipse milestone download, no mise backend | `run_once_before_install-packages.sh.tmpl` downloads and pins a specific version |
| `helix` on Amazon Linux 2 only | Prebuilt release needs glibc ≥ 2.29; AL2 ships 2.26 | Built from source via `run_onchange_after_install-mise-tools.sh.tmpl` |
| `pokeget` | Prebuilt release needs glibc ≥ 2.34 (dead on AL2); not published to crates.io | Built from source from its git repo on every machine, `--locked` (required — an unpinned build pulls a broken `zune-jpeg` version) |

### Theming

The `region` answer from `chezmoi init` (`kanto` / `johto` / `hoenn`) threads
one accent color and a matching Pokémon flavor through three tools at once:
tmux's status bar, starship's prompt, and fastfetch's banner.

## Everyday use

| Command | Effect |
|---|---|
| `update` | Pull + apply dotfiles (refreshing prezto/tpm/overlay), then `mise upgrade` |
| `update -n` | Same as above, but preview-only — no changes made |
| `chezmoi edit <file>` | Edit a chezmoi-managed file |
| `chezmoi apply` | Apply pending changes to managed files |

## Not covered here

This README is a map of what's set up and why, not an exhaustive reference.
For full keybindings and options, read the source templates directly —
`dot_tmux.conf.tmpl`, `executable_dot_zshrc.tmpl`, and the files under
`private_dot_config/`.

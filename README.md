# dotfiles

Personal, machine-agnostic dotfiles managed with [chezmoi](https://chezmoi.io)
and [mise](https://mise.jdx.dev). Generic config only — safe for any machine.
Work-specific config lives in a separate private overlay repo and is layered on
top automatically on work machines (see below).

## New machine

```sh
sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply n-parisi/dotfiles
```

chezmoi prompts for:

- **email** — used for `~/.gitconfig`.
- **region** — cosmetic theme (`kanto`/`johto`/`hoenn`) for tmux/starship/fastfetch.
- **work machine?** — `n` on a personal machine (nothing else needed).

On first apply, mise installs the pinned toolchain (JDK/node/python/rust + CLI
tools) and chezmoi clones prezto + tpm. This can take a few minutes once.

## Work machines (overlay)

Answer `y` to "work machine?" and provide the private overlay repo URL when
prompted. chezmoi then clones that repo to `~/.local/share/dotfiles-work` and:

- `~/.zshrc` sources `zshrc.work` (work env, aliases, account IDs, …).
- `~/.gitconfig` includes `gitconfig.work`.

Personal machines never fetch the overlay (the `.chezmoiexternal` entry is gated
on `work = true`), so no internal URLs or config ever reach this public repo.

## What's here

A tour of what's managed and how the pieces fit together. Not an exhaustive
keybind reference — see the source files for that (`dot_tmux.conf.tmpl`,
`executable_dot_zshrc.tmpl`, etc.).

### Shell — zsh + prezto

Prezto modules: environment, terminal, editor, history, directory, completion,
history-substring-search, git, prompt, syntax-highlighting. Helix (`hx`) is
`$EDITOR`/`$VISUAL`. Notable aliases: `gst`/`gd`/`gsw`/`grb`/`gl`/`ga`/`gc`/`gco`/`glol`
(git), `l`/`ls` → `lsd`, `cat` → `bat`, `t` → `tmux new-session -A -s <region>`.
`ssh` is wrapped to set the terminal title to the target host for the
connection's duration. zoxide replaces `cd` (must init last so nothing
clobbers its hook); fzf/bat/lsd/glow round out the CLI.

### Prompt & startup — starship, fastfetch, pokefetch

starship is a minimal one-liner prompt (`user on ◓<region> in <context>`) with
an accent color keyed to `region` (kanto=blue, johto=gold, hoenn=green).
Every new shell runs `pokefetch`: a fastfetch system-info panel with a random
Pokémon sprite (via `pokeget`) as the logo instead of an OS logo, sized to
match the info panel. Falls back to plain fastfetch if `pokeget` isn't built
yet (fresh machine, before the bootstrap script runs).

### tmux

Prefix is `C-space` (not the default `C-b`). Window nav without prefix:
`M-t` new window, `M-[`/`M-]` prev/next, `M-{`/`M-}` swap, `M-.` rename,
`M-w` kill pane. Splits: `|`/`-`. Pane nav: arrow keys (no prefix). Resize:
prefix + `M-arrow`. The status bar is region-accented to match starship, and
each window tab can show a glyph (`●`/`⏳`/`✅`) driven by Claude Code hooks via
`~/.local/bin/claude-tmux-state` — a glance tells you which window Claude is
working in, waiting on you, or has finished in. Copy mode: `Enter` is "smart"
copy (collapses soft-wrapped lines, keeps paragraph/list breaks — good for
prose), `Y` is literal copy (preserves every newline — good for code); both go
out over OSC 52. `prefix u` greps the pane for URLs, fzf-picks one, and copies
it. `tmux-resurrect`/`tmux-continuum` autosave every 15 min and restore
layout/cwd on server start; manual save/restore is `prefix Ctrl-s`/`Ctrl-r`.

### Git & code review

`delta` is the pager for `git diff` and interactive staging (side-by-side,
line numbers, Dracula theme), with `zdiff3` conflict markers. `gh` is wired as
the credential helper for github.com/gist.github.com HTTPS, so `gh auth login`
once and `git push` just works. `tig` is tuned for relative dates, abbreviated
authors, mouse support, and first-parent diffs.

### Editor

Helix (`hx`) is the daily driver: kanagawa theme, relative line numbers,
auto-save, multi-buffer bufferline, hidden files visible in the picker, indent
guides, soft wrap, and `jdtls` wired up as the Java language server. vimrc is
kept around as a lightweight fallback (line numbers, sane search/indent,
mouse off).

### Toolchain — mise

`~/.config/mise/config.toml` pins CLI tools (bat, delta, lsd, fzf, zoxide,
starship, glow, gh, bun, fastfetch, helix) at `latest`, and languages: node 20,
python 3.10-3.12, rust latest, java (Corretto 21). `update` (see below) keeps
all of it current. A couple of tools mise can't cover are bootstrapped by
`run_once`/`run_onchange` scripts instead — notably `jdtls` (Eclipse milestone
download, no mise backend) and, only on old-glibc boxes like Amazon Linux 2,
`helix`/`pokeget` built from source because their prebuilt releases need newer
glibc than AL2 ships.

### Theming

The `region` answer (kanto/johto/hoenn) from `chezmoi init` threads one accent
color and a matching Pokémon flavor through tmux's status bar, starship's
prompt, and fastfetch's banner — one knob, three tools.

## Everyday use

- `update` — pull + apply dotfiles (refreshes prezto/tpm/overlay) then
  `mise upgrade`. `update -n` previews without changing anything.
- `chezmoi edit <file>` / `chezmoi apply` — edit and apply managed files.

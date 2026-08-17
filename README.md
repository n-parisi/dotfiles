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
- **region** — cosmetic theme (`kanto`/`johto`) for tmux/starship/fastfetch.
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

## Everyday use

- `update` — pull + apply dotfiles (refreshes prezto/tpm/overlay) then
  `mise upgrade`. `update -n` previews without changing anything.
- `chezmoi edit <file>` / `chezmoi apply` — edit and apply managed files.

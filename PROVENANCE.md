# Provenance

The `dotfiles/` tree in this repository is vendored from **ML4W OS 2.17**
("My Linux For Work") by Stephan Raabe:

- upstream: [`mylinuxforwork/dotfiles`](https://github.com/mylinuxforwork/dotfiles)
- pinned commit: `aeadb93395fa2e64a575161696fed978f0588f12`
  (the ML4W 2.17 release commit, merged 2026-10-12)
- license: **GPL-3.0** — `LICENSE` in this repository is ML4W's own license
  file, carried over unchanged. Every file under `dotfiles/` keeps ML4W's
  original copyright and license; see the headers in the tree and the upstream
  repository for the full text.

## What was taken

The complete contents of upstream's `dotfiles/` directory — the tree ML4W's
own installer copies into a user's home:

```
.config/hypr          hyprland.lua, input.lua, monitors.lua, gestures.lua,
                      workspaces.lua, colors.lua, functions.lua, conf/, assets/,
                      effects/, scripts/, hypridle/hyprlock/hyprpaper/hyprsunset
.config/ml4w          settings, listeners, scripts, themes, bin, tpl, colors, assets
.config/quickshell    statusbar, sidebar, wallpaper, welcome and overview apps
.config/waybar        the alternative bar
.config/swaync        notifications
.config/walker        launcher
.config/wlogout       power menu
.config/rofi          menus
.config/kitty         terminal
.config/matugen       wallpaper-driven theming
.config/btop fastfetch ohmyposh vim qt6ct gtk-3.0 gtk-4.0 fish xsettingsd
.config/bashrc .config/zshrc .gtkrc-2.0 .Xresources .bashrc .zshrc
```

The upstream layout is preserved (`dotfiles/` at the repo root, not flattened)
so a future re-pin can be diffed directory-for-directory against a fresh
upstream checkout.

## What was deliberately excluded

- **`.config/ml4w/wallpapers/` (13 image files, ~41 MB).** Not needed to boot:
  `hypr/hyprpaper.conf` defaults to `hypr/assets/blank.png`, which is vendored.
  The directory itself is kept so the wallpaper tooling finds its expected
  path; it ships empty. Bring your own images.
- **Upstream's installer machinery** (`setup/`, `install.sh` handling,
  `*.dotinst`, `.github/`, `dev/`). That is Arch-oriented and does not apply to
  Copper, which stages the tree itself.

## How Copper consumes this

The tree is copied into the Copper image by the `hypr` stage of
`12hrformat/copperlinux`'s `iso/build.sh`, fetched at a commit SHA pinned in
that script and unpacked into `/usr/share/copper/hyprland-skel/`. The
first-boot wizard copies it into the chosen user's home when the desktop
answer is Hyprland. Nothing here floats to `main`: the pin is a commit, not a
branch.

## Re-pinning

To move to a newer ML4W:

```
git clone https://github.com/mylinuxforwork/dotfiles
git -C dotfiles checkout <new-commit>
rm -rf dotfiles/.config/ml4w/wallpapers/*
rsync -a --delete dotfiles/dotfiles/ dotfiles-vendored/
```

Update the pin above and the SHA in copperlinux's `build_hypr`, recompute the
tarball hash, and note the new upstream commit here. ML4W 2.13+ requires
Hyprland 0.55+ (Lua config format); the cppiber PPA version in copperlinux's
build stage must stay at or above that.

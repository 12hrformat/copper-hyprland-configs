# Hyprland on Copper Linux

How the optional Hyprland session works, what it needs, and how it plugs into
Copper. This is a **design document** — the pieces marked *planned* do not exist
in `copperlinux` yet.

---

## 1. Shape of the thing

Copper is a from-source, systemd-free Linux. It boots a GRUB image,
`copper-init` (PID 1) brings up the base, and on first boot the wizard in
`iso/firstboot/copper-firstboot.c` asks a few questions (name, user, hostname,
passwords, timezone) in one centred table. XFCE is already in the image and is
started from the tty with `startxfce`.

We add a second desktop, **Hyprland**, configured with the **ML4W OS** dotfiles.

- **Choice.** The first-boot table gains one more question: *XFCE or Hyprland?*
- **XFCE.** Unchanged — it is already in the ISO.
- **Hyprland.** The compositor and its essentials are baked into the ISO at
  build time; anything optional is fetched on the spot from Copper's **ingot**
  package repo. The answer is recorded as the default session.
- **Launch.** Started from the tty with **`starthyprland`**, the Hyprland twin of
  `startxfce.sh`.

The dotfiles themselves are the contents of this repo.

---

## 2. Why ML4W needs Hyprland 0.55+

ML4W OS **2.17** (current) converted its whole Hyprland configuration to
Hyprland's **Lua** config format:

```
hyprland.lua  input.lua  monitors.lua  gestures.lua  workspaces.lua
conf/animation.lua  conf/keybinding.lua  conf/decoration.lua  ...
```

Lua configuration landed in Hyprland **0.55**. ML4W 2.13 was the first release
to require it. Older ML4W releases used `hyprlang` `.conf` files, but 2.17 — the
current one — does not.

Copper's desktop closure is built from **Ubuntu 24.04 "noble"** (glibc 2.39).
Where Hyprland is available:

| suite | Hyprland | verdict |
|---|---|---|
| noble (24.04) | — | not packaged |
| plucky (25.04) | 0.41.2 | far too old |
| resolute (26.04) | 0.53.3 | still below 0.55 |
| Debian sid | 0.56.2 | right version, wrong glibc (built against sid) |
| **`ppa:cppiber/hyprland`, noble** | **0.56.2** | ✅ right version, right glibc |

The **cppiber/hyprland PPA** is the same source ML4W's own Ubuntu setup uses.
It publishes **for noble**, so its binaries link against noble's glibc 2.39 —
exactly what the rest of Copper's desktop stage already carries. That means *no
suite bump, no glibc swap, and no source build of the compositor*.

---

## 3. Packages

Three sources: the PPA, one source build, and the normal noble/universe pool.

### 3.1 From `ppa:cppiber/hyprland` (noble)

Verified against the PPA's noble index on 2026-10-15. Versions in the 0.56.2
series unless noted.

**Compositor and its libraries**

| package | version | role |
|---|---|---|
| `hyprland` | `0.56.2-1ppa2` | the compositor |
| `hyprland-data` | `0.44.0-10ppa1` | shipped data files |
| `hyprland-protocols` | `0.7.1-1ppa2` | wayland protocol XML |
| `libaquamarine14` | `0.15.1-1ppa1` | backend / renderer |
| `libhyprutils13` | `0.14.2-1ppa1` | shared utilities |
| `libhyprlang2` | `0.6.8-3ppa2` | config language |
| `libhyprcursor1` | `0.1.13-1ppa1` | cursor themes |
| `libhyprgraphics4` | `0.5.1-1ppa3` | graphics helpers |
| `libhyprtoolkit6` | `0.6.0-1ppa3` | toolkit |
| `libhyprwire3` | `0.3.1-1ppa4` | IPC |
| `hyprwire-scanner` | `0.3.1-1ppa4` | codegen |
| `hyprwayland-scanner` | `0.4.6-1ppa1` | wayland codegen |
| `libdisplay-info3` | `0.3.0-1` | EDID parsing |
| `libinput10` | `1.31.1-1` | input (noble's is too old) |
| `libxkbcommon0` | `1.12.3-1` | keymaps (noble's is too old) |
| `libxkbcommon-x11-0` | `1.12.3-1` | X11 keymaps |
| `libxkbregistry0` | `1.12.3-1` | keyboard model registry |
| `libudis86.1` | `1.7.2+186-1ppa1` | disassembler (plugin host) |
| `libxcb-errors1` | `1.0.1-1ppa1` | xcb error strings |
| `libgtk-layer-shell0` | `0.9.0-1ppa1` | layer-shell for GTK apps |
| `libsdbus-c++2` | `2.1.0-1ppa2` | D-Bus bindings |
| `libpolkit-qt6-1-1` | `0.200.0-4ppa1` | polkit Qt6 bindings |

**Hypr ecosystem tools** (what ML4W drives directly)

| package | version |
|---|---|
| `hypridle` | `0.1.8-1ppa1` |
| `hyprlock` | `0.9.6-1ppa1` |
| `hyprpaper` | `0.8.4-1ppa3` |
| `hyprpicker` | `0.4.7-1ppa2` |
| `hyprsunset` | `0.4.0-1ppa2` |
| `hyprpolkitagent` | `0.2.0-1ppa1` |
| `hyprland-guiutils` | `0.2.2-1ppa2` |
| `hyprland-qtutils` | `0.1.5-1ppa3` |
| `hyprland-qt-support0` | `0.1.0-1ppa4` |
| `hyprsysteminfo` | `0.2.0-1ppa4` |
| `hyprshutdown` | `0.1.1-1ppa3` |
| `hyprlauncher` | `0.1.6-1ppa4` |

**Session / portal / bar**

| package | version |
|---|---|
| `xdg-desktop-portal-hyprland` | `1.4.1-1ppa1` |
| `waybar` | `0.14.0-1ppa1` (noble's 0.9.24 is too old) |
| `uwsm` | `0.26.4-1ppa1` |
| `fuzzel` | `1.12.0-1ppa1` |

**Optional window-layout plugins** (off by default): `hyprland-plugins`,
`hy3`, `hyprscroller`, `hyprsplit`, `hyprspace`.

### 3.2 Built from source

| component | why | pin |
|---|---|---|
| **Quickshell** | ML4W 2.12+ uses it for the status bar, sidebar, welcome/wallpaper/overview. It is not packaged in Ubuntu, so ML4W's own Ubuntu setup builds it. Upstream mirror: [`quickshell-mirror/quickshell`](https://github.com/quickshell-mirror/quickshell) (the author's repo moved; `codifryed/quickshell` 404s). | commit `4df562dfb2475a9057f0f33a8db75808efad8670` |

Quickshell needs Qt 6 (Base/Declarative/Wayland/Svg + the `qml6-module-*`
runtime modules) and `libgirepository`/`libcairo` headers. ML4W's Ubuntu preflight
also adds the `ppa:avengemedia/danklinux` PPA (for `cliphist`) and the Charm apt
repo (for `gum`).

### 3.3 From noble / universe

Everything else is ordinary Ubuntu. ML4W ships its dependency lists in
`setup/dependencies/`; the two that matter are quoted verbatim below so the
package set is reproducible rather than retyped.

**`setup/dependencies/packages-ubuntu`**

```
hyprland hyprland-guiutils hyprpolkitagent hyprsunset sway-notification-center
fonts-font-awesome tesseract-ocr tesseract-ocr-eng libsecret-1-0 libnotify-bin
network-manager-gnome nm-connection-editor power-profiles-daemon
gnome-themes-extra gnome-themes-extra-data gnome-themes-ubuntu gvfs-backends
qml6-module-qt5compat-graphicaleffects qml6-module-qt-labs-folderlistmodel
qml6-module-qtquick-effects qml6-module-qtquick-shapes
qml6-module-qtquick-controls qml6-module-qtquick-layouts libgirepository-2.0-dev
libcairo2-dev lua5.4 imagemagick pipx breeze-icon-theme
```

**`setup/dependencies/packages`** (distro-independent part)

```
wget curl git rsync unzip tar jq vim inotify-tools udisks2 gvfs udiskie
hyprpaper hyprlock hypridle hyprpicker xdg-desktop-portal-hyprland kitty vlc
waybar rofi nwg-look pavucontrol neovim blueman qt6ct nautilus
gnome-text-editor firefox xdg-user-dirs xdg-desktop-portal-gtk figlet
fastfetch htop xclip zsh fzf brightnessctl tumbler slurp cliphist grim
wl-clipboard btop cargo eza quickshell
```

The distro-agnostic list assumes the distro already provides `hypr*` and
`quickshell`; on Copper those come from §3.1 and §3.2.

### 3.4 What actually gets baked vs fetched

- **Baked into the ISO** (so Hyprland can start offline): everything in §3.1,
  `xdg-desktop-portal-gtk`, a terminal (`kitty`/`foot`), fonts, and the Quickshell
  build.
- **Fetched on demand via ingot** once the user picks Hyprland: the heavier or
  optional bits (browsers, editors, media, extra themes) — see §8.

---

## 4. How the session starts (no systemd)

ML4W assumes a lot of things Copper does not have: `systemd --user`,
`graphical-session.target`, a display manager, logind, polkitd. Its Ubuntu post
script quietly **masks** ML4W's own `waybar.service`, `hypridle.service` and
`hyprsunset.service` because they duplicate what `autostart.lua` already
`exec-once`s — so even on a systemd distro the real startup path is
`hyprland.lua` → `conf/autostart.lua`, not systemd.

Copper follows the same path, just without any systemd at all:

1. **Seat.** No logind, so a seat comes from `seatd` (already in noble). Hyprland
   needs `XDG_RUNTIME_DIR` and access to the seat; on the live/root session this
   is arranged the same way `startxfce.sh` does it.
2. **Session bus.** A D-Bus session bus is started (`dbus-launch` /
   `dbus-run-session`). Quickshell, portals, `swaync` and `hyprpolkitagent` all
   need it.
3. **Agent.** `hyprpolkitagent` provides the authentication agent (in place of
   polkitd/logind integration).
4. **Compositor.** `Hyprland` is launched with the ML4W config as
   `~/.config/hypr/hyprland.lua`, with a software-render fallback
   (`WLR_RENDERER=pixman` / `LIBGL_ALWAYS_SOFTWARE=1`) for machines with no GPU,
   matching how `startxfce.sh` falls back.
5. **Autostart.** ML4W's `autostart.lua` `exec-once`s the bar, notifications,
   wallpaper, idle daemon, etc. Units that ML4W expects systemd to run are either
   started here or dropped.

This logic lives in **`starthyprland`** on the Copper side (POSIX `sh`, modelled
on the 600-line `startxfce.sh`): find/claim the seat, set `XDG_RUNTIME_DIR`, bring
up the bus, pick the renderer, exec Hyprland, and return a meaningful exit code.

---

## 5. What we deliberately skip

| ML4W expects | Copper does instead |
|---|---|
| `systemd --user` units | `exec-once` from `autostart.lua` |
| display manager / `uwsm` login | start from the tty with `starthyprland` |
| `logind` seat management | `seatd` |
| `polkitd` | `hyprpolkitagent` |
| NetworkManager applet | wired-only Copper (the wifi stack was dropped, G4) |
| `elogind`/`mako`/etc. | `swaync` for notifications; `libelogind0` not needed |

---

## 6. Config layout (what will land in this repo)

The ML4W tree, scoped to the Hyprland session. Copied in from the pinned commit:

```
.config/
  hypr/                     hyprland.lua, input.lua, monitors.lua, gestures.lua,
                            workspaces.lua, hyprland-gui.lua, colors.lua,
                            functions.lua, conf/*.lua, assets/, effects/, scripts/
                            hypridle.conf hyprlock.conf hyprpaper.conf hyprsunset.conf
  ml4w/                     settings, listeners, scripts, themes, bin, tpl
  quickshell/               shell.qml, StatusbarApp, SidebarApp, WallpaperApp,
                            WelcomeApp, overview, shared, CustomTheme
  waybar/                   (alternative bar) modules.json, themes, launch.sh
  swaync/                   notifications
  walker/                   launcher
  wlogout/                  power menu
  rofi/                     menus
  kitty/                    terminal
  matugen/                  wallpaper-driven theming
  btop/ fastfetch/ qt6ct/ gtk-3.0/ gtk-4.0/ ohmyposh/ vim/
.gtkrc-2.0 .zshrc .bashrc
```

The exact subset is decided when porting starts. ML4W's own installer treats a
handful of these as user data to preserve across updates (`input.lua`,
`monitors.lua`, `gesture.lua`, `keybinding.lua`, `environment.lua`,
`layout.lua`, `windowrule.lua`, `animation.lua`, `decoration.lua`, `window.lua`,
`monitor.lua`, `hyprsunset.conf`, `hypridle.conf`, GTK config) — Copper will keep
that split in mind.

---

## 7. Provenance and pinning

- **Configuration:** ML4W OS **2.17**, commit
  `aeadb93395fa2e64a575161696fed978f0588f12`, **GPL-3.0**. Files copied from ML4W
  keep ML4W's copyright and license.
- **Hyprland and friends:** `ppa:cppiber/hyprland` for noble, pinned to the
  versions in §3.1.
- **Quickshell:** source build pinned to `4df562dfb2475a9057f0f33a8db75808efad8670`.

Nothing floats to `main`; every input is a commit or a version.

---

## 8. Copper-side changes (planned)

In `12hrformat/copperlinux`:

1. **Build stage.** A new stage (after the existing `gui` stage) that adds the
   cppiber PPA, computes the closure with
   `apt-get -o Dir::State::status=<empty> --download-only` the same way the XFCE
   stage does, `dpkg-deb -x` into the stage, and `cp -a -n` — stages re-assert
   ownership/modes on warm cache restores. A chroot gate like
   `Hyprland --version` proves it landed. Quickshell is built from source here.
2. **`starthyprland`.** New POSIX-`sh` session launcher in
   `iso/rootfs-overlay`, mirroring `startxfce.sh`; added to the `lint_scripts`
   list in `build.sh`.
3. **First-boot question.** `copper-firstboot.c` gains an `F_SESSION` field to the
   table (`XFCE` / `Hyprland`), and the answer is recorded (e.g. a
   `/etc/copper-desktop` marker).
4. **Config install.** The contents of this repo are vendored at a pinned commit
   and copied into the user's home on first boot (or into `/etc/skel`).
5. **ingot packages.** Heavier/optional apps are published as ingot packages in
   `copper-ingot-repo` and fetched only when Hyprland is chosen.
6. **Docs/tests.** `README.md`, `HANDOFF.md`, `futureplans.md` and
   `tests/branch-gate.sh` updated; CI kept green.

---

## 9. Risks / open questions

- **Quickshell build on noble.** ML4W's own Ubuntu support covers noble, so it is
  expected to build, but the Qt6 version noble ships is older than the QML
  modules Quickshell may want. First thing to prove.
- **`uwsm`.** It is normally a systemd user-session manager; its usefulness
  without systemd is unclear, so it may be dropped.
- **Lua in Hyprland.** Confirm whether Hyprland 0.56.2 needs a system Lua
  (`lua5.4`) or embeds one — ML4W lists `lua5.4`, so the ISO carries it either way.
- **GPU-less machines.** Software rendering must be verified in a VM.
- **GPL-3.0.** Redistributing the ML4W config keeps ML4W's license; the Copper
  repo's own license must remain compatible.

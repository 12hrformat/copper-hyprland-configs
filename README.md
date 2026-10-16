# copper-hyprland-configs

Hyprland desktop configuration for **[Copper Linux](https://github.com/12hrformat/copperlinux)**,
derived from the [ML4W OS](https://ml4w.com/os/) ("My Linux For Work") dotfiles.

Copper ships XFCE. This repo is the configuration for the *second*, optional
session: **Hyprland**. The distro bits live in `copperlinux`; the desktop's
look-and-feel lives here, versioned on its own.

> **Status: in progress.** The ML4W 2.17 dotfiles tree is in this repo under
> [`dotfiles/`](dotfiles/) at the pinned commit, with the taken/excluded split
> recorded in [`PROVENANCE.md`](PROVENANCE.md). The `hypr` build stage in
> [`12hrformat/copperlinux`](https://github.com/12hrformat/copperlinux) fetches
> this tree at the pinned commit, checks its sha256, and installs it to
> `/usr/share/copper/hyprland-skel`, and `starthyprland` seeds it into the
> user's home. **[`hyprland.md`](hyprland.md)** is the design of the session —
> how it works, the packages it needs, and how it hooks into Copper. The
> session is built and gated; work continues to polish booting.

## What to work on & where to start

- **Understand the design:** Read [hyprland.md](hyprland.md) first.
- **Explore the config:** [`dotfiles/`](dotfiles/) is the actual config seeded into user homes.
- **Config package:** `starthyprland` asks about "your Hyprland config files" (Y/n, default y). On y it runs `ingot install hypr-configs` to refresh the shipped skel; the package is published in `12hrformat/copper-ingot-repo`. The package payload is this config tree.
- **Contribute:** Open config-specific issues/PRs here; integration issues go to `copperlinux`.

## The idea

On first boot Copper's wizard asks a handful of questions. It will gain one more:
**XFCE or Hyprland?**

- **XFCE** — already in the ISO. Nothing to do.
- **Hyprland** — the compositor and its essentials are already baked into the
  ISO; the rest is fetched from Copper's own package manager, **ingot**. The
  session is then started from the tty with `starthyprland` (the Hyprland twin of
  `startxfce`).

## Where the pieces live

| piece | repo |
|---|---|
| distro, build pipeline, first-boot wizard, `starthyprland` | [`12hrformat/copperlinux`](https://github.com/12hrformat/copperlinux) |
| Hyprland + ML4W **configuration** (this repo) | `12hrformat/copper-hyprland-configs` |
| extra binary packages fetched at runtime | `12hrformat/copper-ingot-repo` |

Read **[hyprland.md](hyprland.md)** for the whole picture — the session flow, the
package list, and the parts of ML4W that assume systemd and therefore need
adapting.

## Why ML4W needs a PPA

ML4W OS 2.17 uses Hyprland's **Lua** config format, which landed in Hyprland
**0.55**. Copper's desktop closure is built from Ubuntu 24.04 "noble", which has
no Hyprland at all. The `ppa:cppiber/hyprland` PPA — the same one ML4W's own
Ubuntu setup uses — **publishes Hyprland 0.56.2 for noble**, so the binaries are
built against exactly the glibc Copper already stages. Details in
[hyprland.md §2](hyprland.md#2-why-ml4w-needs-hyprland-055).

## Provenance

The configuration is derived from **ML4W OS 2.17** by Stephan Raabe —
[`mylinuxforwork/dotfiles`](https://github.com/mylinuxforwork/dotfiles) — tracked
at commit `aeadb93395fa2e64a575161696fed978f0588f12`.

ML4W's dotfiles are licensed **GPL-3.0**. ML4W is entirely the work of its
authors; this repo only carries the Hyprland/ML4W configuration for Copper plus
the notes needed to run it without systemd. Any file copied from ML4W stays under
its original license and copyright.

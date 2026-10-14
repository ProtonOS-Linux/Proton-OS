# Branding and look

Files that make the system look like Proton OS. Full steps: [BUILD.md sections 6 and 8](../BUILD.md#6-look-and-feel-cubic-terminal).

| File here | Goes to | Notes |
|---|---|---|
| [`kcm-about-distrorc`](kcm-about-distrorc) | `/etc/xdg/kcm-about-distrorc` | About This System: Proton logo, protonos.org, 1.0 Core |
| [`os-release`](os-release) | `/usr/lib/os-release` | Divert Debian's copy first (`dpkg-divert --local --rename --divert /usr/lib/os-release.distrib --add /usr/lib/os-release`). `ID=debian` and the version/codename stay Debian's. `NAME="Proton OS"` gives the EFI folder `proton`. Remove the diversion before any Debian 14 upgrade. Cubic rewrites `PRETTY_NAME`: fix it on the host right before Generate. |
| [`kdeglobals`](kdeglobals) | `/etc/xdg/kdeglobals` | Win11-red-dark icons system-wide (skel's `kdeglobals` gets the same `[Icons]` section) |
| [`proton-exclude-wallpapers`](proton-exclude-wallpapers) | `/etc/dpkg/dpkg.cfg.d/` | Keeps KDE's Nuvole ("Next") wallpaper from coming back with updates |

## Wallpapers

Three KDE wallpaper packages in `/usr/share/wallpapers/`: `Proton-Slate` (default), `Proton-Magenta`, `Proton-Paper`. Each has `contents/images/3840x2160.png` and a `metadata.json` naming it "Proton Business Slate/Magenta/Paper". Source images: `assets/wallpapers/` (see BUILD.md Appendix A).

## Global theme defaults (diverted)

`/usr/share/plasma/look-and-feel/org.kde.breeze.desktop/contents/defaults` is diverted and rewritten with:

```ini
Image=Proton-Slate
Theme=Win11-red-dark
cursorTheme=Win10OS-cursors
```

Breeze Dark and Breeze Twilight get `Image=Proton-Slate` only. These files outrank `/etc/xdg`, which is why they must change.

## Icons and cursors

- Icons: [Win11-icon-theme](https://github.com/yeyushengfan258/Win11-icon-theme), red-dark variant (its installer's `-t green` installs red). The theme's `start-here*` icons (an Apple logo) are deleted.
- Cursors: [Win10OS-cursors](https://github.com/yeyushengfan258/Win10OS-cursors), system-wide in `/usr/share/icons/Win10OS-cursors`.

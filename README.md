# Proton-OS

[![Download Proton OS](https://img.shields.io/sourceforge/dt/proton-os.svg)](https://sourceforge.net/projects/proton-os/)
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](LICENSE)
[![Debian 13](https://img.shields.io/badge/Debian-13%20Trixie-red.svg)](https://www.debian.org/)

🌐 **Website:** [protonos.org](https://protonos.org) &nbsp;|&nbsp; ⬇️ **Download:** [SourceForge](https://sourceforge.net/projects/proton-os/)

A secure, Windows-familiar Linux distribution built for small business operations, zero learning curve, instant productivity, no telemetry, secure and stable out of the box.

> **The zero-friction, high-performance Linux distribution designed for small businesses transitioning from Windows.**

Proton OS is a lightweight, secure, and intuitive operating system built on top of **Debian 13 (Trixie)**. Designed specifically for small office environments, non-technical teams, modern and legacy hardware, Proton OS delivers a familiar desktop experience with zero learning curve and zero downtime.

---

## Screenshots

<p align="center">
  <img src="https://a.fsdn.com/con/app/proj/proton-os/screenshots/Monitor%20Promotion%203-c5e3e05c.png/max/600/1" width="45%" alt="Proton OS Desktop">
  <img src="https://a.fsdn.com/con/app/proj/proton-os/screenshots/web1-de7361a6.png/max/600/1" width="45%" alt="Familiar Desktop Layout">
</p>
<p align="center">
  <img src="https://a.fsdn.com/con/app/proj/proton-os/screenshots/web8-f8f35860.png/max/600/1" width="45%" alt="Dark Mode">
  <img src="https://a.fsdn.com/con/app/proj/proton-os/screenshots/Screenshot%202026-07-24%20120357-1c7d7036.png/max/600/1" width="45%" alt="Office Apps Ready to Go">
</p>

More screenshots on [SourceForge](https://sourceforge.net/projects/proton-os/).

---
## Key Features for Business

* **Familiar Desktop Interface:** Intuitive layout, Start-style application launcher, and system panel designed to mirror traditional Windows workflows.
* **Instant Productivity:** Pre-configured out of the box with web-app integration for Microsoft 365, Google Workspace, cloud storage, and PDF tools.
* **Debian 13 Stability:** Rock-solid upstream core with low system overhead and zero forced bloatware or forced snap packages.
* **Built-in Security:** Security controls enable by default, automated background security patch update, and built-in firewall configurations.
* **Hardware Friendly:** Extends the lifecycle of existing office PCs, significantly lowering hardware refresh costs.
* **Windows App Compatibility:** Run real Windows applications seamlessly alongside native Linux apps via Winboat's Docker + KVM integration — no dual-boot or full VM required.

---

## Pre-installed Software

Proton OS ships ready to use out of the box:

- **Brave Browser** — privacy-focused browser, installed via Brave's official repo
- **ONLYOFFICE Desktop Editors** — full office suite with strong Microsoft Office format compatibility
- **LibreOffice** (full suite: Writer, Calc, Impress, Draw, Math, Base) — included by default with Debian 13
- **Variety** — automatic wallpaper rotation
- **Winboat** — run Windows applications directly on Proton OS via Docker + KVM virtualization

See `software/` for exact install commands and Winboat's prerequisites.

---

## Repository Structure

```text
Proton-OS/
├── sddm/                # Login screen fixes (avatar, theme) - login.defs, kde_settings.conf
├── grub/                # Boot menu configs - grub.cfg, isolinux.cfg
├── calamares/           # Installer shellprocess scripts
├── plymouth/            # Boot splash config - plymouthd.conf
├── skel/                # Default user config templates (/etc/skel overlay)
├── packages/            # Package holds and apt install/purge commands
├── software/            # Pre-installed app install methods (Brave, ONLYOFFICE, Winboat, etc.)
├── security/            # Verified security defaults (AppArmor, UFW, unattended-upgrades)
├── BUILD.md             # Step-by-step Cubic build procedure
├── LICENSE              # GNU General Public License v3.0
└── README.md            # Repository overview

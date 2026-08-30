# Proton-OS
A secure, Windows-familiar Linux distribution built for small business operations, zero learning curve, instant productivity, no telemetry, secure and stable out of the box.

> **The zero-friction, high-performance Linux distribution designed for small businesses transitioning from Windows.**

Proton OS is a lightweight, secure, and intuitive operating system built on top of **Debian 13 (Trixie)**. Designed specifically for small office environments, non-technical teams, modern and legacy hardware, Proton OS delivers a familiar desktop experience with zero learning curve and zero downtime.

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

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

---

## Repository Structure

```text
proton-os/
├── build/               # ISO creation recipes and live-build configurations
├── config/              # Pre-configured desktop, panel, and theme defaults (/etc overlay)
├── package-lists/       # Curated .deb package lists for small business workstations
├── docs/                # Migration guides, deployment notes, and client documentation
├── LICENSE              # GNU General Public License v3.0
└── README.md            # Repository overview

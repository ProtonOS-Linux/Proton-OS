# Contributing to Proton OS

Thanks for your interest in improving Proton OS! Contributions of all kinds are welcome, including:

- **Hardware compatibility reports** — especially legacy BIOS-only systems, different GPU vendors, and unusual motherboard/USB controller combinations (see `security/` and `BUILD.md` for context on known quirks).
- **Bug reports** — open an issue describing the problem, your hardware, and steps to reproduce.
- **Documentation improvements** — corrections, clarifications, or missing setup steps in any of the files under this repo.
- **Feature suggestions** — especially around fleet management / remote administration tooling for small business IT teams.

## Reporting Hardware Compatibility

If Proton OS works (or doesn't) on your hardware, please open an issue with:
- CPU/motherboard model
- GPU (integrated or discrete, vendor)
- BIOS/UEFI mode used (Legacy/CSM or pure UEFI)
- Whether boot succeeded, and any error messages seen (e.g. `grub>` shell, black screen)

This directly helps expand the verified hardware list.

## Reporting Bugs

Please include:
- What you expected to happen vs. what actually happened
- Steps to reproduce
- Relevant output (`journalctl`, error dialogs, etc.) if available

## Documentation Changes

Config files and verification docs under `sddm/`, `grub/`, `calamares/`, `plymouth/`, `skel/`, `packages/`, `software/`, and `security/` reflect what has actually been verified on a real build — if you find something inaccurate or outdated, a pull request or issue is appreciated.

## Code of Conduct

Be respectful and constructive. This project serves small businesses and non-technical users — contributions that keep that audience in mind are especially valued.

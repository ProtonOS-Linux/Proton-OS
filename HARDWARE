# Hardware Compatibility

Known results from testing Proton OS on real hardware. Contributions
welcome — see CONTRIBUTING.md for what to include in a report.

## Known Working

| Hardware | Notes |
|---|---|
| Generic UEFI systems (most modern PCs, 2015+) | Boots and installs normally |

## Known Issues & Workarounds

### GRUB drops to `grub>` shell on boot (USB 3.x / xHCI controllers)

**Symptom:** On some motherboards (observed on a gaming-focused board),
GRUB fails with "unknown filesystem" errors across every partition on
the boot USB, even though the same USB boots fine on other machines.

**Cause:** GRUB's native USB 3.x/xHCI driver is incompatible with that
board's specific USB controller.

**Fix:** Enable CSM (Compatibility Support Module) in BIOS/UEFI
settings. This routes USB reads through legacy USB handling instead
of pure UEFI xHCI. This is a BIOS-side setting on the affected
machine — not something fixable in the Proton OS build itself.

## Not Yet Tested

- Legacy BIOS-only hardware (no UEFI)
- Broader range of GPU vendors (currently tested primarily on NVIDIA 1080ti Pascal)
- Older/lower-spec hardware for Winboat's virtualization requirements
  (KVM support required)

## Reporting Your Results

Open an issue with your CPU/motherboard, GPU, BIOS mode (Legacy/UEFI),
and outcome (see CONTRIBUTING.md for the full template). Both
successes and failures are useful data.

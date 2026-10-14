# Calamares changes in Proton OS 1.0 Core

Calamares 3.3.14 with `calamares-settings-debian`. Full steps: [BUILD.md section 9](../BUILD.md#9-installer-calamares-cubic-terminal).

| File in the image | Change |
|---|---|
| `/etc/calamares/modules/users.conf` | `libpwquality` block removed; `passwordRequirements:` → `minLength: 8` (`nonempty` is ignored by 3.3.14) |
| `/etc/calamares/modules/bootloader.conf` | `efiBootloaderId: "proton"` (matches GRUB's folder from os-release `NAME`) |
| `/etc/calamares/modules/finished.conf` | `restartNowCommand: "/usr/local/sbin/proton-reboot"` |
| `/usr/local/sbin/proton-reboot` | [`scripts/proton-reboot`](scripts/proton-reboot): Proton splash, then forced reboot |
| `/usr/share/calamares/helpers/calamares-sources-final` | backports lines removed; ` main non-free-firmware` → ` main contrib non-free-firmware` |
| `/etc/calamares/modules/packagechooser-thirdparty.conf` | [`modules/packagechooser-thirdparty.conf`](modules/packagechooser-thirdparty.conf): "Third-party software" page |
| `/etc/calamares/modules/contextualprocess-thirdparty.conf` | [`modules/contextualprocess-thirdparty.conf`](modules/contextualprocess-thirdparty.conf): leaves the fonts marker |
| `/usr/local/sbin/proton-msfonts` | [`scripts/proton-msfonts`](scripts/proton-msfonts): installs the fonts once online |
| `/etc/systemd/system/proton-msfonts.{service,timer}` | [`systemd/`](systemd/): runs 2 min after boot, then hourly until done (`systemctl enable proton-msfonts.timer`) |
| `/usr/share/pixmaps/proton-thirdparty.png` | logo, 200×200 on a transparent 960×600 canvas |
| `/etc/calamares/modules/shellprocess_removeautologin.conf` | [`shellprocess_removeautologin.conf`](shellprocess_removeautologin.conf): removes live autologin and installer shortcuts surgically |

## settings.conf

```yaml
sequence:
  - show:
    # ...
    - users
    - packagechooser@thirdparty
    # ...
  - exec:
    # ...
    - sources-final
    - contextualprocess@thirdparty
    # ...

instances:
- id: "thirdparty"
  module: "packagechooser"
  config: "packagechooser-thirdparty.conf"
- id: "thirdparty"
  module: "contextualprocess"
  config: "contextualprocess-thirdparty.conf"
```

Validate after every edit (a YAML error stops the installer from starting):

```bash
python3 -c "import yaml; yaml.safe_load(open('/etc/calamares/settings.conf')); print('YAML OK')"
```

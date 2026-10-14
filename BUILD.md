# Building Proton OS 1.0 Core from scratch

This guide turns a **stock Debian 13 (Trixie) KDE live ISO** into **Proton OS 1.0 Core** with Cubic. It covers every change in 1.0 Core: packages, apps, security, the Windows-familiar desktop (taskbar, Start menu, favorites), look and feel, branding, the installer and the boot menu.

**How to read this guide**

- **Cubic terminal** = the chroot terminal inside Cubic (`root@cubic`). **Host terminal** = a normal Konsole on the build machine. Each step says which one.
- Run **one command at a time** and check the output before moving on. Run host `sudo` commands one per paste: a multi-line paste can be swallowed by the sudo password prompt.
- Before any `sed -i`, run the same command with `sed -n …p` (dry run) and check that it changes only what you expect.
- Write files with one-line `printf '%s\n' 'line' 'line' > file`. Multi-line heredoc pastes get scrambled in the Cubic terminal.
- `$PROJECT` below means your Cubic project folder (for example `/home/thrices/cubic-project`). Quote it if the name has spaces.

---

## 0. What you need

| Item | Where |
|---|---|
| Debian 13 KDE live ISO | `debian-live-13.*-amd64-kde.iso` from [debian.org](https://www.debian.org/CD/live/) |
| Cubic | `sudo apt install cubic` (Cubic PPA/package for your host) |
| This repository | `git clone https://github.com/ProtonOS-Linux/Proton-OS.git ~/Proton-OS` |
| Proton assets | the folders listed below, in this repo |
| ImageMagick (host) | `sudo apt install imagemagick` |

### Proton files this guide copies from the repo

Most of the look and layout is stored as ready-made files in this repo, so a new build copies them instead of recreating them by hand:

| Repo path | Goes to | What it is |
|---|---|---|
| `assets/wallpapers/proton-os-business-{slate,magenta,paper}-4k.png` | `/usr/share/wallpapers/Proton-*` | The three Proton wallpapers (3840×2160) |
| `assets/pixmaps/proton-start.png` | `/usr/share/pixmaps/` | Proton logo (Start button, About page, installer) |
| `plymouth/proton-splash/` | `/usr/share/plymouth/themes/proton-splash/` | Boot splash theme |
| `calamares/branding/proton/` | `/etc/calamares/branding/proton/` | Installer branding and slideshow |
| `skel/` | `/etc/skel/` | Default user settings (panel, Start menu, favorites, shortcuts, effects…) |
| `grub/` | ISO `boot/grub/` and `isolinux/` | Live USB boot menus and GRUB theme |

> **First time only:** if these folders aren't in the repo yet, export them from the current Core Cubic project first. See [Appendix A](#appendix-a-export-the-current-core-files-into-the-repo).

---

## 1. Create the Cubic project (host)

1. Start Cubic, create a new project folder and select the Debian 13 KDE live ISO.
2. On the project page set:
   - **Name / Release:** `Proton OS` / `1.0 Core`
   - **Volume ID:** `PROTON_OS`. Whatever you choose, it must match the `search --label` line in `grub.cfg` and `isolinux.cfg` (section 14). Never change one without the other.
   - **Filename:** `ProtonOS-Core-1.0-64bit-YYYY.MM.DD.iso`
3. Continue to the terminal page. Everything up to section 13 runs in the **Cubic terminal**.

---

## 2. Apt sources and package holds (Cubic terminal)

Core uses plain Debian 13 sources: **no backports** (backports are only for Frontier). The build and the installed system must use the same sources, or Discover shows updates the ISO never had.

Write the sources (`main contrib non-free non-free-firmware`; `contrib` is needed for the Microsoft fonts installer):

```bash
printf '%s\n' 'Types: deb' 'URIs: http://deb.debian.org/debian' 'Suites: trixie trixie-updates' 'Components: main contrib non-free non-free-firmware' 'Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg' '' 'Types: deb' 'URIs: http://security.debian.org/debian-security' 'Suites: trixie-security' 'Components: main contrib non-free non-free-firmware' 'Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg' > /etc/apt/sources.list.d/debian.sources
```

If `/etc/apt/sources.list` exists, empty it so lines aren't duplicated: `: > /etc/apt/sources.list`. Make sure no other repositories are present except Brave's (added in section 4): `ls /etc/apt/sources.list.d/`.

Set the holds. **Holds don't survive between Cubic sessions, so redo this at the start of every session:**

```bash
apt-mark hold grub-efi-amd64-signed shim-signed
apt-mark showhold
```

Update the system (this also brings the current Debian 13 kernel):

```bash
apt update
apt full-upgrade -y
```

Remove any old kernel left behind (keep only the newest `linux-image-*`):

```bash
dpkg -l 'linux-image-[0-9]*' | grep ^ii
apt purge -y linux-image-<old-version>-amd64
```

---

## 3. Remove what Proton OS doesn't ship (Cubic terminal)

Dry run every purge first with `apt-get -s purge …` and check that nothing important is listed.

**Office suites and apps replaced in 1.0:**

```bash
apt purge -y 'libreoffice*' variety
```

**Network-facing GNOME leftovers** (present when the KDE image carries GNOME pieces):

```bash
apt purge -y gnome-remote-desktop gnome-user-share rygel rygel-playbin rygel-tracker libfreerdp-server3-3
```

**Developer tools, extra fonts and data not needed by office users:**

```bash
apt purge -y golang-1.24 golang-go golang-src buildah gcc-14 g++-14 build-essential fonts-noto-extra fonts-noto-cjk-extra fonts-noto-ui-extra ibus-data gnome-user-docs gnome-backgrounds pocketsphinx-en-us
```

**Login screen theme duplicates:** Proton keeps only Breeze.

```bash
apt purge -y sddm-theme-debian-breeze
rm -rf /usr/share/sddm/themes/debian-theme
ls /usr/share/sddm/themes/
```

Expected: only `breeze`.

**Docker** must not be installed (WinBoat uses Podman). If present:

```bash
apt purge -y docker.io docker-ce docker-ce-cli containerd containerd.io docker-compose-plugin
rm -rf /etc/docker /var/lib/docker
groupdel docker
```

**Clean up.** Mark the packages that must stay before autoremove:

```bash
apt-mark manual firefox-esr fuse-overlayfs git bup libdnnl3.6 fonts-noto-core fonts-dejavu-mono
apt-get -s autoremove
apt autoremove --purge -y
dpkg -l | awk '/^rc/{print $2}' | xargs -r dpkg --purge
```

Kept on purpose:
- `libdnnl3.6`: removing it also removes `gstreamer1.0-plugins-bad` and `gnome-snapshot`.
- `git` and `bup`: Kup backup needs them.
- `fonts-noto-core` and `fonts-dejavu-mono`: the boot splash uses them.
- File Roller is the archive manager (not Ark).

**Locales:** generate English, then drop the all-locales package (about 200 MB):

```bash
sed -i 's/^# *en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen
locale-gen
ls /usr/lib/locale/locale-archive
apt purge -y locales-all
```

---

## 4. Install the Proton OS apps (Cubic terminal)

### Brave browser (official repo)

```bash
apt install -y curl
curl -fsSLo /usr/share/keyrings/brave-browser-archive-keyring.gpg https://brave-browser-apt-release.s3.brave.com/brave-browser-archive-keyring.gpg
curl -fsSLo /etc/apt/sources.list.d/brave-browser-release.sources https://brave-browser-apt-release.s3.brave.com/brave-browser.sources
apt update
apt install -y brave-origin
```

### Firefox ESR

Comes with Debian; kept for the web apps (`apt-mark manual firefox-esr` in section 3).

### Collabora Office (Flathub, system-wide)

```bash
apt install -y flatpak
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
flatpak install -y --system flathub com.collabora.Office
flatpak config --set languages '*all*'
flatpak uninstall -y --unused
flatpak list --app --columns=application,runtime
```

Expected: Collabora as the only Flatpak app. Setting `languages` to `*all*` keeps the live and installed systems in agreement, so Discover doesn't offer language updates.

### WinBoat with Podman

```bash
apt install -y podman podman-compose fuse-overlayfs freerdp3-x11
```

Get the latest WinBoat `.deb` (1.0 Core ships 0.9.2):

```bash
curl -s https://api.github.com/repos/winboat-org/winboat/releases/latest | grep -e '"tag_name"' -e 'browser_download_url.*\.deb"'
curl -L -o /tmp/winboat.deb https://github.com/winboat-org/winboat/releases/download/v0.9.2/winboat-0.9.2-amd64.deb
apt install -y /tmp/winboat.deb && rm /tmp/winboat.deb
apt-mark manual freerdp3-x11
podman --version; podman-compose --version; xfreerdp3 --version | head -1
```

Hide the FreeRDP entry from Start → Internet (WinBoat runs it in the background). The override in `/usr/local` survives package updates:

```bash
mkdir -p /usr/local/share/applications && sed '/^\[Desktop Entry\]$/a NoDisplay=true' /usr/share/applications/xfreerdp3.desktop > /usr/local/share/applications/xfreerdp3.desktop
```

> WinBoat's setup wizard always starts on Docker. Users pick **Podman** from the dropdown. This is documented in the README; the app isn't patched.

### earlyoom (memory safety switch)

```bash
apt install -y earlyoom
cat /etc/default/earlyoom
```

Expected default: `EARLYOOM_ARGS="-r 3600"`. Add the list of desktop processes it must never close (no quotes or spaces inside the list; systemd passes it word by word):

```bash
sed -i 's#^EARLYOOM_ARGS="-r 3600"$#EARLYOOM_ARGS="-r 3600 --avoid ^(kwin.*|plasmashell|ksmserver|Xwayland|Xorg|sddm.*|systemd.*|dbus-daemon|dbus-broker|pipewire.*|wireplumber)$"#' /etc/default/earlyoom
grep '^EARLYOOM_ARGS=' /etc/default/earlyoom; systemctl is-enabled earlyoom
```

Don't test-run earlyoom in Cubic: the chroot shares the host's processes. Check it on the live USB.

### Other tools

```bash
apt install -y file-roller kup-backup sqlite3 imagemagick
```

### Media codecs

Check the GStreamer plugins are present (they are on the Debian KDE image):

```bash
dpkg -l gstreamer1.0-plugins-good gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly gstreamer1.0-libav | grep ^ii
```

---

## 5. Security defaults (Cubic terminal)

### Firewall (UFW): deny incoming, allow outgoing

```bash
apt install -y ufw
ufw default deny incoming
ufw default allow outgoing
sed -i 's/^ENABLED=no/ENABLED=yes/' /etc/ufw/ufw.conf
systemctl enable ufw
grep ENABLED /etc/ufw/ufw.conf
```

`ufw enable` doesn't work inside the chroot. `ENABLED=yes` plus the enabled service turns it on at first boot. Verify on the live USB with `sudo ufw status verbose`.

### AppArmor

```bash
apt install -y apparmor
systemctl is-enabled apparmor
```

Proton OS uses the profiles that packages ship. `apparmor-profiles` isn't installed.

### Automatic security updates

```bash
apt install -y unattended-upgrades
printf '%s\n' 'APT::Periodic::Update-Package-Lists "1";' 'APT::Periodic::Unattended-Upgrade "1";' > /etc/apt/apt.conf.d/20auto-upgrades
```

Debian's default `50unattended-upgrades` only installs from Debian security origins, so leave it as is. KDE Discover's own update settings are in section 7.

### Home folder permissions

So the login screen can read each user's avatar (`~/.face`):

```bash
sed -i 's/^HOME_MODE.*/HOME_MODE\t0751/' /etc/login.defs
grep ^HOME_MODE /etc/login.defs
```

Reference copy: [`sddm/login.defs`](sddm/login.defs).

### Secure Boot

Keep the Microsoft-signed boot chain: `shim-signed` and `grub-efi-amd64-signed` are installed and held (section 2). Don't replace `BOOTX64.EFI` with an unsigned GRUB.

---

## 6. Look and feel (Cubic terminal)

### Copy the Proton assets into the chroot (host terminal)

```bash
sudo mkdir -p "$PROJECT/custom-root/root/proton"
```
```bash
sudo cp -r ~/Proton-OS/assets ~/Proton-OS/plymouth ~/Proton-OS/calamares ~/Proton-OS/skel "$PROJECT/custom-root/root/proton/"
```

Back in the **Cubic terminal**, `/root/proton/` now holds the repo files. Remove it in section 12.

### Proton wallpapers

Three KDE wallpaper packages; Slate is the default everywhere:

```bash
for c in slate magenta paper; do mkdir -p /usr/share/wallpapers/Proton-${c^}/contents/images && cp /root/proton/assets/wallpapers/proton-os-business-$c-4k.png /usr/share/wallpapers/Proton-${c^}/contents/images/3840x2160.png; done
```
```bash
for c in slate magenta paper; do printf '{\n  "KPlugin": {\n    "Authors": [ { "Name": "Proton OS" } ],\n    "Id": "Proton-%s",\n    "Name": "Proton Business %s"\n  }\n}\n' "${c^}" "${c^}" > /usr/share/wallpapers/Proton-${c^}/metadata.json; done
```

Keep KDE's "Next" (Nuvole) wallpaper out for good. `breeze-wallpaper` can't be removed (it takes `plasma-workspace` with it), so tell dpkg not to install its files:

```bash
printf '# Proton OS: do not install KDE default "Next" (Nuvole) wallpaper\npath-exclude=/usr/share/wallpapers/Next/*\n' > /etc/dpkg/dpkg.cfg.d/proton-exclude-wallpapers
rm -r /usr/share/wallpapers/Next
ls /usr/share/wallpapers/
```

Remove any other stock wallpapers so the picker shows only the three Proton ones. Expected: `Proton-Magenta  Proton-Paper  Proton-Slate`.

### Proton logo

```bash
cp /root/proton/assets/pixmaps/proton-start.png /usr/share/pixmaps/
```

### Win10OS cursors

```bash
git clone --depth 1 https://github.com/yeyushengfan258/Win10OS-cursors.git /root/Win10OS-cursors
cd /root/Win10OS-cursors && ./install.sh
ls /usr/share/icons/Win10OS-cursors/cursors | wc -l; grep ^Name= /usr/share/icons/Win10OS-cursors/index.theme
cd /root && rm -r /root/Win10OS-cursors
```

Expected: 111 cursors and `Name=Win10OS Cursors`. Running as root installs system-wide to `/usr/share/icons`.

### Win11-red-dark icons

The installer's `-t` menu has the colors swapped: **`-t green` installs red**. Check before installing:

```bash
git clone --depth 1 https://github.com/yeyushengfan258/Win11-icon-theme.git /root/Win11-icon-theme
sed -n '/-t|--theme)/,/esac/p' /root/Win11-icon-theme/install.sh | grep -A1 -e 'red)' -e 'green)'
```

If `red)` is followed by `THEME_VARIANTS[3]`, the bug is still there; use `-t green`. If upstream fixed it, use `-t red`.

```bash
cd /root/Win11-icon-theme && ./install.sh -d /usr/share/icons -t green
ls -d /usr/share/icons/Win11*; grep -m1 '^Name=' /usr/share/icons/Win11-red-dark/index.theme
cd /root && rm -r /root/Win11-icon-theme
```

Expected: `Win11-red`, `Win11-red-dark` and `Name=Win11-red-dark`.

The theme's "system logo" icons (`start-here*`) are an Apple logo. Delete them so KDE uses the Proton logo:

```bash
find /usr/share/icons/Win11-red /usr/share/icons/Win11-red-dark -name 'start-here*' ! -name '*fedora*' ! -name '*manjaro*' ! -name '*pop-os*' -delete
gtk-update-icon-cache -f /usr/share/icons/Win11-red; gtk-update-icon-cache -f /usr/share/icons/Win11-red-dark
```

Set the icons system-wide:

```bash
printf '[Icons]\nTheme=Win11-red-dark\n' > /etc/xdg/kdeglobals
```

### Plasma defaults at the source

New users take their first wallpaper, icons and cursor from the global theme's `defaults` file, and that file outranks `/etc/xdg`. Divert the Breeze files so Debian updates can't put the stock values back:

```bash
f=/usr/share/plasma/look-and-feel/org.kde.breeze.desktop/contents/defaults
cat $f
dpkg-divert --local --rename --divert $f.distrib --add $f
sed -e 's/^Image=Next$/Image=Proton-Slate/' -e 's/^Theme=breeze$/Theme=Win11-red-dark/' -e 's/^cursorTheme=breeze_cursors$/cursorTheme=Win10OS-cursors/' $f.distrib > $f
```
```bash
for t in breezedark breezetwilight; do f=/usr/share/plasma/look-and-feel/org.kde.$t.desktop/contents/defaults; dpkg-divert --local --rename --divert $f.distrib --add $f && sed 's/^Image=Next$/Image=Proton-Slate/' $f.distrib > $f; done
```
```bash
dpkg-divert --list | grep look-and-feel
```

Expected: three diversions.

### Login screen (SDDM)

Slate background (the file must be INI style, with `[General]`):

```bash
printf '[General]\nbackground=/usr/share/wallpapers/Proton-Slate/contents/images/3840x2160.png\ntype=image\n' > /usr/share/sddm/themes/breeze/theme.conf.user
```

SDDM settings: Breeze theme, Win10OS cursor, NumLock on. Reference: [`sddm/kde_settings.conf`](sddm/kde_settings.conf). Set these in `/etc/sddm.conf.d/kde_settings.conf`:

```bash
grep -n -A4 -e '^\[Theme\]' -e '^\[General\]' /etc/sddm.conf.d/kde_settings.conf
sed -i 's/^CursorTheme=.*$/CursorTheme=Win10OS-cursors/' /etc/sddm.conf.d/kde_settings.conf
grep -q '^Numlock=' /etc/sddm.conf.d/kde_settings.conf && sed -i 's/^Numlock=.*/Numlock=on/' /etc/sddm.conf.d/kde_settings.conf || sed -i '/^\[General\]$/a Numlock=on' /etc/sddm.conf.d/kde_settings.conf
```

User avatar default:

```bash
cp /root/proton/skel/.face /etc/skel/.face && chmod 644 /etc/skel/.face
```

### Plasma splash after login: off

```bash
printf '[KSplash]\nEngine=none\nTheme=None\n' > /etc/xdg/ksplashrc
```

(Skel gets the same file in section 7.) Don't delete the Breeze splash files: updates would restore them.

### Boot splash (Plymouth)

```bash
cp -r /root/proton/plymouth/proton-splash /usr/share/plymouth/themes/
cp /root/proton/plymouth/plymouthd.conf /etc/plymouth/plymouthd.conf
ls /usr/share/plymouth/themes/proton-splash/
```

- `plymouthd.conf` is the authoritative setting (`Theme=proton-splash`, `ShowDelay=0`). Don't use `update-alternatives` for this.
- The `two-step` module needs six support images in the theme folder: `lock.png`, `entry.png`, `bullet.png`, `capslock.png`, `keyboard.png`, `keymap-render.png`. Check they're all there.
- In `proton-splash.plymouth`, `Font=` and `TitleFont=` use **Noto Sans** (not Cantarell), otherwise text shows as boxes.

The large title still showed boxes because of the `Plymouth*.ttf` font copies inside the initramfs. A fontconfig rule hides them, and an initramfs hook copies the rule in:

```bash
mkdir -p /usr/share/proton-os
cp /root/proton/plymouth/99-proton-plymouth.conf /usr/share/proton-os/
cp /root/proton/plymouth/proton-plymouth-fonts /etc/initramfs-tools/hooks/
chmod 755 /etc/initramfs-tools/hooks/proton-plymouth-fonts
update-initramfs -u -k all
for i in /boot/initrd.img-*; do echo "== $i"; lsinitramfs $i | grep -i -e 'fonts/conf' -e 'plymouth/themes/proton' | head; done
```

---

## 7. Default user settings: desktop, taskbar, Start menu, favorites (Cubic terminal)

Everything a new user starts with lives in `/etc/skel`. Copy the Proton skel from the repo, then check each setting below:

```bash
cp -a /root/proton/skel/. /etc/skel/
rm -rf /etc/skel/.config/kdedefaults
ls -A /etc/skel /etc/skel/.config
```

`kdedefaults` must not be in skel: it outranks the global theme defaults from section 6, and Plasma rebuilds it on its own.

What each file sets, and how to check it:

### Taskbar and Start menu: `plasma-org.kde.plasma.desktop-appletsrc`

- Full-width bottom panel (not floating), Windows-style layout: Start menu at the left, pinned apps, system tray and clock at the right.
- Start menu (Kickoff) button shows the Proton logo (`proton-start.png`).
- Desktop wallpaper: `Image=file:///usr/share/wallpapers/Proton-Slate/`.
- Taskbar pins: Brave is pinned as **`applications:com.brave.Origin.desktop`**, not `brave-origin.desktop`. On Wayland the Brave window identifies as `com.brave.Origin`, and the wrong pin opens a second icon.

```bash
grep -n -e '^Image=' -e '^launchers=' -e 'proton-start' /etc/skel/.config/plasma-org.kde.plasma.desktop-appletsrc
```

### Start menu favorites: `kickoffrc` and the activity database

The favorites come from the exported skel (Appendix A). Collabora Office must be in the list; **no LibreOffice, ONLYOFFICE or Variety entries.**

```bash
grep -n favorites /etc/skel/.config/kickoffrc /etc/skel/.config/kactivitymanagerd-statsrc 2>/dev/null
grep -ri -e libreoffice -e onlyoffice -e variety /etc/skel
```

Expected: no output from the second command.

Plasma also keeps favorites in an SQLite database at `/etc/skel/.local/share/kactivitymanagerd/resources/database`. For skel use, the activity IDs must be `:global` (not a specific activity UUID):

```bash
sqlite3 /etc/skel/.local/share/kactivitymanagerd/resources/database "SELECT usedActivity, targettedResource FROM ResourceLink;"
```

To fix an entry that shows a UUID:

```bash
sqlite3 /etc/skel/.local/share/kactivitymanagerd/resources/database "UPDATE ResourceLink SET usedActivity=':global' WHERE usedActivity!=':global';"
```

### Keyboard, mouse and cursor: `kcminputrc`

```ini
[Keyboard]
NumLock=0

[Mouse]
cursorTheme=Win10OS-cursors
```

`NumLock=0` means **on** (1 = off, 2 = unchanged). Touchpad natural scrolling isn't preset: Plasma stores it per touchpad model, so no single default works on every laptop.

```bash
grep -n -A1 -e '\[Keyboard\]' -e '\[Mouse\]' /etc/skel/.config/kcminputrc
```

### Icons: `kdeglobals`

skel's own `kdeglobals` always wins, so the icons are set here too:

```bash
grep -q '^\[Icons\]' /etc/skel/.config/kdeglobals || printf '\n[Icons]\nTheme=Win11-red-dark\n' >> /etc/skel/.config/kdeglobals
grep -n -A1 '^\[Icons\]' /etc/skel/.config/kdeglobals
```

### Desktop effects: `kwinrc`

Hide Cursor and Wobbly Windows on:

```bash
grep -n -e hidecursor -e wobbly /etc/skel/.config/kwinrc
```

Expected: `hidecursorEnabled=true` and `wobblywindowsEnabled=true` under `[Plugins]`. If missing:

```bash
sed -i -e '/^\[Plugins\]$/a hidecursorEnabled=true' -e '/^\[Plugins\]$/a wobblywindowsEnabled=true' /etc/skel/.config/kwinrc
```

### Lock screen: `kscreenlockerrc`

```bash
printf '[Greeter][Wallpaper][org.kde.image][General]\nImage=/usr/share/wallpapers/Proton-Slate/contents/images/3840x2160.png\n' > /etc/skel/.config/kscreenlockerrc
```

### Shortcuts: `kglobalshortcutsrc`

Ctrl+Shift+Esc opens System Monitor (like Task Manager), next to KDE's own Meta+Esc:

```bash
grep -q 'plasma-systemmonitor' /etc/skel/.config/kglobalshortcutsrc || printf '\n[services][org.kde.plasma-systemmonitor.desktop]\n_launch=Meta+Esc\\tCtrl+Shift+Esc\n' >> /etc/skel/.config/kglobalshortcutsrc
grep -n -A1 'plasma-systemmonitor' /etc/skel/.config/kglobalshortcutsrc
```

Expected: `_launch=Meta+Esc\tCtrl+Shift+Esc`.

### Splash after login: `ksplashrc`

```bash
printf '[KSplash]\nEngine=none\nTheme=None\n' > /etc/skel/.config/ksplashrc
```

### Software updates (Discover): `discoverrc` and `PlasmaDiscoverUpdates`

```bash
cat /etc/skel/.config/discoverrc /etc/skel/.config/PlasmaDiscoverUpdates
```

Expected:
- `discoverrc`: `[Software]` with `UseOfflineUpdates=true`
- `PlasmaDiscoverUpdates`: `[Global]` with `UseUnattendedUpdates=true` and `RequiredNotificationInterval=2592000` (a monthly check)

Don't ship `LastNotificationTime`: it's a per-user timestamp.

### Things that must not be in skel

```bash
ls /etc/skel/.config/autostart-scripts 2>/dev/null; ls /etc/skel/.config/variety 2>/dev/null; cat /etc/skel/.config/plasmarc 2>/dev/null
```

Expected: no output. No first-login wallpaper script (`set-wallpaper.sh`), no Variety folder, no `plasmarc` with a `usersWallpapers=` path.

---

## 8. Branding (Cubic terminal)

### About This System

```bash
printf '[General]\nLogoPath=/usr/share/pixmaps/proton-start.png\nWebsite=https://www.protonos.org/\nVersion=1.0\nVariant=Core\n' > /etc/xdg/kcm-about-distrorc
```

### os-release

Divert Debian's file so `base-files` updates can't restore it. `ID=debian`, `VERSION_ID` and `VERSION_CODENAME` stay Debian's, so apt and scripts still see Debian 13. Run the second command right after the first; the file is missing in between.

```bash
dpkg-divert --local --rename --divert /usr/lib/os-release.distrib --add /usr/lib/os-release
```
```bash
printf '%s\n' 'PRETTY_NAME="Proton OS 1.0"' 'NAME="Proton OS"' 'VERSION_ID="13"' 'VERSION="1.0 (based on Debian 13 trixie)"' 'VERSION_CODENAME=trixie' "DEBIAN_VERSION_FULL=$(. /usr/lib/os-release.distrib; echo $DEBIAN_VERSION_FULL)" 'ID=debian' 'LOGO=proton-start' 'HOME_URL="https://www.protonos.org/"' 'SUPPORT_URL="https://protonos.discourse.group/"' 'BUG_REPORT_URL="https://github.com/ProtonOS-Linux/Proton-OS/issues"' > /usr/lib/os-release
```
```bash
cat /etc/os-release; dpkg-divert --list | grep os-release; lsb_release -is
```

Expected: the Proton file, the diversion listed, and `Debian`.

> Before any future upgrade to Debian 14, remove this diversion first.

The `NAME` also decides the installed system's EFI folder: GRUB uses `NAME` lowercased and cut at the first space, which gives **`proton`**. The installer is set to match in section 9.

---

## 9. Installer: Calamares (Cubic terminal)

### Branding

```bash
cp -r /root/proton/calamares/branding/proton /etc/calamares/branding/
grep -n '^branding:' /etc/calamares/settings.conf
sed -i 's/^branding:.*/branding: proton/' /etc/calamares/settings.conf
```

The slideshow needs `show.qml` in the branding folder.

### Passwords: at least 8 characters

Calamares 3.3.14 only checks `minLength`, `maxLength` and `libpwquality`; `nonempty` is ignored. Remove the whole `libpwquality` block (it rejects ordinary passwords through dictionary checks) and set `minLength: 8`:

```bash
sed -n '/^passwordRequirements:/,/^[a-zA-Z]/p' /etc/calamares/modules/users.conf
```

Edit the block so it reads:

```yaml
passwordRequirements:
    nonempty: true
    minLength: 8
    maxLength: -1
```

Check with the `sed -n` line above.

### EFI boot entry: `proton`

```bash
sed -i 's/^# efiBootloaderId: "dirname"$/&\nefiBootloaderId: "proton"/' /etc/calamares/modules/bootloader.conf
grep -n '^efiBootloaderId' /etc/calamares/modules/bootloader.conf
```

Expected: `efiBootloaderId: "proton"`, which matches `NAME` in os-release, so the system keeps one `proton` EFI folder and one boot entry.

### Clean restart after install

`systemctl -i reboot` waits for every live service and hangs on the splash. This script shows the Proton splash, then forces the reboot (the new system is already unmounted at that point):

```bash
printf '%s\n' '#!/bin/sh' '# Proton OS: forced reboot after installation, keeping the boot splash on screen.' 'sync' 'timeout 5 plymouthd --mode=reboot --attach-to-session >/dev/null 2>&1' 'timeout 5 plymouth show-splash >/dev/null 2>&1' 'sleep 1' 'exec systemctl --force reboot' > /usr/local/sbin/proton-reboot && chmod 755 /usr/local/sbin/proton-reboot
sed -i 's|^restartNowCommand:.*|restartNowCommand: "/usr/local/sbin/proton-reboot"|' /etc/calamares/modules/finished.conf
grep -n restartNowCommand /etc/calamares/modules/finished.conf
```

### Installed system's apt sources

The installed system must use the same sources as the build: plain Debian 13 with `contrib`, **no backports**.

```bash
sed -i '/[Bb]ackports/d' /usr/share/calamares/helpers/calamares-sources-final
sed -i 's/ main non-free-firmware$/ main contrib non-free-firmware/' /usr/share/calamares/helpers/calamares-sources-final
grep -c backports /usr/share/calamares/helpers/calamares-sources-final; grep -c 'main contrib non-free-firmware' /usr/share/calamares/helpers/calamares-sources-final
```

Expected: `0` for backports, then a count above 0.

### Remove live-session leftovers after install

Proton's shellprocess removes autologin and the installer shortcuts from the installed system. It edits SDDM settings surgically (never a broad `rm -rf`, which once wiped all SDDM settings). Reference: [`calamares/shellprocess_removeautologin.conf`](calamares/shellprocess_removeautologin.conf).

```bash
cp /root/proton/calamares/shellprocess_removeautologin.conf /etc/calamares/modules/
grep -n removeautologin /etc/calamares/settings.conf
```

> On a stock Debian live ISO the live user is **`user`**, not `live`. Check with `grep -r -e 'LIVE_USERNAME' -e 'username' /etc/live/ /lib/live/config/ 2>/dev/null | head` and make the `userdel` line match.

Also check the "Install System" launcher isn't left in Start → System after install: the packages step should remove `calamares` and `calamares-settings-debian`:

```bash
grep -n -A8 '^operations' /etc/calamares/modules/packages.conf
```

### "Third-party software" page: Microsoft fonts

The fonts can't ship in the ISO (license), so the installer page only leaves a marker. A timer on the installed system downloads them once online (2 minutes after boot, then hourly until done).

Font installer script:

```bash
printf '%s\n' '#!/bin/sh' '# Proton OS: install Microsoft core fonts once online.' '# Runs only if the installer left the marker (option ticked in Calamares).' 'MARKER=/var/lib/proton/msfonts-pending' '[ -e "$MARKER" ] || exit 0' 'export DEBIAN_FRONTEND=noninteractive' 'echo "ttf-mscorefonts-installer msttcorefonts/accepted-mscorefonts-eula select true" | debconf-set-selections' 'if ! dpkg -s ttf-mscorefonts-installer >/dev/null 2>&1; then' '    apt-get -o DPkg::Lock::Timeout=300 update || exit 0' '    apt-get -o DPkg::Lock::Timeout=300 install -y ttf-mscorefonts-installer || exit 0' 'fi' 'fc-list | grep -qi "Arial" || /usr/lib/msttcorefonts/update-ms-fonts' 'fc-list | grep -qi "Arial" && rm -f "$MARKER"' 'exit 0' > /usr/local/sbin/proton-msfonts && chmod 755 /usr/local/sbin/proton-msfonts
```

Service and timer:

```bash
printf '%s\n' '[Unit]' 'Description=Proton OS - install Microsoft core fonts when online' 'ConditionPathExists=/var/lib/proton/msfonts-pending' 'Wants=network-online.target' 'After=network-online.target' '' '[Service]' 'Type=oneshot' 'ExecStart=/usr/local/sbin/proton-msfonts' 'TimeoutStartSec=15min' > /etc/systemd/system/proton-msfonts.service
```
```bash
printf '%s\n' '[Unit]' 'Description=Proton OS - retry Microsoft core fonts install until done' 'ConditionPathExists=/var/lib/proton/msfonts-pending' '' '[Timer]' 'OnBootSec=2min' 'OnUnitActiveSec=1h' '' '[Install]' 'WantedBy=timers.target' > /etc/systemd/system/proton-msfonts.timer
```
```bash
systemctl enable proton-msfonts.timer
```

Page image: the logo sits small on a transparent 960×600 canvas (Calamares stretches the image):

```bash
convert /usr/share/pixmaps/proton-start.png -resize 200x200 -background none -gravity center -extent 960x600 -define png:color-type=6 /usr/share/pixmaps/proton-thirdparty.png
```

Page config:

```bash
printf '%s\n' '---' 'mode: optional' 'method: legacy' 'labels:' '    step: "Third-party software"' 'default: msfonts' 'items:' '    - id: ""' '      name: "Do not install Microsoft fonts"' '      description: "Proton OS will use its built-in open fonts. Microsoft fonts can be added later."' '      screenshot: "/usr/share/pixmaps/proton-thirdparty.png"' '    - id: msfonts' '      name: "Microsoft fonts (recommended)"' '      description: "Installs the Microsoft core fonts (Arial, Times New Roman, Verdana, Courier New and more) so Word, Excel and PowerPoint files look as intended. The fonts are downloaded automatically after installation, as soon as the computer is online. Selecting this option accepts the Microsoft End User License Agreement for these fonts."' '      screenshot: "/usr/share/pixmaps/proton-thirdparty.png"' > /etc/calamares/modules/packagechooser-thirdparty.conf
```

Marker step (the `-` prefix makes errors non-fatal):

```bash
printf '%s\n' '---' 'dontChroot: false' 'timeout: 30' 'packagechooser_thirdparty:' '    msfonts:' '        - "-mkdir -p /var/lib/proton"' '        - "-touch /var/lib/proton/msfonts-pending"' > /etc/calamares/modules/contextualprocess-thirdparty.conf
```

Wire both into `settings.conf`: the page after the **first** `- users` (the show section), and the step after `- sources-final`. Back up and dry run first:

```bash
cp /etc/calamares/settings.conf /root/settings.conf.bak && tail -8 /etc/calamares/settings.conf
sed -e '0,/^  - users$/s//&\n  - packagechooser@thirdparty/' -e 's/^  - sources-final$/&\n  - contextualprocess@thirdparty/' /etc/calamares/settings.conf | grep -n -e '- users' -e packagechooser -e 'sources-final' -e contextualprocess
```
```bash
sed -i -e '0,/^  - users$/s//&\n  - packagechooser@thirdparty/' -e 's/^  - sources-final$/&\n  - contextualprocess@thirdparty/' /etc/calamares/settings.conf
```

If `instances:` already exists in `settings.conf`, add the two entries under it by hand. If it doesn't (check `tail` above), append:

```bash
printf '%s\n' 'instances:' '- id: "thirdparty"' '  module: "packagechooser"' '  config: "packagechooser-thirdparty.conf"' '- id: "thirdparty"' '  module: "contextualprocess"' '  config: "contextualprocess-thirdparty.conf"' >> /etc/calamares/settings.conf
```

### Validate every Calamares file you changed

A YAML error stops the installer from starting:

```bash
for f in settings modules/users modules/bootloader modules/finished modules/packagechooser-thirdparty modules/contextualprocess-thirdparty modules/shellprocess_removeautologin; do python3 -c "import yaml; yaml.safe_load(open('/etc/calamares/$f.conf')); print('$f: YAML OK')"; done
```

---

## 10. Polkit rules

Any polkit rule must use the JavaScript `.rules` format in `/etc/polkit-1/rules.d/`. The old `.pkla` format was dropped in polkit 126.

---

## 11. Final package check (Cubic terminal)

```bash
apt update && apt full-upgrade -y
flatpak update -y
apt-mark showhold
apt-get -s autoremove | grep -c '^Remv'; dpkg -l | grep -c '^rc'
docker --version 2>&1 | head -1; dpkg -l | grep -i -e libreoffice -e onlyoffice -e variety
```

Expected: the two holds, `0` and `0`, Docker not found, and no office leftovers.

---

## 12. Clean up the chroot (Cubic terminal)

```bash
rm -rf /root/proton /root/settings.conf.bak
apt clean
unset HISTFILE; history -c; rm -rf /root/.bash_history /root/.cache /root/.lesshst /root/.local
ls -A /root
```

Expected: only default dotfiles (`.bashrc`, `.profile`). Build history must not ship in the image.

> **If the live user's home is baked into the image** (as on the penguins-eggs-based builds, where `/home/live` exists in the chroot), re-sync it from skel after every skel change: `rsync -a --delete --chown=live:live /etc/skel/ /home/live/`, then `diff -rq /etc/skel /home/live` (no output). A stock Debian live ISO creates the live home from skel at boot, so this isn't needed there; check with `ls /home`.

Leave the Cubic terminal now. Don't come back to it after section 13.

---

## 13. Fix os-release right before Generate (host terminal)

Cubic rewrites `PRETTY_NAME` when you leave its terminal page. Go through Cubic's screens up to Generate, **don't click it yet**, and in a host terminal:

```bash
head -2 "$PROJECT/custom-root/usr/lib/os-release"
```
```bash
sudo sed -i 's|^PRETTY_NAME=.*|PRETTY_NAME="Proton OS 1.0"|' "$PROJECT/custom-root/usr/lib/os-release"
```
```bash
head -2 "$PROJECT/custom-root/usr/lib/os-release"
```

Expected: `PRETTY_NAME="Proton OS 1.0"`. Don't return to the terminal page after this.

---

## 14. Live USB boot menus (host terminal)

Both menus live in the ISO tree at `$PROJECT/custom-disk/`. **Always edit `boot/grub/grub.cfg` and `isolinux/isolinux.cfg` together.** Reference copies: [`grub/`](grub/).

Settings for both:

| | GRUB (UEFI) | isolinux (BIOS) |
|---|---|---|
| Timeout | `set timeout=5` | `TIMEOUT 50` (tenths of a second) |
| Entries | Start Proton OS, Start Proton OS - Safe Graphics | the same |
| Volume label | `search --no-floppy --set=root --label <Volume ID>` | — |
| Background | `/boot/grub/splash.png` (1920×1080) + Proton theme | `MENU BACKGROUND splash.png` (640×480) |

**Safe Graphics** entry: the normal entry's kernel line with `nomodeset` added and `splash` removed, in both files. After adding it, check `set default=` in GRUB still points at **Start Proton OS** (counting from 0).

**Font under Secure Boot:** signed GRUB refuses `loadfont /boot/grub/font.pf2` ("prohibited by secure boot policy") and the menu falls back to text with no background. Use the built-in font:

```bash
sudo sed -i 's|^if loadfont /boot/grub/font.pf2; then|if loadfont unicode; then|' "$PROJECT/custom-disk/boot/grub/grub.cfg"
grep -n loadfont "$PROJECT/custom-disk/boot/grub/grub.cfg"
```

**Backgrounds:**

```bash
mkdir -p ~/splash-new && convert ~/Proton-OS/assets/wallpapers/proton-os-business-slate-4k.png -resize 1920x1080 -strip -interlace none -define png:color-type=2 ~/splash-new/grub-splash.png && convert ~/Proton-OS/assets/wallpapers/proton-os-business-slate-4k.png -resize 640x480^ -gravity center -extent 640x480 -strip -interlace none -define png:color-type=2 ~/splash-new/isolinux-splash.png
```
```bash
sudo cp ~/splash-new/grub-splash.png "$PROJECT/custom-disk/boot/grub/splash.png"
```
```bash
sudo cp ~/splash-new/isolinux-splash.png "$PROJECT/custom-disk/isolinux/splash.png"
```

isolinux must use `UI vesamenu.c32` for the background to show.

**GRUB theme (no box around the entries):** copy the Proton theme from the repo and load it right after the background line:

```bash
sudo mkdir -p "$PROJECT/custom-disk/boot/grub/themes"
```
```bash
sudo cp -r ~/Proton-OS/grub/themes/proton "$PROJECT/custom-disk/boot/grub/themes/"
```
```bash
sudo sed -i '/^background_image \/boot\/grub\/splash.png$/a set theme=/boot/grub/themes/proton/theme.txt' "$PROJECT/custom-disk/boot/grub/grub.cfg"
grep -n -e '^background_image' -e '^set theme=' "$PROJECT/custom-disk/boot/grub/grub.cfg"; ls "$PROJECT/custom-disk/boot/grub/themes/proton/" | wc -l
```

Expected: the theme line right after the background, and 11 files (`theme.txt`, `background.png` and nine `select_*.png`).

**Secure Boot check:** `EFI/boot/bootx64.efi` in the ISO must be the Microsoft-signed shim, not an unsigned GRUB:

```bash
ls -la "$PROJECT/custom-disk/EFI/boot/"
```

---

## 15. Generate and check the ISO (Cubic)

1. Click **Generate** and let it finish. Don't interrupt it.
2. Check that the live folder has all three files (an interrupted Generate leaves only `vmlinuz`):

```bash
ls -la "$PROJECT/custom-disk/live/"
```

Expected: `vmlinuz`, `initrd` (named `initrd.img` or `initrd.xz`, matching what `grub.cfg` loads) and `filesystem.squashfs`.

3. Create the checksums for the README and SourceForge:

```bash
sha256sum ProtonOS-Core-1.0-64bit-*.iso
md5sum ProtonOS-Core-1.0-64bit-*.iso
```

---

## 16. Test checklist (live USB and a fresh install)

**Boot**
- [ ] UEFI menu: Slate background, no box, highlight bar, countdown; works with Secure Boot on
- [ ] BIOS menu: Slate background, readable
- [ ] Safe Graphics entry boots
- [ ] Proton splash with readable text (no boxes)

**Desktop (live and installed)**
- [ ] Slate from the first second (no Nuvole flash); wallpaper picker lists only Proton Business Slate, Magenta, Paper
- [ ] Full-width taskbar, Start button with Proton logo, favorites include Collabora Office and no LibreOffice
- [ ] Win11-red-dark icons, Win10OS cursor, no Plasma splash after login
- [ ] NumLock on after login; Hide Cursor and Wobbly Windows on
- [ ] Ctrl+Shift+Esc opens System Monitor; xfreerdp3 not in Start → Internet
- [ ] Pinned Brave opens under the same taskbar icon
- [ ] About This System: Proton logo, protonos.org, 1.0 Core; no Apple logo anywhere

**Installer**
- [ ] Password under 8 characters keeps Next disabled
- [ ] "Third-party software" page with a small logo, Microsoft fonts pre-selected
- [ ] Restart at the end reboots within seconds with the Proton splash

**Installed system**
- [ ] Login and lock screen: Slate, Win10OS cursor, avatar shown; lock screen asks for the password and rejects a wrong one
- [ ] No "Install System" in Start → System
- [ ] Online for 3 minutes: `fc-list | grep -ci arial` above 0 and `ls /var/lib/proton/` empty
- [ ] `sudo ls /boot/efi/EFI/` shows one `proton` folder (plus `boot`)
- [ ] `sudo ufw status verbose`: active, deny incoming
- [ ] `systemctl is-enabled apparmor unattended-upgrades earlyoom`: all enabled
- [ ] `systemctl status earlyoom --no-pager`: active (running)
- [ ] Collabora opens .docx, .xlsx, .pptx and .odt files
- [ ] WinBoat: all requirements green after choosing Podman
- [ ] Discover: no pending "System Software" update (one "KDE Application Platform" Flatpak item is known and harmless)

---

## Known quirks (document, not fixable in the ISO)

- **`grub>` prompt / "unknown filesystem" from USB:** some boards' USB 3 (xHCI) firmware doesn't work with GRUB. Turn on CSM in the BIOS. See [HARDWARE.md](HARDWARE.md).
- **Dell business laptops (e.g. Latitude 5350) won't boot the USB with Secure Boot on:** turn on "Enable Microsoft UEFI CA" in the BIOS.
- **Touchpad natural scrolling:** set per machine in System Settings.

## Open items

- Installed-system GRUB background: on hold until the Btrfs + Snapper decision.
- One machine waits 1:30 on "Job dev-tpmrm0.device/start" at boot. Planned fix: `systemd.tpm2_wait=0` on the live boot entries plus a `/etc/default/grub.d/` drop-in for installed systems (confirm with a one-time GRUB `e` edit first).
- NVIDIA driver option: not started.

---

## Appendix A: Export the current Core files into the repo

Run this once on the build machine (host terminal) to save the files this guide copies from the repo. `$CORE` is the current Core Cubic project (`/home/thrices/cubic-project`).

```bash
CORE=/home/thrices/cubic-project; R=$CORE/custom-root; REPO=~/Proton-OS
mkdir -p $REPO/assets/wallpapers $REPO/assets/pixmaps $REPO/calamares/branding $REPO/grub/themes
cp ~/Documents/"Proton Background"/proton-os-business-*-4k.png $REPO/assets/wallpapers/
cp $R/usr/share/pixmaps/proton-start.png $REPO/assets/pixmaps/
cp -r $R/usr/share/plymouth/themes/proton-splash $REPO/plymouth/
cp $R/etc/plymouth/plymouthd.conf $REPO/plymouth/
cp $R/usr/share/proton-os/99-proton-plymouth.conf $R/etc/initramfs-tools/hooks/proton-plymouth-fonts $REPO/plymouth/
cp -r $R/etc/calamares/branding/$(grep '^branding:' $R/etc/calamares/settings.conf | awk '{print $2}') $REPO/calamares/branding/proton
cp $R/etc/calamares/modules/shellprocess_removeautologin.conf $REPO/calamares/
cp -r $CORE/custom-disk/boot/grub/themes/proton $REPO/grub/themes/
cp $CORE/custom-disk/boot/grub/grub.cfg $CORE/custom-disk/isolinux/isolinux.cfg $REPO/grub/
sudo cp -a $R/etc/skel/. $REPO/skel/ && sudo chown -R $USER: $REPO/skel
rm -rf $REPO/skel/.config/kdedefaults $REPO/skel/.cache
```

Then review before committing. `git status` should show only Proton files: no shell history, caches or personal data. The 4K wallpapers are large (several MB each); that's fine for GitHub.

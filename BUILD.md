**Building Proton OS**  
Base: Debian 13 (Trixie), KDE Plasma, customized via Cubic.  
**Core system fixes**  
1. Start Cubic with a stock Debian 13 KDE ISO.  
2. Enter the chroot terminal.  
3. Re-establish package holds — see packages/holds.txt.  
4. Apply SDDM avatar fix: sddm/login.defs (HOME_MODE 0700 -> 0751).  
5. Apply SDDM theme fix: sddm/kde_settings.conf +  
   
 calamares/shellprocess_removeautologin.conf (surgical removal only).  
6. Install Win10OS cursor theme from  
   
 github.com/yeyushengfan258/Win10OS-cursors via its install.sh.  
7. Apply boot menu changes: grub/grub.cfg and grub/isolinux.cfg  
   
 (Safe Graphics entry, 10s timeout).  
8. Copy skel/ files into /etc/skel in the chroot (kcminputrc, .face,  
   
 discoverrc, PlasmaDiscoverUpdates).  
**Pre-installed software**  
See software/ for exact install commands:  
- Brave Browser (brave-origin) - official Debian repo method  
- ONLYOFFICE Desktop Editors - .deb from onlyoffice.com  
- LibreOffice - included by default in base Debian 13  
- Variety - from Debian repos  
- Winboat (Docker + KVM) - see software/winboat-prerequisites.txt  
**Security defaults**  
See security/ for verification details:  
- unattended-upgrades: installed, Debian security origins only  
- Software Update (KDE Discover): automatic, monthly check  
   
 (skel/discoverrc, skel/PlasmaDiscoverUpdates)  
- AppArmor: installed and enabled, base profiles only  
- UFW firewall: active by default, deny incoming / allow outgoing  
**Finishing the build**  
1. Exit chroot, verify custom-disk/live/ has vmlinuz, initrd.xz,  
   
 and filesystem.squashfs.  
2. Run Cubic's Generate step to completion without interrupting it.  
3. Create the ISO.  

# ArkSys-ISO

ArkSys-ISO is an ArchISO (Arch Linux disk image) forked from [XeroLinux ISO](https://github.com/xerolinux/xero_iso).

## How to build th ArchISO
- Build ISO `sudo mkarchiso -v -w /work-dir -o /out-dir /archiso-dir/`.
```sh
sudo mkarchiso -v -w ./work -o ./ ./archsys-iso/
```
The work dir can be in `/tmp/archiso-tmp`.


- Rebuild ISO:
    - Remove all content inside directory: `sudo rm -rf ./work/*`
    - Remove only files starting with "base": `sudo rm -v ./work/base._*`
    - Delete only the files in directory: `find ./work -maxdepth 1 -type f -delete`

## Configure ArchISO
- Generate a password with openssl: `openssl passwd -6`
- Sign ISO:
```sh
sudo pacman -S gpg archiso

gpg --full-generate-key
gpg --export --armor > ~/Linux-distro-build/keyring.gpg

cp ~/Linux-distro-build/keyring.gpg ~/Linux-distro-build/archiso/airootfs/etc/pacman.d/gnupg/archlinux*

cat << EOF >> ~/Linux-distro-build/archiso/pacman.conf 
[archlinux]
SigLevel = Optional TrustAll
Server = http://mirror.archlinux.org/$repo/os/$arch
EOF

sudo ~/Linux-distro-build/archiso/mkarchiso -v releng/
gpg --detach-sign --armor out/archlinux-x86_64.iso
```


## TODO
- [ ] Sign the ISO image
- [ ] Add Calamares to Archiso
    - [ ] Add local repositories
    - [ ] Build Calamares installer

## DONE
- Add sudoers.d
- Add `sddm.conf.d/kde_settings.conf`
- Login manager for SDDM: `ln -s /usr/lib/systemd/system/sddm.service ~/Linux-distro-build/archiso/archlive/airootfs/etc/systemd/system/display-manager.service`
- Change autologin `~/Linux-distro-build/archiso/archlive/airootfs/etc/systemd/system/getty@tty1.service.d/autologin.conf`. You can modify this file to change the auto login user:
```
[Service]
ExecStart=
ExecStart=-/sbin/agetty --autologin username --noclear %I 38400 linux
```

- To create a user which will be available in the live environment, you must manually edit inside `airootfs/etc/`:
    - passwd
    - shadow
    - group
    - gshadow
- Add password (123) with `openssl passwd -6`: $6$8IVL1j2vTO8C8fX/$J2nWQZt./oqF5jGDcHlo4FxSrooBwuhG1aITc/.yZiTt9I79TKyGazHsgjWjbGbky1PgUenzX2MYC1nRrQA5L1
- Add SSDM theme:
    - Add theme`cp /usr/share/sddm/ ~/Linux-distro-build/archiso/archlive/airootfs/usr/share/sddm/themes/breeze`
    - Config SDDM `~/Linux-distro-build/archiso/archlive/airootfs/etc/sddm.conf`
- Edit shadow and add password

## Changes
## Remove:
-  ./airootfs/usr/share/grub/themes/XeroKDE
-  ./airootfs/usr/share/sddm/themes/XeroDark

## To edit
-  ./airootfs/etc/lightdm
-  ./airootfs/etc/mkinitcpio.d/arksys
-  ./airootfs/etc/sddm.conf
-  ./airootfs/efiboot/loader/entries/archiso-x86_64-linux.conf

## Edited
- ./packages.x86_64


## Tree of directories and files to edit
```
./
├── airootfs/
│   ├── etc/
│   │   ├── calamaresv
│   │   │   └── settings.conf
│   │   ├── gshadow
│   │   ├── hostname
│   │   ├── locale.conf
│   │   ├── mkinitcpio.conf
│   │   │   └── broadcom-wl.conf
│   │   ├── motd
│   │   ├── pacman.d
│   │   │   └── mirrorlist
│   │   ├── passwd
│   │   ├── resolv.conf -> /run/systemd/resolve/stub-resolv.conf
│   │   ├── shadow
│   │   ├── ssh/
│   │   │   └── sshd_config
│   │   ├── systemd/
│   │   │   ├── journald.conf.d
│   │   │   │   └── volatile-storage.conf
│   │   │   ├── logind.conf.d
│   │   │   │   └── do-not-suspend.conf
│   │   │   ├── network
│   │   │   ├── system
│   │   │   │   ├── getty@tty1.service.d
│   │   │   │   │   └── autologin.conf
│   │   │   │   ├── reflector.service.d
│   │   │   │   │   └── archiso.conf
│   │   │   │   └── systemd-networkd-wait-online.service.d
│   │   │   │       └── wait-for-only-one-interface.conf
│   │   │   └── system-generators
│   │   └── xdg/
│   │       └── reflector
│   │           └── reflector.conf
│   ├── root/
│   │   ├── custom/  # --- added ---
│   │   │   ├── customize_rootfs.sh
│   │   │   └── packages.both
│   │   └── run_archiso.sh
│   └── usr
├── bootstrap_packages.x86_64
├── efiboot/
│   └── loader
│       ├── entries
│       │   ├── 01-archiso-x86_64-linux.conf
│       │   └── 02-archiso-x86_64-speech-linux.conf
│       └── loader.conf
├── grub/
│   └── grub.cfg
├── packages.x86_64
├── pacman.conf
├── profiledef.sh
├── syslinux/
│   └── splash.png
├── tree-edit.txt
└── TODO.md
```

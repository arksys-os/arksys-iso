# ArkSys-ISO

ArkSys-ISO is an [ArchISO](https://wiki.archlinux.org/title/Archiso) (Arch Linux live CD/USB ISO image) forked from [XeroLinux ISO](https://github.com/xerolinux/xero_iso).

## How to build ArkSys-ISO

1. First you need to clone this repository (via https or ssh)
```
git clone https://github.com/David7ce/arksys-iso.git
```

2. To build ISO you need to create a work directory and a output directory for the ISO. Then us `mkarchiso`:
> The work directory can be in `/tmp/archiso-tmp`.

```sh
mkdir -p ~/archiso/work && cd ~/archiso

sudo mkarchiso -v -w ./work -o ./ ./archsys-iso
```

- To rebuild ISO, just remove files of work directory in one of these ways:
    - Remove all content inside directory: `sudo rm -rf ./work/*`
    - Remove only files starting with "base": `sudo rm -v ./work/base._*`
    - Delete only the files in directory: `find ./work -maxdepth 1 -type f -delete`

## TO DO
- [ ] Sign the ISO image:
```sh
sudo pacman -S gpg archiso
gpg --full-generate-key
gpg --export --armor >  ~/archiso/work/keyring.gpg
cp ~/archiso/arksys-iso/keyring.gpg ~/archiso/arksys-iso/airootfs/etc/pacman.d/gnupg/archlinux*

cat << EOF >> ~/archiso/arksys-iso/pacman.conf 
[archlinux]
SigLevel = Optional TrustAll
Server = http://mirror.archlinux.org/$repo/os/$arch
EOF

sudo ~/archiso/arksys-iso/mkarchiso -v releng/
gpg --detach-sign --armor out/archlinux-x86_64.iso
```
- [ ] Build own config Calamares installer and the repository from local

## DONE
- Add sudoers.d
- Login manager for SDDM: `ln -s /usr/lib/systemd/system/sddm.service ~/archiso/arksys-iso/airootfs/etc/systemd/system/display-manager.service`
- Change autologin `~/archiso/arksys-iso/airootfs/etc/systemd/system/getty@tty1.service.d/autologin.conf`. You can modify this file to change the auto login user:
```sh
[Service]
ExecStart=
ExecStart=-/sbin/agetty --autologin username --noclear %I 38400 linux
```
- To create a user which will be available in the live environment, you must manually edit inside `~/archiso/arksys-iso/airootfs/etc/shadow`:
    - passwd
    - shadow
    - group
    - gshadow
- Add password with `openssl passwd -6` and copy to airootfs/shadow/shadow.
```sh
openssl passwd -6
Password:  # Type password then copy the output (106 characters)
```
- Add SSDM theme:
    - Add theme`cp /usr/share/sddm/ ~/archiso/arksys-iso/airootfs/usr/share/sddm/themes/breeze`
    - Config SDDM `~/archiso/arksys-iso/airootfs/etc/sddm.conf`

## Changes from [XeroLinux ISO](https://github.com/xerolinux/xero_iso/tree/main/Xero)

- Edited:
    - ./packages.x86_64

- Removed:
    - ./airootfs/usr/share/grub/themes/XeroKDE
    - ./airootfs/usr/share/sddm/themes/XeroDark

- To edit:
    - ./airootfs/etc/lightdm
    - ./airootfs/etc/mkinitcpio.d/arksys
    - ./airootfs/etc/sddm.conf
    - ./airootfs/efiboot/loader/entries/archiso-x86_64-linux.conf

---

## Tree of directories and files to edit
```
./
├── airootfs/
│   ├── etc/
│   │   ├── calamares/
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
└── README.md
```

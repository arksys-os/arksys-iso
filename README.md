# ArkSys-ISO

ArkSys-ISO is an [ArchISO](https://wiki.archlinux.org/title/Archiso) profile to generate a disk ISO image. ArkSys-ISO is similar to [XeroLinux-ISO](https://github.com/xerolinux/xero_iso), a disk image with Arch Linux, KDE and Calamares installer to install the system in a graphical and easy way.

## How to build ArkSys-ISO

0. Install archiso tool with pacman, necessary to build an Arch Linux live ISO image.
```sh
sudo pacman -S archiso
```

1. Download arksys with git
```
git clone https://github.com/David7ce/arksys-iso.git
```

> If you want to build a basic Arch Linux system, copy the archiso basic profile `cp -r /usr/share/archiso/configs/releng/ archlive` and follow [archiso documentation](https://wiki.archlinux.org/title/Archiso).

2. To build the ISO you need to create a work directory and an output directory for the ISO. Then us `mkarchiso` and do one of these:
```sh
# sudo mkarchiso -v -w  ~/Linux-distro-build/work -o ./ ./arksys-iso
# sudo mkarchiso -v -w  /tmp/archiso-tmp ./arksys-iso
```
> Tip: If memory allows, it is preferred to place the working directory on tmpfs (/tmp/archiso-tmp)

- To rebuild ISO, just remove files of work directory in one of these ways:
    - Remove all content inside directory: `sudo rm -rf ./work/*`
    - Remove only files starting with "base": `sudo rm -v ./work/base._*`
    - Delete only the files in directory: `find ./work -maxdepth 1 -type f -delete`

## Calamares installer
Calamares installer is a system installer, it is necessary if you want to install the OS permanently, because archiso is a live environment that is stored on the RAM.

### Configure and build Calamares
There are two options:

- A. Download calamares from GitHub, configure and build.

To configure the app you need the clone the github repo.
```sh
git clone https://github.com/calamares/calamares.git
```

After that you can configure the appareance, the packages to install, the partitions, etc. inside calamares folder:
- ./src/branding/distroname
- ./src/modules/*.conf
- ./settings.conf

```
./  # calamares directory
├── src/
|   ├── branding/
|   |   └── distroname/
|   └── modules/
|       └── *.conf
└── settings.conf
```

Once you have configured it you can build the app inside build directory.
```sh
mkdir calamares/build && cd calamares/build
cmake -DCMAKE_BUILD_TYPE=Debug .. # cmake ..
make
```

- B. Use a custom calamares configuration like [calamares-xerolinux](https://github.com/xerolinux/calamares-cfg), edit with ouwn taste and build. To build the package use `makepkg`
```sh
makepkg
```
> You can build and install the package with `makepg -si`

This will generate a compressed package `pkgname-pkgver-arch.pkg.tar.zst` that you can install with pacman or add it to your database of packages.
```sh
sudo pacman -U calamares-cfg-*.pkg.tar.zst
```

### How install Calamares
Calamares can be installed using pacman from a repository (local or remote) or importing the necessary files in the system.

#### A. Calamares installation with pacman (recommended)
Calamares is not in the Arch repo, so you need to have your own repositories and configure the mirrorlist, the keys and the database. This better is beter if you plan on updating Calamares frequently.

To download the package with pacman you can use an online or local repository:

- To use an **online repo** configure pacman.conf and add the repository of your calamares-installer package:
```
# XeroLinux Repository
#[valen_repo]
#SigLevel = Never
#Server = https://keyaedisa.github.io/$repo/$arch
```
And add the package name to `packages.x86_64`.

- To use a **local repo** you need to generate a database. The database is a tar file with the extension .db or .files followed by an archive extension of .tar, .tar.gz, .tar.bz2, .tar.xz, .tar.zst, .tar.Z.

```sh
# add a package to the database
repo-add /path/to/repo.db.tar.gz /path/to/package-1.0-1-x86_64.pkg.tar.zst

# add all packages in that path to the database
repo-add /path/to/repo.db.tar.gz /path/to/*.pkg.tar.zst
```

Use a text editor and add the name of the package (`package-1.0-1-x86_64.pkg.tar.zst`) to `packages.x86_64`. For example with the package `calamares-arksys-1-x86_64.pkg.tar.zst` add calamares-arksys to the list of packages.

Then add the database to the pacman.conf
```
# Local repo (packages in my system)
#[my_repo]
#SigLevel = Optional TrustAll
#Server = file:////home/username/arch-repo/pkgname-pkgver-arch.pkg.tar.zst
```

#### B. Calamares installation importing libs and configuration (not recommended)
If you are going to stay in a version of calamares is easier just import calamares. To do that just need to copy these files and dirs into airootfs/ of the archISO:
> There is no /usr/bin/calamares /usr/share/calamares/modules

```sh
./ # calamares directory
└── airootfs/
    ├── etc/
    │   └── calamares/
    │       ├── branding/
    │       ├── modules/
    │       └── settings.conf
    └── usr/
        ├── lib/
        │   ├── calamares/
        │   │     ├── modules/
        │   │     │   └── default/
        │   │     └── libcalamares.so
        │   ├── libcalamares.so -> libcalamares.so.3.3.0
        │   ├── libcalamares.so.3.3 -> libcalamares.so.3.3.0
        │   ├── libcalamares.so.3.3.0
        │   └── libcalamaresui.so -> libcalamares.so.3.3.0
        └── share/
            ├── applications/
            │    └── calamares.desktop
            └── calamares/
               ├── branding/
               │   └── default/
               └── qml/
                   ├── calamares/
                   └── slideshow/
```

```sh
# copy calamares (branding/, modules/, settigns.conf)
cp -r ~/ldb/calamares-cfg/etc/ ~/ldb/arksys-iso/airootfs/etc/

cp -r ~/ldb/calamares/build/libcalamaresui.so.3.3.0 ~/ldb/airootfs/usr/lib/calamares
cp -r ~/ldb/calamares/build/src/modules/ ~/ldb/arksys-iso/airootfs/usr/lib/calamares

cp -r ~/ldb/calamares/build/src/branding/ ~/ldb/arksys-iso/airootfs/usr/share/calamares
cp -r ~/ldb/calamares/build/src/qml/ ~/ldb/arksys-iso/airootfs/usr/share/calamares

# cp -r src/calamares ~/ldb/arksys-iso/airootfs/usr/share/

cat << EOF >> ~/ldb/arksys-iso./airootfs/usr/share/applications/calamares.desktop
[Desktop Entry]
Type=Application
Version=1.0
Name=Install System
GenericName=System Installer
Comment=Calamares — System Installer
Keywords=calamares;system;installer;
TryExec=calamares
Exec=sh -c "pkexec calamares"

Categories=Qt;System;
Icon=calamares
Terminal=false
SingleMainWindow=true
StartupNotify=true
X-AppStream-Ignore=true
EOF
```

## Tree of archiso (important files)
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


<!--
## Errors
- Can't install calamares from AUR, dependecy error.

## TO DO
- [ ] Configure Calamares installer and build
- [ ] Install Calamares on archiso (with pacman or importing files)
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
-->
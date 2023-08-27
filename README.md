> Arksys login:
- user: arklive
- password: 123

# Steps to build Arksys ISO

ArkSys-ISO is an [ArchISO](https://wiki.archlinux.org/title/Archiso) profile to generate a disk ISO image. ArkSys-ISO is a disk image with Arch Linux, KDE and Calamares installer to install the system in a graphical and easy way.

## 0. Download 'archiso' package
- Install 'archiso' or 'archiso-git' tool with pacman, necessary to build an Arch Linux live ISO image: `sudo pacman -S archiso`


## 1. Create archiso profile: 'archiso-arksys'
- Adapt a custom archiso profile, copy archiso profile 'releng' to a new dir:`cp -r /usr/share/archiso/configs/releng/. archlive` or download an archiso-profile `git clone https://github.com/David7ce/arksys-iso.git`
- Edit the archiso profile adding packages for installation in 'packages.x86_64'
- Add the mirrorlist in 'pacman.conf and 'airootfs/etc/pacman.conf'
```sh
# $repo is the word between [name-repo] must be the same as name of the database name-repo.db
# $arch is the architecture (x86_64, arm, ...)

# ArkSys server: Online (https://) or offline/local (file://)
[arksys-repo]
SigLevel = Optional TrustAll
Server = https://github.com/arksys-os/$repo/raw/main/$arch
# Server = https://arksys-os.github.io/$repo/$arch
# Server = file:///0/Workspaces/ArkSys-project/$repo/$arch
# Include = /etc/pacman.d/arksys-mirrorlist

## Examples
# https://github.com/arksys-os/arksys-repo/raw/main/x86_64/grub-tools-1.6.7-1-any.pkg.tar.zst
# https://arksys.github.io/arksys-repo/x86_64/calamares-3.2.62-1-x86_64.pkg.tar.zst
# file:///0/Workspaces/ArkSys-project/arksys-repo/x86_64/calamares-3.2.62-1-x86_64.pkg.tar.zst
```

- Configure user profile in 'airootfs/usr'
    - Login manager for SDDM:
    ```sh
    ln -s /usr/lib/systemd/system/sddm.service airootfs/etc/systemd/system/display-manager.service`
    ```
    - Change autologin in `airootfs/etc/systemd/system/getty@tty1.service.d/autologin.conf`. You can modify this file to change the auto login user:
    ```sh
    [Service]
    ExecStart=
    ExecStart=-/sbin/agetty --autologin username --noclear %I 38400 linux
    ```

- Configure live environment in 'airootfs/etc'
    - Create a user in live environment, editing `airootfs/etc/shadow`:
        - passwd
        - `openssl passwd -6` > shadow
        - group
        - gshadow

> Optional configure a welcome-app and add to the package repository

## 2. Configure framework installer: calamares-installer
- We need two apps:
    - 'calamares-app', calamares main app
    - 'calamares-config', calamares branding and modules configuration

There are two options for installing both apps:

A. Create a pkgbuild 'calamares-app' and 'calamares-config' and build with `makepkg` then move the generated 'tar.gz' package to the repository for installing with pacman.
B. Or copy the packages permanently in "airootfs/etc/calamares", easier for offline installation.


> To build calamares installer from GitHub repo:
```sh
git clone https://github.com/calamares/calamares.git
mkdir calamares/build && cd calamares/build
cmake -DCMAKE_BUILD_TYPE=Debug ..
make
```

## 3. Configure the package repository: 'arksys-repo'
- Create a package database as a tar file, valid extensions are .db or .files followed by:  .tar.gz, .tar.bz2, .tar.xz, .tar.zst, or .tar.Z.
```sh
cd ./arksys-repo/x86_64/

# add all packages with ".pkg.tar.zst" extension in that path to the database
repo-add arksys-repo.db.tar.zst *.pkg.tar.zst

# remove symlinks
rm arksys-repo.db
rm arksys-repo.files

# rename files without ".tar.gz" because the symlinks do not exist
mv arksys-repo.db.tar.gz arksys-repo.db
mv arksys-repo.files.tar.gz arksys-repo.files
```

- Upload the repository to a remote server, for example a GitHub repository

> For remote server is better adding the db symlink and the db tarball with extensions for downloading with the correct format. While for local server you need the remove the symlink and change the name of the db tarball to redirect directly.

## 4. Build the ISO
```sh
# build the iso
sudo mkarchiso -v -w ~/work -o ./ ./arksys-iso

# To rebuild the ISO you need to remove content of work
rm -rf ~/work/*
```
where:
-w work directory
-o is output directory

## Tree of archiso (important files)
```sh
./
├── airootfs/
│   ├── etc/
│   │   ├── calamares/
│   │   │   ├── branding/       # can be installed via pacman
│   │   │   │   └── distroname/ # can be installed via pacman
│   │   │   ├── modules/        # can be installed via pacman
│   │   │   │   └── *.conf      # can be installed via pacman
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

## References
- [Archiso - ArchWiki](https://wiki.archlinux.org/title/Archiso)
- [Unofficial repositories - ArchWiki](https://wiki.archlinux.org/index.php/unofficial_user_repositories)

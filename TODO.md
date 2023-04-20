## TODO

- Add sudoers.d

- Login manager for SDDM: `ln -s /usr/lib/systemd/system/sddm.service ~/archiso/arksys-iso/airootfs/etc/systemd/system/display-manager.service`

- Change autologin `~/archiso/arksys-iso/airootfs/etc/systemd/system/getty@tty1.service.d/autologin.conf`. You can modify this file to change the auto login user:
```sh
[Service]
ExecStart=
ExecStart=-/sbin/agetty --autologin username --noclear %I 38400 linux
```

- Create a user in live environment, editing `~/archiso/arksys-iso/airootfs/etc/shadow`:
    - passwd
    - `openssl passwd -6` > shadow
    - group
    - gshadow

- Generate packages, calamares and calamares-config with `mkpkg -s`
`tar -cJf calamares-arksys-config-0.0.1-any.pkg.tar.zst ./calamares-arksys-config-0.0.1-any.pkg/`

- Add packages to db repository: `repo-add ./arksys_repo.db.tar.gz ./*.pkg.tar.zst`

> To add a repository that can be used in the live environment, create a suitably modified pacman.conf and place it in archlive/airootfs/etc/. 

If the repository also uses a key, place the key in archlive/airootfs/usr/share/pacman/keyrings/. The key file name must end with .gpg. Additionally, the key must be trusted. This can be accomplished by creating a GnuPG exported trust file in the same directory. The file name must end with -trusted. The first field is the key fingerprint, and the second is the trust. You can reference /usr/share/pacman/keyrings/archlinux-trusted for an example. 

- Sign package database file with gpg. For example, you might run the following command:
`gpg --detach-sign /0/Workspaces/David7ce-repos/ArkSys-project/arksys-repo/x86_64/arksys_repo.db.tar.gz`

- Add repo server (local or online)
```sh
# local
[myrepo]
SigLevel = Optional TrustAll
Server = file:///path/to/myrepo/x86_64

# online
[iso-repo]
SigLevel = Optional TrustAll
Server = https://repos.xerolinux.xyz/$repo/$arch
```

- Build the ISO
```sh
sudo rm -r /tmp/archiso-tmp/*
sudo mkarchiso -v -w  /tmp/archiso-tmp /0/Workspaces/David7ce-repos/ArkSys-project/arksys-iso
```

- Sign the ISO image:
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

<!--
## Errors

```sh
warning: cannot resolve "libfprint-1", a dependency of "fingerprint-gui"
warning: cannot resolve "noto-color-emoji-fontconfig", a dependency of "printer-support"
warning: cannot resolve "samba-support", a dependency of "printer-support"

warning: cannot resolve "fonts-tlwg", a dependency of "asian-fonts"
warning: cannot resolve "lohit-fonts", a dependency of "asian-fonts"

error: failed to prepare transaction (could not satisfy dependencies)
```
-->
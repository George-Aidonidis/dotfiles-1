# Étapes de l'installation d'Arch Linux

Ce n'est pas un script ! Ceci me permet de me rappeler les différentes étapes !
## 1. Partitionnement
- **512Mo** pour **/boot**
- **100%** Reste en **luks** -> **vg** :
	- **RAM+1Go** pour swap -> **vg-swap**
	- **~100Go** pour swap -> **vg-var**
	- **~50Go** pour swap -> **vg-root**
	- **100%FREE** pour swap -> **vg-home**

[si UEFI]

 **/boot** **1Go** en FAT32.
Il faut qu’elle soit étiquetée en EF00 à sa création.
Pour le swap, c’est la référence 8200.

On doit remplir la partition cryptée avec des données aléatoires avant:

```shell
cryptsetup luksFormat --cipher twofish-xts-plain64 --key-size 512  --hash sha512 --iter-time 2000 /dev/sda2

crypsetup luksOpen /dev/sda2 luks

dd if=/dev/zero of=/dev/mapper/luks status=progress
```

## 2. Téléchargement d'Archlinux Boostrap

```shell
cd /tmp/
wget https://archlinux/iso/latest/ ...archlinux-boostrap-...

```

## 3. Chroot dans l'environnement Arch Boostrap

```shell
tar xzf <path-to-boostrap-image>/archlinux-boostrap-...
/tmp/root.x86_64/bin/arch-chroot /tmp/root.x86_64
```

## 4. Installation de logiciels de base

```shell
 # Installation système de base
 pacstrap /mnt base base-devel nvim git openssh grub zsh sudo
 # /etc/fstab pour lister les partitions
 genfstab -U -p /mnt >> /mnt/etc/fstab
 # chroot dans le nouveau système
 arch-chroot /mnt /bin/zsh
```

# 5. Dans le nouveau système configuration

```shell
  nvim /etc/vconsole.conf
  # KEYMAP=fr-latin9
  # FONT=ter-powerline-v32n

  nvim /etc/locale.conf
  # LANG=fr_FR.UTF-8
  # LC_COLLATE=C
  locale-gen
  export LANG=fr_FR.UTF-8

  # Name machine
  nvim /etc/hostname

  # Date time machine
  ln -sf /usr/share/zoneinfo/Africa/Kinshasa /etc/localtime
  hwclock --systohc --utc

  # Ajouter dans /etc/mkinitcpio.conf
  nvim /etc/mkinitcpio.conf
  # HOOKS=(base udev autodetect modconf block keyboard keymap consolefonts encrypt lvm2 filesystems resume fsck)
  mkinitcpio -p linux
  # Activer l'hibernation
  nvim /etc/default/grub
  # GRUB_CMDLINE_LINUX="cryptdevice=/dev/sda2:luks:allow-discards resume=UUID=8b9c73e9-d514-4cd0-ad5c-174c7228f110"
  grub-mkconfig -o /boot/grub/grub.cfg
```

## 6. Installation grub

- Récuperer et compiler ma propre version de grub
- Teste si on est sur UEFI ou BIOS ?
   ```shell
     efivars on /sys/firmware/efi/efivars type efivars (rw,nosuid,nodev   noexec,relatime)
   ```
- Si UEFI
	```shell
	   grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=arch_grub --recheck
	   mkdir /boot/EFI/boot
	   cp /boot/EFI/arch_grub/grubx64.efi /boot/EFI/boot/bootx64.efi
	```
- Si BIOS
	```shell
	grub-install --no-floppy --recheck /dev/sda
	```

## 7. Utilisateur

```shell
# changer mot de passe du root
passwd root
# création d'un utilisateur administrateur
useradd  -m  -g  wheel -c 'Nom complet de l’utilisateur' -s  /bin/zsh  nom-de-l’utilisateur
passwd nom-de-l’utilisateur
# décommenter wheel dans sudoers
nvim /etc/sudoers

# se connecter en tant nom-de-l'utilisateur
su nom-de-l'utilsateur
```

## 8. Dotfiles [ Optionel et relatif m'aide à me rappeler ]

-cp -r Terminus/PSF/*.psf.gz /usr/share/kkbd/consolefonts/
installation de megasync et confiuration
installation des logiciels avec yaourt, vscode et yarn

récuperer les conf:
ForwardToSyslog=yes on /etc/systemd/journald.conf

Activez les services ci-après:

systemctl enable syslog-ng  #gestion des fichiers d’enregistrement d’activité
systemctl enable networkManager
systemctl enable vboxservice
systemctl enable cronie  #pour les tâches récurrentes
systemctl enable avahi-daemon  →dépendance de Cups
systemctl enable avahi-dnsconfd  →autre dépendance de Cups
systemctl enable org.cups.cupsd  →cups pour les imprimantes
systemctl enable bluetooth  →uniquement si on a du matériel bluetooth
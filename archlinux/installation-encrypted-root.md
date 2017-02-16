# Arch Linux installation with encrypted root, home and swap

## Preparing the hard disk
### Creating the partition layout
```bash
parted /dev/sda
mklabel gpt
```

#### ONLY if you are in legacy bios mode
```bash
mkpart primary 1MB 2MB
mkpart primary 2MB 1GB
set 1 bios_grub on
```

#### ONLY if you are in efi bios mode
```bash
mkpart primary 1MB 1GB
mkpart primary 1GB 2GB
set 1 boot on
```

### Legacy bios and efi
```bash
mkpart logical 2GB 100%
```

## Preparing the encrypted system partitions
### Creating of the LUKS device
```bash
cryptsetup -v --cipher aes-xts-plain64 --key-size 512 --hash sha512 --iter-time 10000 --use-urandom --verify-passphrase luksFormat /dev/sda3
cryptsetup luksOpen /dev/sda3 crypt0
```

### Creating partitions within the Luks device using LVM
```bash
pvcreate /dev/mapper/crypt0
vgcreate crypt0-vg0 /dev/mapper/crypt0
lvcreate -L 8GiB -n swap crypt0-vg0
lvcreate -L 70GiB -n root crypt0-vg0
lvcreate -l 100%FREE -n home crypt0-vg0
```

### Formatting the partitions
#### ONLY if you are in efi bios mode
```bash
mkfs.vfat -F 32 -v -n EFIBOOT /dev/sda1
```

#### Legacy and EFI mode
```bash
mkfs.ext4 -L boot /dev/sda2
mkfs.ext4 -L root /dev/mapper/crypt0-vg0-root
mkfs.ext4 -L home /dev/mapper/crypt0-vg0-home
mkswap -L swap /dev/mapper/crypt0-vg0-swap
```

### Mount partitions
```bash
mount -t ext4 /dev/mapper/crypt0-vg0-root /mnt
mkdir /mnt/boot && mount -t ext4 /dev/sda2 /mnt/boot
mkdir /mnt/home && mount -t ext4 /dev/mapper/crypt0-vg0-home /mnt/home
swapon /dev/mapper/crypt0-vg0-swap
```

#### ONLY if you are in efi bios mode
```bash
mkdir /mnt/boot/efi && mount /dev/sda1 /mnt/boot/efi
```

## Installing & configuring Arch Linux
### Bootstrapping Arch
```bash
pacstrap /mnt base base-devel
```

### Configuring Arch
#### Generate fstab
```bash
genfstab -p -U /mnt > /mnt/etc/fstab
```

#### Nothing to say here
```bash
arch-chroot /mnt /bin/bash
```

#### Update mirror list
```bash
curl "https://www.archlinux.org/mirrorlist/?country=FI&country=DE&country=IS&country=LU&country=NL&country=NZ&country=CH&protocol=https&ip_version=4&ip_version=6&use_mirror_status=on" | sed 's/#Server/Server' > /etc/pacman.d/mirrorlist
pacman -Syu
```

#### Hostname
```bash
echo "ihavecookies" > /etc/hostname
```

#### Time zone
```bash
ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime
```

#### Generate locales
```bash
sed -ie 's/#de_DE/de_DE/g'
sed -ie 's/#en_US/en_US/g'
locale-gen
```

#### Set default locales
`/etc/locale.conf`
```bash
LANG=en_US.UTF-8
LC_CTYPE="de_DE.UTF-8"
LC_NUMERIC="de_DE.UTF-8"
LC_TIME="de_DE.UTF-8"
LC_COLLATE="de_DE.UTF-8"
LC_MONETARY="de_DE.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="de_DE.UTF-8"
LC_NAME="de_DE.UTF-8"
LC_ADDRESS="de_DE.UTF-8"
LC_TELEPHONE="de_DE.UTF-8"
LC_MEASUREMENT="de_DE.UTF-8"
LC_IDENTIFICATION="de_DE.UTF-8"
LC_ALL=
```

#### Set console keymap
```bash
echo "KEYMAP=de-latin1-nodeadkeys"
```

#### Add scripts to ramdisk
`/etc/mkinitcpio.conf`
```bash
HOOKS=" ... keyboard block encrypt lvm2 filesystems ..."
mkinitcpio -p linux
```

#### Install Grub
```bash
pacman -S grub
```

##### ONLY if your are in efi boot mode
```bash
pacman -S efibootmgr dosfstools
```

#### Configure grub
`/etc/default/grub`
```bash
GRUB_CMDLINE_LINUX_DEFAULT="cryptdevice=UUID=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX:crypt0"
```

#### Install Grub to disk
```bash
grub-install /dev/sda
```

#### Generate Grub config
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

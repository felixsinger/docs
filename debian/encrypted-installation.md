# Debian encrypted installation

## Creation of the partition layout
```bash
parted /dev/sda
```
```bash
mklabel gpt
```
```bash
mkpart primary 1MB 2MB
```
```bash
mkpart primary 2MB 1GB
```
```bash
mkpart logical 1GB 100%
```
```bash
set 1 bios_grub on
```

## Creation of logical volumes in the LUKS device
```bash
cryptsetup -v --cipher aes-xts-plain64 --key-size 512 --hash sha512 --iter-time 5000 --use-urandom --verify-passphrase luksFormat /dev/sda3
```
```bash
cryptsetup luksOpen /dev/sda3 sda3_crypt
```
```bash
pvcreate /dev/mapper/sda3_crypt
```
```bash
vgcreate vg0 /dev/mapper/sda3_crypt
```
```bash
lvcreate -L 1GiB -n swap vg0
```
```bash
lvcreate -l 100%FREE -n root vg0
```

## Formatting the partitions
```bash
mkfs.ext4 -L boot /dev/sda2
```
```bash
mkfs.ext4 -L root /dev/mapper/vg0-root
```
```bash
mkswap -L swap /dev/mapper/vg0-swap
```

## Getting Debian
```bash
mount -t ext4 /dev/mapper/vg0-root /mnt
```
```bash
debootstrap --arch amd64 jessie /mnt/ http://ftp.debian.org/debian/
```
```bash
mount -t ext4 /dev/sda2 /mnt/boot && mount --bind /sys /mnt/sys && mount --bind /run /mnt/run && mount --bind /proc /mnt/proc && mount --bind /dev /mnt/dev && mount --bind /dev/pts /mnt/dev/pts
```
```bash
chroot /mnt /bin/bash
```

## Configuration of Debian
```bash
ln -sf /proc/mounts /etc/mtab
```
`nano /etc/apt/sources.list`
```bash
deb http://ftp.de.debian.org/debian/ jessie main
deb-src http://ftp.de.debian.org/debian/ jessie main

deb http://security.debian.org/ jessie/updates main
deb-src http://security.debian.org/ jessie/updates main

# jessie-updates, previously known as 'volatile'
deb http://ftp.de.debian.org/debian/ jessie-updates main
deb-src http://ftp.de.debian.org/debian/ jessie-updates main
```
```bash
apt update && apt upgrade -y
```
`nano /etc/network/interfaces`
```bash
auto lo
 iface lo inet loopback

auto eth0
 iface eth0 inet dhcp
```
```bash
echo "debian-server" > /etc/hostname
```
`nano /etc/crypttab`
```bash
sda3_crypt UUID= none luks
```
`nano /etc/fstab`
```bash
# root
UUID=   /       ext4  errors=remount-ro   0   1

# boot
UUID=   /boot   ext4  rw,nosuid,nodev     0   2

# swap
UUID=   none    swap  sw                  0   0
```
```bash
apt install busybox cryptsetup grub-pc initramfs-tools linux-image-amd64 locales lvm2 ssh
```
```bash
apt install dropbear
```
```bash
/usr/lib/dropbear/dropbearconvert openssh dropbear /etc/ssh/ssh_host_rsa_key /etc/initramfs-tools/etc/dropbear/dropbear_rsa_host_key
```
```bash
/usr/lib/dropbear/dropbearconvert openssh dropbear /etc/ssh/ssh_host_dsa_key /etc/initramfs-tools/etc/dropbear/dropbear_dss_host_key
```

`nano /etc/initramfs-tools/hooks/unlock`
```bash
#!/bin/sh

PREREQ=""
prereqs()
{
  echo "$PREREQ"
}
case $1 in
  prereqs)
    prereqs
    exit 0
    ;;
esac

. /usr/share/initramfs-tools/hook-functions

cat > "${DESTDIR}/root/unlock" << EOF
#!/bin/sh
/lib/cryptsetup/askpass 'Passphrase: ' > /lib/cryptsetup/passfifo
EOF

chmod u+x "${DESTDIR}/root/unlock"

exit 0
```
```bash
rm /etc/initramfs-tools/root/.ssh/id_rsa*
```
```bash
echo "YOUR PUBLIC KEY" > /etc/initramfs-tools/root/.ssh/authorized_keys
```

`nano /etc/initramfs-tools/initramfs.conf`
```bash
DEVICE=eth0

DROPBEAR=y
CRYPTSETUP=y
```
```bash
update-initramfs -u -k all
```
```bash
passwd root
```

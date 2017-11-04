# Basic packages

```bash
pacman -S acpid chrony htop cronie zip unzip unrar tmux smartmontools rsync pciutils p7zip openssh openssl hdparm lm_sensors net-tools nmap bind-tools openbsd-netcat sudo mtr whois linux-headers wget curl bash-completion parted git vim dosfstools ntfs-3g
```

```bash
systemctl enable acpid
systemctl enable cronie
systemctl enable smartd
systemctl enable chronyd
```

```bash
hwclock -w
```

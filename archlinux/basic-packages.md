# Basic packages

```bash
pacman -S acpid ntp htop cronie zip unzip unrar tmux smartmontools rsync pciutils p7zip openssh openssl hdparm lm_sensors net-tools nmap bind-tools openbsd-netcat sudo mtr whois linux-headers wget curl bash-completion parted git vim
```

```bash
systemctl enable acpid
systemctl enable cronie
systemctl enable smartd
```

```bash
ntpdate -u 0.de.pool.ntp.org
hwclock -w
```

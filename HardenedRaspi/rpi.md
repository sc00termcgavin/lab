## Hardened Raspberry Pi Setup 

### Step 1: Create SSH key pair

```shell
$ cd ~/.ssh

$ ssh-keygen -t ed25519 -C "pi"
```

### Step 2: Generate Heredoc for the output to be used at [step 15](#step-15-Configure-Pi-ssh-authorized-keys)

```shell
cat << EOF
cat << "_EOF" > ~/.ssh/authorized_keys
$(cat ~/.ssh/pi.pub)
_EOF
EOF
```

### Step 3: Copy Raspberry Pi OS Lite to microSD card

#### On macOS


> Run: `diskutil list` to find disk ID of microSD card
> Replace: `disk>N<` and `rdisk>N<` with disk ID (`disk4` and `rdisk4` in my case)
> Replace  `2024-03-15-raspios-bookworm-arm64-lite.img` with most up to date image
```console
$ diskutil list 

$ sudo diskutil unmountDisk /dev/disk>N< 
Unmount of all volumes on disk4 was successful

$ sudo dd bs=1m if=$HOME/Downloads/2024-03-15-raspios-bookworm-arm64-lite.img of=/dev/rdisk>N<
2640+0 records in
2640+0 records out
2768240640 bytes transferred in 38.358055 secs (72168431 bytes/sec)

$ sudo diskutil unmountDisk /dev/disk>N<
Unmount of all volumes on disk4 was successful
```

#### On linux

- Extract PI OS LITE archive

```shell
unxz 2024-03-15-raspios-bookworm-arm64-lite.img.xz
```

- Copy raspi to microSD

```console
$ sudo fdisk --list

$ sudo umount /dev/sdn*

$ sudo dd bs=1M if=2024-03-15-raspios-bookworm-arm64-lite.img of=/dev/sdn
```

### Step 4: Configure keyboard

### Step 5: Create User

- `pi-admin`
- password output from `openssl rand -base64 24`

### Step 6: Fixing Locale issue
> edit /etc/locale.gen uncomment en_US.UTF-8

```shell
sudo nano /etc/locale.gen

sudo locale-gen en_US.UTF-8

sudo update-locale en_US.UTF-8
```

### Step 7: Set hostname

```shell
sudo raspi-config
```

- Select “System Options”, then “hostname”, name: `tinyca` | `pihole`

### Step 8: Wifi Configure

- Select “System Options”, then “Wireless LAN”, choose country, then select “OK”, enter “SSID” and, finally, enter passphrase.

### Step 9: disable auto login

- Select “System Options”, then “Boot / Auto Login” and, finally, select “Console”.
  
### Step 10: enable SSH

- Select “Interface Options”, then “SSH”, then “Yes”, then “OK” and, finally, select “Finish”.

When asked if you wish to reboot, select “No”.


### Step 11: Set Static IP for Raspberry Pi's. 
> One for Pi-hole, a second for TinyCA. Make sure to change the DHCP range to prevent automatic assignment of an IP address.

> ASUS has a guide on their page [Router Setup](https://www.asus.com/support/faq/1046062/)


### Step 12: Obtain IP of Raspberry Pi (`eth0`: Ethernet | `wlan0`: WI-FI)
> Make sure it matches your reserved static IP on the Router

```shell
ip a
```

### Step 13: SSH into the Pi
> Heads-up: replace `10.0.1.94` with IP of Raspberry Pi

```shell
ssh pi-admin@10.0.1.94
```

### Step 14: Disable Bash history for the Pi

```shell
sed -i -E 's/^HISTSIZE=/#HISTSIZE=/' ~/.bashrc
sed -i -E 's/^HISTFILESIZE=/#HISTFILESIZE=/' ~/.bashrc
echo "HISTFILESIZE=0" >> ~/.bashrc
history -c; history -w
source ~/.bashrc
```

### Step 15: Configure Pi SSH authorized keys

#### Create `.ssh` directory

```shell
mkdir ~/.ssh
```

#### Create `~/.ssh/authorized_keys` using heredoc generated at [step 2](#step-2-generate-heredoc-the-output-of-following-command-will-be-used-at-step-13)

```shell
cat << "_EOF" > ~/.ssh/authorized_keys
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHLwQ2fk5VvoKJ6PNdJfmtum6fTAIn7xG5vbFm0YjEGY pi
_EOF
```

## Step 16: log out

```shell
exit
```

### Step 17: log in
> Heads-up: replace `10.0.1.94` with IP of Raspberry Pi.

> Heads-up: when asked for passphrase, enter passphrase from [step 1](#step-1-create-ssh-key-pair-on-macos).


```shell
ssh -i ~/.ssh/pi pi-admin@10.0.1.94
```


### Step 18: switch to root

```shell
sudo su -
```

### Step 19: disable root Bash history

```shell
echo "HISTFILESIZE=0" >> ~/.bashrc
history -c; history -w
source ~/.bashrc
```

### Step 20: disable pi sudo `nopassword` “feature”

```shell
rm /etc/sudoers.d/010_*
```

### Step 21: set root password

When asked for password, use output from `openssl rand -base64 24` (and store password in password manager).

```console
$ passwd
New password:
Retype new password:
passwd: password updated successfully
```

### Step 22: disable root login and password authentication

```shell
sed -i -E 's/^(#)?PermitRootLogin (prohibit-password|yes)/PermitRootLogin no/' /etc/ssh/sshd_config
sed -i -E 's/^(#)?PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
systemctl restart ssh
```

### Step 23: Disable Bluetooth and Wi-Fi

> Will take effect after reboot.

> Disable Bluetooth

```shell
echo "dtoverlay=disable-bt" >> /boot/config.txt
```

> Disable Wi-Fi (if using ethernet)

```shell
echo "dtoverlay=disable-wifi" >> /boot/config.txt
```

### Step 24: configure sysctl (if network is IPv4-only)

> Heads-up: only run following if network is IPv4-only.

```shell
cp /etc/sysctl.conf /etc/sysctl.conf.backup
cat << "EOF" >> /etc/sysctl.conf
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
EOF
sysctl -p
```

### Step 25: enable nftables and configure firewall rules

#### Enable nftables

```shell
systemctl enable nftables
systemctl start nftables
```

#### Configure firewall rules

```shell
nft flush ruleset
nft add table ip firewall
nft add chain ip firewall input { type filter hook input priority 0 \; policy drop \; }
nft add rule ip firewall input iif lo accept
nft add rule ip firewall input iif != lo ip daddr 127.0.0.0/8 drop
nft add rule ip firewall input tcp dport ssh accept
nft add rule ip firewall input ct state established,related accept
nft add chain ip firewall forward { type filter hook forward priority 0 \; policy drop \; }
nft add chain ip firewall output { type filter hook output priority 0 \; policy drop \; }
nft add rule ip firewall output oif lo accept
nft add rule ip firewall output tcp dport { http, https } accept
nft add rule ip firewall output udp dport { domain, ntp } accept
nft add rule ip firewall output ct state established,related accept
```

If network is IPv4-only, run:

```shell
nft add table ip6 firewall
nft add chain ip6 firewall input { type filter hook input priority 0 \; policy drop \; }
nft add chain ip6 firewall forward { type filter hook forward priority 0 \; policy drop \; }
nft add chain ip6 firewall output { type filter hook output priority 0 \; policy drop \; }
```

If network is dual stack (IPv4 + IPv6) run:

```shell
nft add table ip6 firewall
nft add chain ip6 firewall input { type filter hook input priority 0\; policy drop\; }
nft add rule ip6 firewall input iif lo accept
nft add rule ip6 firewall input iif != lo ip6 daddr ::1 drop
nft add rule ip6 firewall input meta l4proto ipv6-icmp icmpv6 type { destination-unreachable, packet-too-big, time-exceeded, parameter-problem } accept
nft add rule ip6 firewall input meta l4proto ipv6-icmp icmpv6 type { nd-router-advert, nd-neighbor-solicit, nd-neighbor-advert, nd-redirect } ip6 hoplimit 255 accept
nft add rule ip6 firewall input tcp dport ssh accept
nft add rule ip6 firewall input ct state established,related accept
nft add chain ip6 firewall forward { type filter hook forward priority 0\; policy drop\; }
nft add chain ip6 firewall output { type filter hook output priority 0\; policy drop\; }
nft add rule ip6 firewall output oif lo accept
nft add rule ip6 firewall output meta l4proto ipv6-icmp icmpv6 type { destination-unreachable, packet-too-big, time-exceeded, parameter-problem } accept
nft add rule ip6 firewall output meta l4proto ipv6-icmp icmpv6 type { nd-router-advert, nd-neighbor-solicit, nd-neighbor-advert } ip6 hoplimit 255 accept
nft add rule ip6 firewall output tcp dport { http, https } accept
nft add rule ip6 firewall output udp dport { domain, ntp } accept
nft add rule ip6 firewall output ct state related,established accept
```

### Step 26: log out and log in to confirm firewall is not blocking SSH

#### Log out

```console
$ exit

$ exit
```


#### Log in

> Heads-up: replace `10.0.1.94` with IP of Raspberry Pi.

```shell
ssh -i ~/.ssh/pi pi-admin@10.0.1.94
```

### Step 27: switch to root

```shell
sudo su -
```

### Step 28: make firewall rules persistent

```shell
cat << "EOF" > /etc/nftables.conf
#!/usr/sbin/nft -f

flush ruleset

EOF
```

```shell
nft list ruleset >> /etc/nftables.conf
```

### Step 29: set timezone

See https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
> Be sure NTP is working. Check status with `timedatectl` - make sure "NTP Service" is "active".

```shell
timedatectl set-timezone America/Montreal
```

### Step 30: disable swap

```shell
systemctl disable dphys-swapfile
```

### Step 31: update APT index and upgrade packages

```console
$ apt update

$ apt upgrade -y
```
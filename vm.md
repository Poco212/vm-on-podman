# 1. Partitioning

## physical volume

### disk layout
| disk | partition | type              | luks  | lvm   | label    | size      | format | mount                      |
| ---- | --------- | ----------------- | ----- | ----- | -------- | --------- | ------ | -------------------------- |
| 0    | 1         | efi               | false | false | boot     | 320M      | fat 32 | /boot                      |
| 0    | 4         | linux server data | true  | true  | data     | 100% Free | luks   | see logical layout point 1 |

gunakan cfdisk untuk membuat physical volume sesuai dengan guide line

### disk encryption
```
cryptsetup luksFormat --sector-size=4096 /dev/vda2
```

```
cryptsetup luksOpen /dev/vda2 proc
```

## logical volume

### disk layout
| partition | list | group  | name | size | mount                 | format |
| --------- | ---- | ------ | ---- | ---- | --------------------- | ------ |
| 2         | 1    | proc   | root | 5G   | /mnt                  | btrfs  |
| 2         | 2    | proc   | vars | 3G   | /mnt/var              | btrfs  |
| 2         | 3    | proc   | libs | 2G   | /mnt/var/usr/         | btrfs  |
| 2         | 4    | proc   | vlog | 2G   | /mnt/var/log/         | btrfs  |
| 2         | 5    | proc   | vaud | 1G   | /mnt/var/log/audit    | btrfs  |
| 2         | 6    | proc   | vtmp | 512M | /mnt/var/tmp/         | btrfs  |
| 2         | 7    | proc   | vpac | 2G   | /mnt/var/cache/pacman | btrfs  |
| 2         | 8    | proc   | home | 5G   | /mnt/home             | btrfs  |


```
mkfs.btrfs -L proc /dev/mapper/proc
```

```
mount /dev/mapper/proc /mnt
```

```
btrfs subvolume create /mnt/@
```

```
btrfs subvolume create /mnt/@vars
```

```
btrfs subvolume create /mnt/@libs
```

```
btrfs subvolume create /mnt/@vlog
```

```
btrfs subvolume create /mnt/@vaud
```

```
btrfs subvolume create /mnt/@vtmp
```

```
btrfs subvolume create /mnt/@tmp
```

```
btrfs subvolume create /mnt/@vpac
```

```
btrfs subvolume create /mnt/@home
```

```
umount -R /mnt
```

## root

```
mount subvol=@ /dev/mapper/proc /mnt
```

## var

```
mkdir -p /mnt/var
```

```
mount subvol=@vars,rw,nodev,noexec,nosuid,relatime /dev/mapper/proc /mnt
```

## boot

```
mkfs.vfat -F32 -n BOOT /dev/vda1
```

```
mkdir -p /mnt/boot
```

```
mount -o uid=0,gid=0,fmask=0077,dmask=0077 /dev/vda1 /mnt/boot
```

## libs

```
mkdir -p /mnt/var/usr
```

```
mount subvol=@libs,rw,nodev,noexec,nosuid,relatime /dev/mapper/proc /mnt/var/usr
```

## vlog

```
mkdir -p /mnt/var/log
```

```
mount subvol=@vlog,rw,nodev,noexec,nosuid,relatime /dev/mapper/proc /mnt/var/log
```

## vaud

```
mkdir -p /mnt/var/log/audit
```

```
mount subvol=@vaud,rw,nodev,noexec,nosuid,relatime /dev/mapper/proc /mnt/var/log/audit
```

## vtmp

```
mkdir -p /mnt/var/tmp
```

```
mount subvol=@vaud,rw,nodev,noexec,nosuid,relatime /dev/mapper/proc /mnt/var/tmp
```

## vpac

```
mkdir -p /mnt/var/cache/pacman
```

```
mount subvol=@vaud,rw,nodev,noexec,nosuid,relatime /dev/mapper/proc /mnt/var/cache/pacman
```

## home

```
mkdir -p /mnt/home
```

```
mount subvol=@vaud,rw,nodev,noexec,nosuid,relatime /dev/mapper/proc /mnt/home
```

## home

```
mkdir -p /mnt/tmp
```

```
mount subvol=@vaud,rw,nodev,noexec,nosuid,relatime /dev/mapper/proc /mnt/tmp
```

# 2.installation


## intel server
```
pacstrap /mnt linux-hardened linux-hardened-headers linux-firmware mkinitcpio clevis luksmeta tpm2-tools libpwquality cracklib git base neovim btrfs-progs openssh polkit ethtool iptables-nft firewalld apparmor rsync sudo debugedit fakeroot pkgconf bison gcc pcre flex wget make gcc curl irqbalance tuned which nginx --noconfirm
```

```
mkdir -p /mnt/etc/backup
```
## network configuration
```
cp /etc/systemd/network/* /mnt/etc/systemd/network/
```

## generate partition layout
```
genfstab -U /mnt > /mnt/etc/fstab
```


# 3. chrooting


```
arch-chroot /mnt
```

## hostname

```
echo 'oceansaw' > /etc/hostname
```


## local time

```
ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
```

```
hwclock --systohc
```
```
mkdir /etc/systemd/timesyncd.conf.d
```
```
nvim /etc/systemd/timesyncd.conf.d/local.conf
```
```
[Time]
NTP=0.id.pool.ntp.org 1.id.pool.ntp.org 2.id.pool.ntp.org 3.id.pool.ntp.org
FallbackNTP=time.cloudflare.com time.google.com time.aws.com
```
```
timedatectl set-ntp true
```
```
timedatectl set-timezone Asia/Jakarta
```
```
timedatectl status
```
```
timedatectl show-timesync --all
```
```
systemctl enable systemd-timesyncd.service
```

### locale

```
nvim /etc/locale.gen
```
uncommenting
```
en_US.UTF-8 UTF-8
en_US ISO-8859-1
```
```
locale-gen && locale > /etc/locale.conf
```
```
sed -i '1s/.*/LANG=en_US.UTF-8/' /etc/locale.conf
```
```
sed -i '14s/.*/LC_ALL=en_US.UTF-8/' /etc/locale.conf
```
```
cat /etc/locale.conf
```
```
touch /etc/vconsole.conf
```
```
nvim /etc/vconsole.conf
```
```
FONT=lat2-16
FONT_MAP=8859-2
```

### user
```
rm /etc/skel/.bash_profile
```
```
rm /etc/skel/.bashrc
```
```
rm /etc/skel/.bash_logout
```
```
echo 'loki ALL=(ALL:ALL) ALL' >> /etc/sudoers
```
```
cat /etc/sudoers
```
```
useradd -d /var/usr loki
```
```
chown -R loki:loki /var/usr
```
```
passwd loki
```
```
su loki
```
```
cd ~
```
```
mkdir /var/usr/.ssh
```
```
sudo su
```
```
exit
```
```
exit
```
```
su loki
```
```
cd ~
```
```
git clone https://aur.archlinux.org/mkinitcpio-systemd-extras.git
```
```
cd mkinitcpio-systemd-extras
```
```
makepkg -sri
```
```
cd ..
```
```
rm -fr mkinitcpio-systemd-extras
```
```
exit
```
```
sudo echo test | clevis encrypt tang '{"url":"http://172.27.1.10:13300"}' -y | clevis decrypt
```
```
clevis luks bind -d /dev/vda2 tang '{"url":"http://172.27.1.10:33000"}'
```
```
systemctl enable clevis-luks-askpass.path
```


## os release

```
echo '' > /usr/lib/os-release
```
```
nvim /usr/lib/os-release
```
```
NAME="Blackbird"
PRETTY_NAME="Blackbird"
ID=blackbird
BUILD_ID=rolling
ANSI_COLOR="38;2;23;147;209"
HOME_URL="https://blackbird.lektor.co.id/"
DOCUMENTATION_URL="https://blackbird.lektor.co.id/"
SUPPORT_URL="https://blackbird.lektor.co.id/support/"
BUG_REPORT_URL="https://gitlab.blackbird.org/groups/issues"
PRIVACY_POLICY_URL="https://blackbird.lektor.co.id/privacy-policy/"
LOGO=blackbird-logo
```

## package manager

```
nvim /etc/pacman.conf
```
```
SigLevel = Required DatabaseOptional TrustedOnly
```
uncomment
```
Color
VerbosePkgLists
```

## apparmor 

```
systemctl enable apparmor.service
```

## secure shell hardening

```
systemctl enable sshd
```


## kernels harden

```
nvim /etc/sysctl.d/30-secs.conf
```

```
## disable ipv6
net.ipv6.conf.all.disable_ipv6 = 1

# prevent the automatic loading of line disciplines
# https://lore.kernel.org/patchwork/patch/1034150
dev.tty.ldisc_autoload=0


# additional protections for fifos, hardlinks, regular files, and symlinks
# https://patchwork.kernel.org/patch/10244781
# slightly tightened up from the systemd default values of "1" for each
fs.protected_fifos=2
fs.protected_regular=2


## yama ptrac
## https://theprivacyguide1.github.io/linux_hardening_guide
kernel.yama.ptrace_scope=2


# prevents processes from creating new io_uring instances
# https://security.googleblog.com/2023/06/learnings-from-kctf-vrps-42-linux.html
kernel.io_uring_disabled=2


# disable unprivileged user namespaces
# https://lwn.net/Articles/673597
# (these two values are redundant, but not all kernels support the first one)
user.max_user_namespaces=0


# reverse path filtering to prevent some ip spoofing attacks
# (default in some distributions)
net.ipv4.conf.all.rp_filter=1
net.ipv4.conf.default.rp_filter=1


# reverse path filtering to prevent some ip spoofing attacks
# (default in some distributions)
net.ipv4.conf.all.rp_filter=1
net.ipv4.conf.default.rp_filter=1


# disable icmp redirects and RFC1620 shared media redirects
net.ipv4.conf.all.accept_redirects=0
net.ipv4.conf.all.secure_redirects=0
net.ipv4.conf.all.send_redirects=0
net.ipv4.conf.all.shared_media=0
net.ipv4.conf.default.accept_redirects=0
net.ipv4.conf.default.secure_redirects=0
net.ipv4.conf.default.send_redirects=0
net.ipv4.conf.default.shared_media=0
net.ipv6.conf.all.accept_redirects=0
net.ipv6.conf.default.accept_redirects=0


# disable tcp timestamps to avoid leaking some system information
# https://www.whonix.org/wiki/Disable_TCP_and_ICMP_Timestamps
net.ipv4.tcp_timestamps=0

# disable usb ( commenting when server had an external drive )
# kernel.deny_new_usb=1

#disable coredum
kernel.core_pattern=|/bin/false
```

## module hardening

### network
```
nvim /etc/modprobe.d/disable-network-protocols.conf
```
```
install dccp /bin/true
install sctp /bin/true
install rds /bin/true
install tipc /bin/true
install n-hdlc /bin/true
install ax25 /bin/true
install netrom /bin/true
install x25 /bin/true
install rose /bin/true
install decnet /bin/true
install econet /bin/true
install af_802154 /bin/true
install ipx /bin/true
install appletalk /bin/true
install psnap /bin/true
install p8023 /bin/true
install p8022 /bin/true
```

### filesystem
```
nvim /etc/modprobe.d/disable-filesystem-protocols.conf
```
```
install cramfs /bin/true
install freevxfs /bin/true
install jffs2 /bin/true
install hfs /bin/true
install hfsplus /bin/true
install squashfs /bin/true
install udf /bin/true
```


## loging config
```
mkdir -p /etc/systemd/journald.conf.d/
```
```
nvim /etc/systemd/journald.conf.d/01-default.conf
```
```
[Journal]
SystemMaxUse=1G
SystemKeepFree=500M
RuntimeMaxUse=200M
RuntimeKeepFree=50M
MaxFileSec=1month
Storage=persistent
```

## sleep config
```
mkdir -p /etc/systemd/sleep.conf.d/
```
```
nvim /etc/systemd/sleep.conf.d/01-blackbird.conf
```
```
[Sleep]
AllowSuspend=no
AllowHibernation=no
AllowHybridSleep=no
AllowSuspendThenHibernate=no
```

## coredump config
```
nvim /etc/systemd/coredump.conf
```
tambah di akhir document
```
Storage=none
ProcessSizeMax=0
```

## login sudoers
```
nvim /etc/sudo.conf
```
tambahkan pada bagian paling bawah
```
## Config Log
Defaults logfile="/var/log/sudo.log"
```

## autoupdate
```
nvim /etc/systemd/system/update.service
```
```
[Unit]
Description=Run system update

[Service]
Type=oneshot
ExecStart=/usr/bin/pacman --sync --refresh --sysupgrade --noconfirm
```
```
nvim /etc/systemd/system/update.timer
```
```
[Unit]
Description=Run the system update daily

[Timer]
OnCalendar=hourly
Persistent=true
Unit=update.service

[Install]
WantedBy=timers.target
```
```
systemctl enable update.timer 
```


### irqbalance
```
systemctl enable irqbalance
```

### tuned
```
systemctl enable tuned
```

### nginx
```
systemctl enable nginx
```
### network

```
systemctl enable systemd-networkd
```
```
systemctl enable systemd-resolved
```


### boot directory
#### intel server
```
rm /boot/initramfs-linux-hardened*
```

```
mkdir -p /boot/efi /boot/efi/linux /boot/efi/systemd /boot/efi/rescue /boot/efi/boot /boot/kernel
```


```
mv /boot/vmlinuz-linux-hardened /boot/kernel
```


### kernel parameter

```
mkdir /etc/cmdline.d
```
```
touch /etc/cmdline.d/{01-boot.conf,02-mods.conf,03-secs.conf,04-perf.conf,05-nets.conf,06-misc.conf}
```
### 01-boot
```
echo "rd.luks.name=UUID=$(blkid -s UUID -o value /dev/vda2)=proc root=/dev/mapper/cryptroot rootflags=subvol=@,rw rw" > /etc/cmdline.d/01-boot.conf
```

### 03-secs
```
echo "lsm=landlock,lockdown,yama,integrity,apparmor,bpf lockdown=integrity init_on_alloc=1 init_on_free=1 page_alloc.shuffle=1 slab_nomerge vsyscall=none randomize_kstack_offset=1" > /etc/cmdline.d/03-secs.conf
```

### 04-perf
```
echo "ipv6.disable=1" > /etc/cmdline.d/04-perf.conf
```
### 05-nets
```
echo "ip=10.10.1.40::10.10.1.1:255.255.255.0::eth0:none nameserver=10.10.1.1 nameserver=1.1.1.1 nameserver=8.8.8.8" > /etc/cmdline.d/05-nets.conf
```
```
nvim /etc/cmdline.d/05-nets.conf
```
lalu ubah ip address

### 06-misc

```
echo "rw quiet" > /etc/cmdline.d/06-misc.conf
```


### initram directory

```
rm -fr /etc/mkinitcpio.conf.d
```
```
mv /etc/mkinitcpio.conf /etc/mkinitcpio.d/default.conf
```
```
nvim /etc/mkinitcpio.d/default.conf
```
cari lalu commenting
```
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block filesystems fsck)
```
tambahkan
```
HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole sd-network sd-clevis sd-encrypt btrfs block filesystems fsck)
```
tambahkan pada bagian binaries
```
/usr/bin/curl
```

tambahkan pada bagian MODULES
```
virtio_pci
```
#### configure linux preset

```
nvim /etc/mkinitcpio.d/linux-hardened.preset
```
uncommenting 

```
#ALL_config="/etc/mkinitcpio.conf"
```
lalu ubah
```
ALL_config="/etc/mkinitcpio.d/default.conf"
```
edit
```
ALL_kver="/boot/vmlinuz-linux-hardened"
```
menjadi
```
ALL_kver="/boot/kernel/vmlinuz-linux-hardened" 
```
uncommenting
```
#default_uki="/efi/EFI/Linux/arch-linux-hardened.efi"
```
lalu ubah
```
default_uki="/boot/efi/linux/blackbird-hardened.efi"
```
commenting
```
fallback_image="/boot/initramfs-linux-hardened-fallback.img"
```
```
fallback_options="-S autodetect"
```
```
bootctl --path=/boot install
```
```
mkinitcpio -P
```


### instrusion detection

```
cd /tmp
```
```
pkg-config --libs --cflags glib-2.0
```
```
wget https://github.com/aide/aide/releases/download/v0.19.2/aide-0.19.2.tar.gz
```
```
tar xf aide-0.19.2.tar.gz
```
```
cd aide-0.19.2
```
```
./configure --with-zlib --with-posix-acl --with-xattr --with-curl --with-locale --with-syslog-ident --with-config-file=/etc/aide.conf
```
```
make && make install
```
```
nvim /etc/systemd/system/aide.service
```
```
[Unit]
Description=Aide Check
ConditionACPower=true

[Service]
Type=simple
ExecStart=/usr/local/bin/aide --check

[Install]
WantedBy=multi-user.target
```
```
nvim /etc/systemd/system/aide.timer
```
```
[Unit]
Description=Aide check every 8 Hours

[Timer]
OnCalendar=*:0/8:00
Unit=aide.service

[Install]
WantedBy=multi-user.target
```
```
systemctl enable aide.timer
```
```
mkdir -p /var/log/aide
```
```
mkdir -p /var/lib/aide
```
```
touch /var/log/aide/aide.log 
```
```
nvim /etc/aide.conf 
```
```
# Example configuration file for AIDE.
# More information about configuration options available in the aide.conf manpage.
# Inspired from https://src.fedoraproject.org/rpms/aide/raw/rawhide/f/aide.conf

# ┌───────────────────────────────────────────────────────────────┐
# │ CONTENTS OF aide.conf                                         │
# ├───────────────────────────────────────────────────────────────┘
# │
# ├──┐VARIABLES
# │  ├── DATABASE
# │  └── REPORT
# ├──┐RULES
# │  ├── LIST OF ATTRIBUTES
# │  ├── LIST OF CHECKSUMS
# │  └── AVAILABLE RULES
# ├──┐PATHS
# │  ├──┐EXCLUDED
# │  │  ├── ETC
# │  │  ├── USR
# │  │  └── VAR
# │  └──┐INCLUDED
# │     ├── ETC
# │     ├── USR
# │     ├── VAR
# │     └── OTHERS
# │
# └───────────────────────────────────────────────────────────────

# ################################################################ VARIABLES

# ################################ DATABASE

@@define DBDIR /var/lib/aide
@@define LOGDIR /var/log/aide

# The location of the database to be read.
database_in=file:@@{DBDIR}/aide.db.gz

# The location of the database to be written.
#database_out=sql:host:port:database:login_name:passwd:table
#database_out=file:aide.db.new
database_out=file:@@{DBDIR}/aide.db.new.gz

# Whether to gzip the output to database
gzip_dbout=yes

# ################################ REPORT

# Default.
log_level=warning
report_level=changed_attributes

report_url=file:@@{LOGDIR}/aide.log
report_url=stdout
#report_url=stderr
#NOT IMPLEMENTED report_url=mailto:root@foo.com
#NOT IMPLEMENTED report_url=syslog:LOG_AUTH

# ################################################################ RULES

# ################################ LIST OF ATTRIBUTES

# These are the default parameters we can check against.
#p:             permissions
#i:             inode:
#n:             number of links
#u:             user
#g:             group
#s:             size
#b:             block count
#m:             mtime
#a:             atime
#c:             ctime
#S:             check for growing size
#acl:           Access Control Lists
#selinux        SELinux security context (must be enabled at compilation time)
#xattrs:        Extended file attributes

# ################################ LIST OF CHECKSUMS

#md5:           md5 checksum
#sha1:          sha1 checksum
#sha256:        sha256 checksum
#sha512:        sha512 checksum
#rmd160:        rmd160 checksum
#tiger:         tiger checksum
#haval:         haval checksum (MHASH only)
#gost:          gost checksum (MHASH only)
#crc32:         crc32 checksum (MHASH only)
#whirlpool:     whirlpool checksum (MHASH only)

# ################################ AVAILABLE RULES

# These are the default rules
#R:             p+i+l+n+u+g+s+m+c+md5
#L:             p+i+l+n+u+g
#E:             Empty group
#>:             Growing logfile p+l+u+g+i+n+S

# You can create custom rules - my home made rule definition goes like this 
ALLXTRAHASHES = sha1+rmd160+sha256+sha512+whirlpool+tiger+haval+gost+crc32
ALLXTRAHASHES = sha1+rmd160+sha256+sha512+tiger
# Everything but access time (Ie. all changes)
EVERYTHING = R+ALLXTRAHASHES

# Sane, with multiple hashes
# NORMAL = R+rmd160+sha256+whirlpool
# NORMAL = R+sha256+sha512
NORMAL = p+i+l+n+u+g+s+m+c+sha256

# For directories, don't bother doing hashes
DIR = p+i+n+u+g+acl+xattrs

# Access control only
PERMS = p+i+u+g+acl

# Logfile are special, in that they often change
LOG = >

# Just do sha256 and sha512 hashes
FIPSR = p+i+n+u+g+s+m+c+acl+xattrs+sha256
LSPP = FIPSR+sha512

# Some files get updated automatically, so the inode/ctime/mtime change
# but we want to know when the data inside them changes
DATAONLY = p+n+u+g+s+acl+xattrs+sha256

# ################################################################ PATHS

# Next decide what directories/files you want in the database.

# ################################ EXCLUDED

# ################ ETC

# Ignore backup files
!/etc/.*~

# Ignore mtab
!/etc/mtab

# ################ USR

# These are too volatile
!/usr/src
!/usr/tmp

# ################ VAR

# Ignore logs
!/var/lib/pacman/.*
!/var/cache/.*
!/var/log/.*  
!/var/log/aide.log
!/var/run/.*  
!/var/spool/.*

# ################################ INCLUDED

# ################ ETC

# Check only permissions, inode, user and group for /etc, but cover some important files closely.
/etc                               PERMS
/etc/aliases                       FIPSR
/etc/at.allow                      FIPSR
/etc/at.deny                       FIPSR
/etc/audit/                        FIPSR
/etc/bash_completion.d/            NORMAL
/etc/bashrc                        NORMAL
/etc/cron.allow                    FIPSR
/etc/cron.daily/                   FIPSR
/etc/cron.deny                     FIPSR
/etc/cron.d/                       FIPSR
/etc/cron.hourly/                  FIPSR
/etc/cron.monthly/                 FIPSR
/etc/crontab                       FIPSR
/etc/cron.weekly/                  FIPSR
/etc/cups                          FIPSR
/etc/exports                       NORMAL
/etc/fstab                         NORMAL
/etc/group                         NORMAL
/etc/grub/                         FIPSR
/etc/gshadow                       NORMAL
/etc/hosts.allow                   NORMAL
/etc/hosts.deny                    NORMAL
/etc/hosts                         FIPSR
/etc/inittab                       FIPSR
/etc/issue                         FIPSR
/etc/issue.net                     FIPSR
/etc/ld.so.conf                    FIPSR
/etc/libaudit.conf                 FIPSR
/etc/localtime                     FIPSR
/etc/login.defs                    FIPSR
/etc/login.defs                    NORMAL
/etc/logrotate.d                   NORMAL
/etc/modprobe.conf                 FIPSR
/etc/nscd.conf                     NORMAL
/etc/pam.d                         FIPSR
/etc/passwd                        NORMAL
/etc/postfix                       FIPSR
/etc/profile.d/                    NORMAL
/etc/profile                       NORMAL
/etc/rc.d                          FIPSR
/etc/resolv.conf                   DATAONLY
/etc/securetty                     FIPSR
/etc/securetty                     NORMAL
/etc/security                      FIPSR
/etc/security/opasswd              NORMAL
/etc/shadow                        NORMAL
/etc/skel                          NORMAL
/etc/ssh/ssh_config                FIPSR
/etc/ssh/sshd_config               FIPSR
/etc/stunnel                       FIPSR
/etc/sudoers                       NORMAL
/etc/sysconfig                     FIPSR
/etc/sysctl.conf                   FIPSR
/etc/vsftpd.ftpusers               FIPSR
/etc/vsftpd                        FIPSR
/etc/X11/                          NORMAL
/etc/zlogin                        NORMAL
/etc/zlogout                       NORMAL
/etc/zprofile                      NORMAL
/etc/zshrc                         NORMAL

# ################ USR

/usr                               NORMAL
/usr/sbin/stunnel                  FIPSR

# ################ VAR

/var/log/faillog                   FIPSR
/var/log/lastlog                   FIPSR
/var/spool/at                      FIPSR
/var/spool/cron/root               FIPSR

# ################ OTHERS

/boot                              NORMAL
/bin                               NORMAL
/lib                               NORMAL
/lib64                             NORMAL
/opt                               NORMAL
/root                              NORMAL
```
```
aide --init
```
```
mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz
```
```
exit
```
```
umount -R /mnt
```
```
reboot
```
# 3. post instalation
```
passwd -l root
```

## firewall

```
systemctl enable firewalld
```

```
systemctl start firewalld
```

```
firewall-cmd --zone=internal --remove-service={dhcpv6-client mdns samba-client} --permanent
```
```
firewall-cmd --zone=public --remove-service=dhcpv6-client --permanent
```
```
firewall-cmd --zone=work --remove-service=dhcpv6-client --permanent
```
```
firewall-cmd --zone=home --remove-service={dhcpv6-client mdns samba-client} --permanent
```


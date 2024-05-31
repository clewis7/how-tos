# Installing `Arch Linux`

> **_NOTE:_**  I installed this on a 2024 Lenovo Yoga i9 for dual boot with Windows and Linux.

## Step 1: Shrink Windows SSD (`C:`) disk partition 
Can do this using Disk Manager GUI in Windows. Have 1TB of memory, so shrunk data chunk by about half to create a large file system for my Linux side. 

## Step 2: Create an Arch Linux installation image
Take a 32GB USB memory stick, download and Arch Linux ISO image, and burn the image using [Rufus](https://rufus.ie/en/) to create a bootable drive. 

## Step 3: Boot into live environment and install

### Step 3A: Verify boot mode
```
cat /sys/firmware/efi/fw_platform_size
```
Should return `64`.

### Step 3B: Connect to the internet
Use `iwctl` for wireless connection.
```
iwctl

# get devices
device list

# turn on device
device _name_ set-property Powered on

# turn on adapter
adapter _adapter_ set-property Powered on

# scan for networks
station _name_ scan

# get available networks
station _name_ get-networks

# connect to desired network
station _name_ connect _SSID_

# exit iwctl
exit
```

Check connection to internet by pinging `archlinux.org`
```
ping archlinux.org
```

### Step 3C: Check system clock
```
timedatectl
```

### Step 4D: Partition the disks
```
# list available disks
lsblk

# select correct disk to partition, `nvme0n1` here
fdisk /dev/nvme0n1

# create 1GB EFI system partition
n # new partition 
5 # partition number
'Enter' # default first segment number
+1G
t # change partition type
5 # partition to change
1 # change to "EFI System"

# create a 4GB Linux swap partition
n # new partition 
6 # partition number
'Enter' # default first segment number
+4G
t # change partition type
6 # partition to change
19 # change to "Linux Swap"

# use the remainder of the disk space for the root
n # new partition 
7 # partition number
'Enter' # default first segment number
'Enter' # default last segment number
t # change partition type
7 # partition to change
23 # change to "/root x86-64"

# write changes to disk
w
```

### Step 3E: Format the partitions
```
# add Ext4 file system to root partition
mkfs.ext4 /dev/nvme0n1p7

# initialize swap partition
mkswap /dev/nvme0n1p6

# initalize EFI partition
mkfs.fat -F 32 /dev/nvme0n1p5
```

### Step 3F: Mount the file systems
```
# mount the root partition
mount /dev/nvme0n1p7 /mnt

# mount the EFI system to /mnt/boot
mount --mkdir /dev/nvme0n1p5 /mnt/boot

# turn the swap on
swapon /dev/nvme0n1p6
```

### Step 3G: Install essential packages
```
pacstrap -K /mnt base linux linux-firmware
```

### Step 3H: Configure the system
```
# generate fstab
genfstab -U /mnt >> /mnt/etc/fstab

# change root and set time zone
arch-chroot /mnt

ln -sf /usr/share/zoneinfo/America/New_York /etc/localtime

hwclock --systohc

# install some things
pacman -S dhcpcd networkmanager iwd vim
pacman -S xorg plasma kde-applications

# localization
locale-gen

vim /etc/local.gen
# uncomment en_US.UTF-8 UTF-8

echo LANG=en_US.UTF-8 > /etc/locale.conf

# create hostname
echo caitlinlewis > /etc/hostname

# configure the network
####

# set the root password
passwd
```

### Step 3I: Install and configure a bootloader
```
# install grub and efibootmgr
pacman -S grub efibootmgr

# install grub packages to mounted /boot
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=LINUX

# make grub cfg
grub-mkconfig -o /boot/grub/grub.cfg
```

### Step 3J: Unmount and reboot
```
exit

umount -R /mnt

reboot
```

## Step 4: Connecting to the Internet
```
# setup interface
ip link set wlan0 up

# if error received check rfkill list to see if interface is blocked
rfkill
# if soft blocked
rfkill unblock wlan
# set up interface again using ip link

# check status of NetworkManager
systemctl status NetworkManager

# will likely need to be setup
systemctl restart NetworkManager

# after restarting, enable
systemctl enable NetworkManager

# check available wifi list
nmcli device wifi list

# connect to desired wifi
nmcli device wifi connect "LAN name" password LAN_password

# ping a website to check connection
ping archlinux.org
```

## Step 5: Creating a new user
```
useradd -m caitlin

# add password
passwd caitlin

# install sudo and nano
pacman -S sudo nano

# use nano as editor
export EDITOR=nano visudo

# add line to /etc/sudoers under 'User privilege specification`
caitlin ALL=(ALL) ALL
```

## Step 6: Installing plasma and kde
```
# install xorg, plamsa, and kde-applications
pacman -S xorg plasma kde-applications

# put in ~/.bash_profile
if [ -z "$DISPLAY" ] && [ "$XDG_VTNR" = 1 ]; then
  exec startx
fi

# start a session 
startx /usr/bin/startplasma-x11
```

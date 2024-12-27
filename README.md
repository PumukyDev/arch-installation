### Set the console keyboard layout

First we will start by setting the proper console keymap, the defaut console keymap is US, however if you need other keymap you can change it with the following command:

```bash
loadkeys <keymap>
```
e.g: 

```bash
loadkeys es
```

> [!NOTE]
> In order to see available layouts, just run: `localectl list-keymaps`


### Verify the boot mode

Before starting, check the content of `/sys/firmware/efi/fw_platform_size`


If you make:

```bash
cat /sys/firmware/efi/fw_platform_size
```

And it returns `64` then the system is booted in UEFI mode and you can follow this tutorial, otherwhise if the command retuns `32` the system is booted in BIOS

### Connect to the internet 

Ensure tour network interface is listed and enabled with `ip-link`:

```bash
ip link
```

1. lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT qlen 1000
    link/looback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2. enp3s0f3u3c2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether ab:cd:ef:gh:ij:kl brd ff:ff:ff:ff:ff:ff
4. wlan0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DORMANT group default qlen 1000
    link/ether ab:cd:ef:gh:ij:kl brd ff:ff:ff:ff:ff:ff


Connect to the network:

* Ethernet—plug in the cable.
* Wi-Fi—authenticate to the wireless network using iwctl.

```bash
iwlist wlan0 scan | more
```

```bash
systemctl start --now iwd.service
iwctl --passphrase 'Your_Router_Password' station wlan0 connect 'Your_Router_Name'
```

Verify the connection pinging google or archlinux.org

```bash
ping google.com
```
```bash
ping 8.8.8.8
```

```bash
ping archlinux.org
```

lsblk

NAME      MAJ:MIN   RM  SIZE    RO  TYPE    MOUNTPOINT
loop0       7:0     0   795.7M  1   loop    /run/archiso/airootfs
sda         8:0     1   29GB    0   disk
|-sda1      8:1     1   954M    0   part
|-sda2      8:2     1   165M    0   part
nvme0n1     259:0   0   1.8T    0   disk
................................


```bash
cfdisk /dev/nvme0n1
```


| Select lable type |
---------------------
| gpt               |
| dos               |
| sgi               |
| sun               |


Select you partition type, in my case GPT

You have now 5 options `New`(to create partitions) `Quit` (to quit cfdisk without writing changes) `Help` (to print some help) `Write` (to write partition table to disk) and `Dump` (? Dice "Dump partition table to sfdisk compatible script file")

We will start creating /boot

The official wiki has a tebla like this

| Mount point on the installed system | Partition | Partition type | Suggested size |
|----|----|----|--------
| /boot | /dev/efi_system_partition | Efi system partition | 1GiB |
| [SWAP] | /dev/swap_partition | Linux swap | At least 4 GiB
| / | /dev/root_partition | Linux x86-64 root (/) | Remainder of the device. At least 23-32 GiB|

https://www.gbmb.org/gib-to-gb

Click on `New` a type `1.073GB` (1GB)

Select it and click on `Type`, select EFI System.

Click on `New` a type `4.29GB` (4GB)

Select it and click on `Type`, select Linux swap

Click on `New` a type `34.36GB` (32GB)

Select it and click on `Type`, select Linux root (x86-64)

Click on `New` and click enter, the rest of the disk will be the home space. 

Click on `Write` and type yes


If you want to check the partitions, just run the following command:

```bash
fdisk -l /dev/nvme0n1
```

### Format the partitions

Once the partitions have been created, each newly created partition must be formatted with an appropriate file system. See File systems#Create a file system for details.

```bash
mkfs.fat -F 32 /dev/efi_system_partition
```

For example, to create an Ext4 file system on /dev/root_partition, run:

```bash
mkfs.ext4 /dev/root_partition
mkfs.ext4 /dev/home_partition
```

It will be probably /dev/sda3 or /dev/nvme0n3

If you created a partition for swap, initialize it with mkswap(8): 

```bash
mkswap /dev/swap_partition
```

It will be probably /dev/sda2 or /dev/nvme0n2d



### Mount the file systems



Mount the root volume to /mnt. For example, if the root volume is /dev/root_partition:

# mount /dev/root_partition /mnt

Create any remaining mount points under /mnt (such as /mnt/boot for /boot) and mount the volumes in their corresponding hierarchical order.
Tip: Run mount(8) with the --mkdir option to create the specified mount point. Alternatively, create it using mkdir(1) beforehand.

For UEFI systems, mount the EFI system partition:

```bash
mount --mkdir /dev/efi_system_partition /mnt/boot
```

If you created a swap volume, enable it with swapon:

```bash
swapon /dev/swap_partition
```



```bash
mkdir -p /mnt/boot /mnt/home

mount /dev/nvme0n1p1 /mnt/boot
mount /dev/nvme0n1p2 /mnt
mount /dev/nvme0n1p3 /mnt/home
swapon /dev/nvme0n1p4
```

## Installation

Packages to be installed must be downloaded from mirror servers, which are defined in /etc/pacman.d/mirrorlist. On the live system, after connecting to the internet, reflector updates the mirror list by choosing 20 most recently synchronized HTTPS mirrors and sorting them by download rate.

The higher a mirror is placed in the list, the more priority it is given when downloading a package. You may want to inspect the file to see if it is satisfactory. If it is not, edit the file accordingly, and move the geographically closest mirrors to the top of the list, although other criteria should be taken into account.

This file will later be copied to the new system by pacstrap, so it is worth getting right.
### Install essential packages
Note: No software or configuration (except for /etc/pacman.d/mirrorlist) gets carried over from the live environment to the installed system.

Use the pacstrap(8) script to install the base package, Linux kernel and firmware for common hardware: 

```bash
pacstrap -K /mnt base linux linux-firmware nano grub networkmanager efibootmgr
```

I hardly recommend you to install amd-ucode or intel-ucode too, they are CPU microcode hardware bug and security fixes


If you will use wifi later you need to install other 3 packages:

```bash
pacstrap -K /mnt netctl wpa_supplicant dialog
```

### Generate fstab

```bash
genfstab -U /mnt > /mnt/etc/fstab
```

You can check it with:

```bash
cat /mnt/etc/fstab
```

### Chroot

Change root into the new system:

```bash
arch-chroot /mnt
```

### Time

Set the time zone

```bash
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

```bash
ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime
```

To see timeszones, just run

timedatectl list-timazones | more



Run hwclock(8) to generate /etc/adjtime:

```bash
hwclock -w
```

This command assumes the hardware clock is set to UTC. See System time#Time standard for details.

To prevent clock drift and ensure accurate time, set up time synchronization using a Network Time Protocol (NTP) client such as systemd-timesyncd. 


### Localization


Edit `/etc/locale.gen` with nano and uncomment `en_US.UTF-8 UTF-8` and other needed UTF-8 locales (I will uncomment `es_ES.UTF-8 UTF-8`). Generate the locales by running:

```bash
locale-gen
```

Create the locale.conf(5) file, and set the LANG variable accordingly: 

```bash
echo KEYMAP=es > /etc/vconsole.conf
```


This will be the language

```bash
echo LANG=es_ES.UTF-8 > /etc/locale.conf
```

And add `LANG=en_US.UTF-8`

If you set the console keyboard layout, make the changes persistent in vconsole.conf(5): 








### Set pc name

echo PC_name > /etc/hostname

echo HP > /etc/hostname



### Root password

```bash
passwd
```

And type your new password 2 times






useradd -m adrian

passwd adrian





sudo mkdir -p /boot/efi
sudo mount /dev/sdXn /boot/efi


grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB

grub-mkconfig -o /boot/grub/grub.cfg








RESTART, QUIT PENDRIVE
exit
poweroff



log as root

ping 8.8.8.8

Network us unreachable

So we can to configure the connection wired/wifi

Wired

systemctl start NetworkManager
systemctl enable NetworkManager


ping 8.8.8.8


```bash
ip link
```

1. lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT qlen 1000
    link/looback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2. wlo1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DORMANT group default qlen 1000
    link/ether ab:cd:ef:gh:ij:kl brd ff:ff:ff:ff:ff:ff



 ip link set wlo1 up

 nmcli dev wifi connect 'Your_Router_Name' password 'Your_Router_Password'


ping 8.8.8.8


lspci | grep VGA
03:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Barcelo (rev c4)


Generic controllers

pacman -S xf86-video-vesa  


Then you can install a more specific controller, if you have a 
amd -> xf86-video-ati
nvidia -> xf86-video-nouveau
intel -> xf86-video-intel intel-ucode















make sudo commands as your user:

as root

pacman -S sudo
usermod -aG wheel user



## Export nano as your default editor
echo "export EDITOR=nano" >> ~/.bashrc
source ~/.bashrc



sudo visudo




scroll down


search
# %wheel ALL=(ALL:ALL) ALL

uncomment it and save























instalar paru

su -l user
sudo pacman -S base-devel git rust

mkdir -p desktop/repos
cd desktop/repos
git clone https://aur.archlinux.org/paru.git
cd paru
makepkg -si

tarda mucho!




sudo pacman -S xorg-server xorg-xinit mesa mesa-demos

sudo pacman -S lightdm

sudo systemctl start lightdm
sudo systemctl enable lightdm

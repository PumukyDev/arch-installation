### Set the console keyboard layout

The defaut console keymap is US, however if you need other keymap you can change it with the following command:

```bash
loadkeys <keymap>
```


In order to see available layouts, just run: `localectl list-keymaps`

e.g: 

```bash
loadkeys es
```

### Verify the boot mode

Before starting, check the content of `/sys/firmware/efi/fw_platform_size`


If you make:

```bash
cat /sys/firmware/efi/fw_platform_size
```

Anf it returns `64`then the system us booted in UEFI mode and you can follow this tutorial, otherwhise if the command retuns `32`the system is booted in BIOS

### Connect to the internet 

Ensure tour network interface is listed and enabled with `ip-link`:

```bash
ip link
```

Connect to the network:

* Ethernet—plug in the cable.
* Wi-Fi—authenticate to the wireless network using iwctl.

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


### Format the partitions

Once the partitions have been created, each newly created partition must be formatted with an appropriate file system. See File systems#Create a file system for details.

For example, to create an Ext4 file system on /dev/root_partition, run:

```bash
mkfs.ext4 /dev/root_partition
```

It will be probably /dev/sda3 or /dev/nvme0n3

If you created a partition for swap, initialize it with mkswap(8): 

```bash
mkswap /dev/swap_partition
```

It will be probably /dev/sda2 or /dev/nvme0n2d

```bash
mkfs.fat -F 32 /dev/efi_system_partition
```

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

## Installation

Packages to be installed must be downloaded from mirror servers, which are defined in /etc/pacman.d/mirrorlist. On the live system, after connecting to the internet, reflector updates the mirror list by choosing 20 most recently synchronized HTTPS mirrors and sorting them by download rate.

The higher a mirror is placed in the list, the more priority it is given when downloading a package. You may want to inspect the file to see if it is satisfactory. If it is not, edit the file accordingly, and move the geographically closest mirrors to the top of the list, although other criteria should be taken into account.

This file will later be copied to the new system by pacstrap, so it is worth getting right.
### Install essential packages
Note: No software or configuration (except for /etc/pacman.d/mirrorlist) gets carried over from the live environment to the installed system.

Use the pacstrap(8) script to install the base package, Linux kernel and firmware for common hardware: 

```bash
pacstrap -K /mnt base linux linux-firmware
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

Run hwclock(8) to generate /etc/adjtime:

```bash
hwclock --systohc
```

This command assumes the hardware clock is set to UTC. See System time#Time standard for details.

To prevent clock drift and ensure accurate time, set up time synchronization using a Network Time Protocol (NTP) client such as systemd-timesyncd. 

### Localization

```bash
pacman -S nano
```

Edit `/etc/locale.gen` with nano and uncomment `en_US.UTF-8 UTF-8` and other needed UTF-8 locales (I will uncomment `es_ES.UTF-8 UTF-8`). Generate the locales by running:

```bash
locale-gen
```

Create the locale.conf(5) file, and set the LANG variable accordingly: 

```bash
nano /etc/locale.conf
```

And add `LANG=en_US.UTF-8`

If you set the console keyboard layout, make the changes persistent in vconsole.conf(5): 
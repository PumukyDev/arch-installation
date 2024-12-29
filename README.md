# ARCH LINUX INSTALLATION GUIDE

Arch Linux has its own [official installation guide](https://wiki.archlinux.org/title/Installation_guide), however I made this tutorial to create a more customized Arch installation experience.

## Getting started

### Why Arch Linux

Arch Linux is a [GNU](https://wiki.archlinux.org/title/GNU)/Linux distribution known for its flexibility, and **rolling release model**. It provides users with the latest software and updates without the need for major system upgrades. Arch’s minimalist approach allows you to build a system tailored exactly to your needs, **avoiding unnecessary bloat**. The distribution offers **excellent documentation** through the [Arch Wiki](https://wiki.archlinux.org/title/Main_page), which is widely regarded as one of the best resources for Linux users. Arch also features the powerful [Pacman](https://wiki.archlinux.org/title/Pacman) package manager and access to the [Arch User Repository (AUR)](https://aur.archlinux.org/), giving you an extensive library of software. With bleeding-edge updates, a **highly customizable system**, and a strong focus on user control, Arch Linux is ideal for those who want a **lightweight**, **efficient**, Linux experience.





















$${\color{#ff5a5a}root}$$
$${\color{red}Red}$$
$${\color{red}Red}$$
$${\color{green}Green}$$
$${\color{green}Green}$$
$${\color{lightgreen}Light \space Green}$$ 	
$${\color{lightgreen}Light \space Green}$$
$${\color{blue}Blue}$$ 	
$${\color{blue}Blue}$$
$${\color{lightblue}Light \space Blue}$$ 	
$${\color{lightblue}Light \space Blue}$$
$${\color{black}Black}$$
$${\color{black}Black}$$
$${\color{white}White}$$
$${\color{white}White}$$
$${\color{red}Welcome \space \color{lightblue}To \space \color{orange}Stackoverflow}$$
$${\color{red}Welcome \space \color{lightblue}To \space \color{orange}Stackoverflow}$$

> [!NOTE]
> Highlights information that users should take into account, even when skimming.

> [!TIP]
> Optional information to help a user be more successful.

> [!IMPORTANT]
> Crucial information necessary for users to succeed.

> [!WARNING]
> Critical content demanding immediate user attention due to potential risks.

> [!CAUTION]
> Negative potential consequences of an action.













## PREPARATION

## INSTALLATION

Once 

### Set the keyboard layout

By default, the keyboard layout is set to **US**. You can see available layouts with:

<dl><dd>
<pre>
root@archiso ~ # <b>localectl list-keymaps</b>
</pre>
</dd></dl>

If you require a different layout, you can change it using the following command:

<dl><dd>
<pre>
root@archiso ~ # <b>loadkeys <i>keymap</i></b>
</pre>
</dd></dl>

For example, to set the layout to Spanish:

<dl><dd>
<pre>
root@archiso ~ # <b>loadkeys es</b>
</pre>
</dd></dl>





### Verify the boot mode

Make sure you are running in UEFI mode with the following command:

<dl><dd>
<pre>
root@archiso ~ # <b>cat /sys/firmware/efi/fw_platform_size</b>
</pre>
</dd></dl>

If it returns `64`, the system is booted in UEFI mode, and you can proceed with the tutorial. Otherwise, if the command returns `32`, the system is booted in BIOS (Legacy) mode.





### Connect to the internet 

To install Arch Linux, you will need an internet connection. Let's begin by checking the available network interfaces:

<dl><dd>
<pre>
root@archiso ~ # <b>ip link</b>
</pre>
</dd></dl>

You should see something like this:

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT qlen 1000
    link/looback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp3s0f3u3c2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether ab:cd:ef:gh:ij:kl brd ff:ff:ff:ff:ff:ff
4: wlan0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DORMANT group default qlen 1000
    link/ether ab:cd:ef:gh:ij:kl brd ff:ff:ff:ff:ff:ff
```
The first interface is the **loopback interface**, which is essential and should always be present. After that, you may have **more or fewer interfaces depending on your computer hardware**.

If you have an interface named **enpX or ethX**, it is an **Ethernet** interface. If you have one named **wlanX or wloX**, it is a **Wi-Fi** network card.

In order to establish a connection, you need to configure one of the following options. **Choose one based on your needs**.


<details>
    <summary><b>Wired Connection - ethX or enpX</b></summary><br/>

To establish a connection with a physical network interface, simply connect the Ethernet cable to the Ethernet port, no additional configuration is needed.

</details>

<details>
    <summary><b>Wireless Connection - wlan0 or wlo0</b></summary><br/>

To establish a connection with a Wi-Fi card, you need to **start the [iwd (iNet Wireless Daemon)](https://wiki.archlinux.org/title/Iwd) service**:

<dl><dd>
<pre>
root@archiso ~ # <b>systemctl start --now iwd.service</b>
</pre>
</dd></dl>

Then, you can connect to your **router** with the following command:

<dl><dd>
<pre>
root@archiso ~ # <b>iwctl --passphrase <i>'Your_Router_Password'</i> station wlan0 connect <i>'Your_Router_Name'</i></b>
</pre>
</dd></dl>

> [!TIP]
> If you don't know your router name (weird), run the following command:
><dl><dd>
><pre>
>root@archiso ~ # <b>iwlist wlan0 scan | more</b>
></pre>
></dd></dl>

</details>


Verify the connection by pinging Google or archlinux.org. One of them is enough.

<dl><dd>
<pre>
root@archiso ~ # <b>ping google.com</b>
root@archiso ~ # <b>ping 8.8.8.8</b>
root@archiso ~ # <b>ping archlinux.org</b>
</pre>
</dd></dl>





### Partition the disks

Disks are assigned to a **block device** such as `/dev/sdX`, `/dev/nvmeXnY`. To identify these devices use the following command:

<dl><dd>
<pre>
root@archiso ~ # <b>lsblk</b>
</pre>
</dd></dl>

You will see an output similar to this:

```
NAME     MAJ:MIN  RM    SIZE  RO  TYPE  MOUNTPOINT
loop0      7:0     0  795.7M   1  loop  /run/archiso/airootfs
sda        8:0     1    29GB   0  disk
|-sda1     8:1     1    954M   0  part
|-sda2     8:2     1    165M   0  part
nvme0n1  259:0     0    1.8T   0  disk
```

In the example you can see `sda`, which is the flash device. The other device is a 2TB NVMe where I will install Arch Linux, that's why it's called `nvme0n1`. If you have more devices thay can be called sdb, sdc, etc or nvme1n1, nvme2n1, and so on. 

Results ending in rom, loop or airootfs may be ignored. mmcblk* devices ending in rpbm, boot0 and boot1 can be ignored. 

> [!NOTE]
> Here is a short explanation of each column:
> NAME: Name of the block device (e.g., sda, nvme0n1, loop0).
> MAJ:MIN: Major and minor device numbers identifying the device.
> RM: Removable (1 for removable, 0 for fixed).
> SIZE: Size of the device or partition.
> RO: Read-only status (1 for read-only, 0 for read/write).
> TYPE: Type of device (disk, part, or loop).
> MOUNTPOINT: Where the device is mounted, if applicable.

Now that we know the name of the disk where we want to install Arch Linux, we can partition it:

<dl><dd>
<pre>
root@archiso ~ # <b>cfdisk <i>/dev/disk_to_partition</i></b>
</pre>
</dd></dl>

Here is an example if your disk is called `sdb`:
<dl><dd>
<pre>
root@archiso ~ # <b>cfdisk <i>/dev/sdb</i></b>
</pre>
</dd></dl>

Or if you have an `nvme` as me:

<dl><dd>
<pre>
root@archiso ~ # <b>cfdisk <i>/dev/nvme0n1</i></b>
</pre>
</dd></dl>

A semigraphical tool will appear.

It will ask you to select a partition type. Press <kbd>Return</kbd> on `gpt`

| Select lable type |
|-------------------|
| gpt               |
| dos               |
| sgi               |
| sun               |

You now have 5 options:
* `New`: to create partitions
* `Quit`: to quit cfdisk without saving changes
* `Help`: to display help
* `Write`: to write the partition table to disk
* `Dump`: to dump the partition table to a sfdisk-compatible script file

You are free to make the partitions as you wish. The official wiki provides a table like this:

| Mount point on the installed system |         Partition         |    Partition type     | Suggested size |
|-------------------------------------|---------------------------|-----------------------|----------------|
|               /boot                 | /dev/efi_system_partition | Efi system partition  |      1GiB      |
|               [SWAP]                |    /dev/swap_partition    |      Linux swap       | At least 4 GiB |
|                 /                   | /dev/root_partition       | Linux x86-64 root (/) | Remainder of the device. At least 23-32 GiB|


Apart from the partitions in the table above, I'm going to make a home partition for my user

[!TIP]
To navigate between partitions and options, use the arrow keys: <kbd>&#8592;</kbd> <kbd>&#8593;</kbd> <kbd>&#8595;</kbd> <kbd>&#8594;</kbd>

#### 1. /boot

Press <kbd>Return</kbd> on `New` and type `1GB`, then press <kbd>Return</kbd> again.

Navigate to the newly created partition, press <kbd>Return</kbd> on `Type`, and select `EFI System`.

#### 2. [swap]

Press <kbd>Return</kbd> on `New` and type double the amount of your RAM in GB. For example, if you have 6GB of RAM, type `12GB`, then press <kbd>Return</kbd> again.

Navigate to the newly created partition, press <kbd>Return</kbd> on `Type`, and select `Linux swap`.

#### 3. /

Press <kbd>Return</kbd> on `New` and enter the amount of GB you want for your root partition, for example, `250GB`, then press <kbd>Return</kbd> again.

Navigate to the newly created partition, press <kbd>Return</kbd> on `Type`, and select `Linux root (x86-64)`.

#### 4. /home

Press <kbd>Return</kbd> on `New`, and to use the entire disk, press <kbd>Return</kbd> again.

Navigate to the newly created partition, press <kbd>Return</kbd> on `Type`, and select `Linux home`.

#### 5. Write changes

Press <kbd>Return</kbd> on `Write` and type `yes`

#### 6. Check partitions

To check the partitions, simply run the following command:

<dl><dd>
<pre>
root@archiso ~ # <b>fdisk -l <i>/dev/partitioned_disk</i></b>
</pre>
</dd></dl>





### Format the partitions

Once the partitions have been created, each newly created partition must be formatted with an appropriate file system. 

#### 1. /boot

<dl><dd>
<pre>
root@archiso ~ # <b>mkfs.fat -F 32 <i>/dev/efi_system_partition</i></b>
</pre>
</dd></dl>

<details>
    <summary style="font-size: 8px"><b>Examples</b></summary><br/>

<dl><dd>
<pre>
root@archiso ~ # <b>mkfs.fat -F 32 <i>/dev/sdb1</i></b>
</pre>
</dd></dl>

<dl><dd>
<pre>
root@archiso ~ # <b>mkfs.fat -F 32 <i>/dev/nvme0n1p1</i></b>
</pre>
</dd></dl>

</details>



#### [swap]

<dl><dd>
<pre>
root@archiso ~ # <b>mkswap <i>/dev/swap_partition</i></b>
</pre>
</dd></dl>

<details>
    <summary style="font-size: 8px"><b>Examples</b></summary><br/>

<dl><dd>
<pre>
root@archiso ~ # <b>mkswap <i>/dev/sdb2</i></b>
</pre>
</dd></dl>

<dl><dd>
<pre>
root@archiso ~ # <b>mkswap <i>/dev/nvme0n1p2</i></b>
</pre>
</dd></dl>

</details>



#### /

<dl><dd>
<pre>
root@archiso ~ # <b>mkfs.ext4 <i>/dev/root_partition</i></b>
</pre>
</dd></dl>

<details>
    <summary style="font-size: 8px"><b>Examples</b></summary><br/>

<dl><dd>
<pre>
root@archiso ~ # <b>mkfs.ext4 <i>/dev/sdb3</i></b>
</pre>
</dd></dl>

<dl><dd>
<pre>
root@archiso ~ # <b>mkfs.ext4 <i>/dev/nvme0n1p3</i></b>
</pre>
</dd></dl>

</details>



#### /home

<dl><dd>
<pre>
root@archiso ~ # <b>mkfs.ext4 <i>/dev/home_partition</i></b>
</pre>
</dd></dl>

<details>
    <summary style="font-size: 8px"><b>Examples</b></summary><br/>

<dl><dd>
<pre>
root@archiso ~ # <b>mkfs.ext4 <i>/dev/sdb4</i></b>
</pre>
</dd></dl>

<dl><dd>
<pre>
root@archiso ~ # <b>mkfs.ext4 <i>/dev/nvme0n1p4</i></b>
</pre>
</dd></dl>

</details>





### Mount the file systems

#### 1. /boot

<dl><dd>
<pre>
root@archiso ~ # <b>mount --mkdir <i>/dev/efi_system_partition</i> /mnt/boot/efi</b>
</pre>
</dd></dl>

<details>
    <summary style="font-size: 8px"><b>Examples</b></summary><br/>

<dl><dd>
<pre>
root@archiso ~ # <b>mount --mkdir <i>/dev/sdb1</i> /mnt/boot/efi</b>
</pre>
</dd></dl>

<dl><dd>
<pre>
root@archiso ~ # <b>mount --mkdir <i>/dev/nvme0n1p1</i> /mnt/boot/efi</b>
</pre>
</dd></dl>

</details>



#### [swap]

<dl><dd>
<pre>
root@archiso ~ # <b>swapon <i>/dev/swap_partition</i></b>
</pre>
</dd></dl>

<details>
    <summary style="font-size: 8px"><b>Examples</b></summary><br/>

<dl><dd>
<pre>
root@archiso ~ # <b>swapon <i>/dev/sdb2</i></b>
</pre>
</dd></dl>

<dl><dd>
<pre>
root@archiso ~ # <b>swapon <i>/dev/nvme0n1p2</i></b>
</pre>
</dd></dl>

</details>



#### /

<dl><dd>
<pre>
root@archiso ~ # <b>mount <i>/dev/root_partition</i> /mnt</b>
</pre>
</dd></dl>

<details>
    <summary style="font-size: 8px"><b>Examples</b></summary><br/>

<dl><dd>
<pre>
root@archiso ~ # <b>mount <i>/dev/sdb3</i> /mnt</b>
</pre>
</dd></dl>

<dl><dd>
<pre>
root@archiso ~ # <b>mount <i>/dev/nvme0n1p3</i> /mnt</b>
</pre>
</dd></dl>

</details>



#### /home

<dl><dd>
<pre>
root@archiso ~ # <b>mount --mkdir <i>/dev/home_partition</i> /mnt/home</b>
</pre>
</dd></dl>

<details>
    <summary style="font-size: 8px"><b>Examples</b></summary><br/>

<dl><dd>
<pre>
root@archiso ~ # <b>mount --mkdir <i>/dev/sdb4</i> /mnt/home</b>
</pre>
</dd></dl>

<dl><dd>
<pre>
root@archiso ~ # <b>mount --mkdir <i>/dev/nvme0n1p4</i> /mnt/home</b>
</pre>
</dd></dl>

</details>





## Installation

Packages are downloaded from mirrors listed in `/etc/pacman.d/mirrorlist`. After connecting to the internet, the reflector tool updates the list by selecting the 20 most recent HTTPS mirrors and sorting them by download speed.

### Install essential packages

Use the `pacstrap` script to install the base package

<dl><dd>
<pre>
root@archiso ~ # <b>pacstrap -K /mnt base linux linux-firmware nano grub networkmanager efibootmgr</b>
</pre>
</dd></dl>

If you plan to use Wi-Fi later, you will need to install three additional packages:

<dl><dd>
<pre>
root@archiso ~ # <b>pacstrap -K /mnt netctl wpa_supplicant dialog</b>
</pre>
</dd></dl>

> [!IMPORTANT]
> I strongly recommend installing `amd-ucode` or `intel-ucode` as well. These packages provide CPU microcode updates for hardware bugs and security fixes. Just add `amd-ucode` or `intel-ucode` to the `pacstrap` installation.





### Generate fstab

We have to generate `fstab`, which is used to define how disk partitions are mounted during **system startup**. The genfstab tool automatically detects the partitions mounted on /mnt and writes them to /mnt/etc/fstab, allowing the system to mount them correctly on **boot**.

<dl><dd>
<pre>
root@archiso ~ # <b>genfstab -U /mnt > /mnt/etc/fstab</b>
</pre>
</dd></dl>





### Chroot

Change root into the new system:

<dl><dd>
<pre>
root@archiso ~ # <b>arch-chroot /mnt</b>
</pre>
</dd></dl>





### Time

To set the time zone, you first need to list the available time zones. Run the following command to see the available options:

<dl><dd>
<pre>
[root@archiso /]# <b>timedatectl list-timezones | more</b>
</pre>
</dd></dl>

Then, create a symbolic link to the correct time zone for your region and city:

<dl><dd>
<pre>
[root@archiso /]# <b>ln -sf /usr/share/zoneinfo/<i>Region</i>/<i>City</i> /etc/localtime</b>
</pre>
</dd></dl>

For example, to set the time zone to Europe/Madrid:

<dl><dd>
<pre>
[root@archiso /]# <b>ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime</b>
</pre>
</dd></dl>


Run the following command to generate the `/etc/adjtime` file, which is used to store hardware clock settings:

<dl><dd>
<pre>
[root@archiso /]# <b>hwclock -w</b>
</pre>
</dd></dl>





### Locale and Keymap


To set the system locale, edit the `/etc/locale.gen` file with nano and **uncomment** `en_US.UTF-8` and any other necessary UTF-8 locales. For example, I'll uncomment `es_ES.UTF-8`.

<dl><dd>
<pre>
[root@archiso /]# <b>nano /etc/locale.gens</b>
</pre>
</dd></dl>

> [!TIP]
> You can find them easily with <kbd>CTRL</kbd> + <kbd>W</kbd>

After uncommenting the needed locales, generate them by running:

<dl><dd>
<pre>
[root@archiso /]# <b>locale-gen</b>
</pre>
</dd></dl>

You should see an output similar to:

```
Generation locales...
    en_US.UTF-8 ... done
    xx_XX.UTF-8 ... done
Generation complete.
```
Next, set your preferred system language by saving it in `etc/locale.conf`:

<dl><dd>
<pre>
[root@archiso /]# <b>echo LANG=en_US.UTF-8 > /etc/locale.conf</b>
</pre>
</dd></dl>

Then, set the console keymap according to your preference and save it in `/etc/vconsole.conf`. For example:

For **English (US)** keymap:

<dl><dd>
<pre>
[root@archiso /]# <b>echo KEYMAP=en > /etc/vconsole.conf</b>
</pre>
</dd></dl>

For **Spanish** keymap:

<dl><dd>
<pre>
[root@archiso /]# <b>echo KEYMAP=es > /etc/vconsole.conf</b>
</pre>
</dd></dl>





### Set Hostname

To set the hostname of your system, use the following command:

<dl><dd>
<pre>
[root@archiso /]# <b>echo <i>hostname</i> > /etc/hostname</b>
</pre>
</dd></dl>

For example, to set the hostname to HP:

<dl><dd>
<pre>
[root@archiso /]# <b>echo HP > /etc/hostname</b>
</pre>
</dd></dl>





### Root password

To set the root password, run the following command:

<dl><dd>
<pre>
[root@archiso /]# <b>passwd</b>
</pre>
</dd></dl>





### Create a User

To create a new user, run the following command

<dl><dd>
<pre>
[root@archiso /]# <b>useradd -m <i>user</i></b>
</pre>
</dd></dl>

For example, to create a user named adrian:

<dl><dd>
<pre>
[root@archiso /]# <b>useradd -m adrian</b>
</pre>
</dd></dl>

To set a password for it user, run:

<dl><dd>
<pre>
[root@archiso /]# <b>passwd <i>user</i></b>
</pre>
</dd></dl>

For example, to set a password for adrian

<dl><dd>
<pre>
[root@archiso /]# <b>passwd adrian</b>
</pre>
</dd></dl>





### Grub

???
mkdir -p /boot/efi
mount /dev/sdXn /boot/efi

you will see
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
???

To install the GRUB bootloader, run the following command:

<dl><dd>
<pre>
[root@archiso /]# <b>grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB</b>
</pre>
</dd></dl>

If the installation finishes without any issues, generate the GRUB configuration file with:

<dl><dd>
<pre>
[root@archiso /]# <b>grub-mkconfig -o /boot/grub/grub.cfg</b>
</pre>
</dd></dl>





### Exit

If you followed the steps correctly, congratulations! You have finished installing Arch Linux. To check if everything is working, follow these steps:

1. Exit from chroot
<dl><dd>
<pre>
[root@archiso /]# <b>exit</b>
</pre>
</dd></dl>

2. Turn of the computer
<dl><dd>
<pre>
root@archiso ~ # <b>poweroff</b>
</pre>
</dd></dl>

3. Remove the installation media (flash drive).
4. Turn on your computer.

Upon rebooting, you should see the GRUB menu. After selecting Arch Linux, you will be taken to the tty1 terminal, where we will finish the final configuration of your system.

Log in as `root` and check that you can execute commands as `whoami`or `ls` for example





### Connect to the internet

After rebooting we have to configure the network for a last time. If you try pinging google for example you can see that you can't even if you have a wired connection:

<dl><dd>
<pre>
root@hostname ~ # <b>ping 8.8.8.8</b>
Network is unreachable
</pre>
</dd></dl>

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




sudo pacman -S xterm xorg-server xorg-xinit mesa mesa-demos

sudo pacman -S lightdm lightdm-gtk-greeter


sudo systemctl enable lightdm

sudo pacman -S qtile




sudo pacman -S alacritty feh xclip fastfetch openssh picom code stow wireless_tools python-psutils python-iwlib rofi kitty fish bat dunst arandr pulseaudio pamixer pulseaudio-alsa pulseaudio-jack alsa-utils pamixer 

paru -S python-pulsectl-asyncio
paru -S qtile-extras


systemctl --user enable pulseaudio

lo de arriba es muy importante que sea sin sudo!

 firefox



if it does not work make sure that everything inside your userfolder is yours, if not:

sudo chown -R user:user /home/user


copy the fonts
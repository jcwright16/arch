# Arch Linux Manual Install Guide

## Create Bootable Media

1. Download an iso image from archlinux.org
2. Download usbimager
3. Run usbimager and select the iso file.
4. Choose the usb device to be made into bootable media.
5. Click "write" and enter password. The device will be written to.

## Configure BIOS and Reboot

Enter the BIOS setup and disble secure boot on the device. Restart the device with the bootable media inserted. The installation environment should boot.

## Connect to a Network

Find your device name by entering `ip addr show`. My device is wlan0.

Enter `iwctl`. This should open a command prompt.

To view available networks, enter `station <device name> get-networks`

To connect to the network by entering `station <device name> connect <network name>`. If there are spaces in the network name, enter the name in quotes. You should be prompted to enter the network password.

Enter `exit`.

To check the connection, run `ping archlinux.org`. The terminal should show a response. Type Ctrl+C to continue.

## Configure Disk

To get the device id for the drive you want to install Arch on, run `lsblk`. This command can be used throughout the process to show the disk layout, and to check where file paths have been mounted. Make sure to select your hard drive and not your bootable media. As a way to differentiate, the hard drive should have more memory than the bootable media. My drive is sda.

> Write Random Data [Optional]

    #This is an optional step for added security. It can take several hours to complete. 

    dd if=/dev/urandom of=/dev/<device id> status=progress bs=4096

Create partitions on the disk. We will create three here. Run `gdisk /dev/<device id>`. This will open a command prompt. You can view the current partition table using the `p` command.

Here we will be running a clean install. Replace the existing partition table with a new one using the `o` command. Then enter `n` to create your first new partition. After creating the new partion, you will be prompted to enter a series of values: the partition id, the first sector, the last sector, and the partition code, which sets the type of partition you are creating. For our three partitions, use the following values (blank bullet points indicate accepting the default):

* 1
* 
* +1G
* ef00

* 2
* 
* +4G
* ef02

* 3
* 
* 
* 8309

You should now have an EFI system partition, a BIOS boot partition, and a Linux LUKS partition, respectively.

## Encryption and Configuring Partitions

Load the encryption modules.

    modprobe dm-crypt
    modprobe dm-mod

Set up encryption on our luks lvm partition.

    cryptsetup luksFormat -v -s 512 -h sha512 /dev/<partion 3>

Mount the drive.

    cryptsetup open /dev/<partition 3> luks_lvm

Create volumes and volume groups.

    pvcreate /dev/mapper/luks_lvm
    vgcreate arch /dev/mapper/luks_lvm

Create swap space. A good size is your RAM + 2GB.
For example, if you have 64GB of RAM, make your swap size 66GB.

    lvcreate -n swap -L 66G arch

Create root and home volumes. The below commands will create a root volume with 200GB and use the remaining space for the home volume.

    lvcreate -n root -L 200G arch
    lvcreate -n home -l +100%FREE arch

Create the filesystems.

    mkfs.fat -F32 /dev/<partition 1>
    mkfs.fat ext4 /dev/<partition 2>
    mkfs.btrfs -L root /dev/mapper/arch-root
    mkfs.btrfs -L home /dev/mapper/arch-home

Set up swap device

    mkswap /dev/mapper/arch-swap

## Mounting

Mount swap

    swapon /dev/mapper/arch-swap
    swapon -a

Mount root

    mount /dev/mapper/arch-root /mnt

Create home and boot

    mkdir -p /mnt/{home,boot}

Mount the boot partition

    mount /dev/mapper/arch-home /mnt/home

Create the EFI directory

    mkdir /mnt/boot/efi

Mount the EFI directory

    mount /dev/<partition 1> /mnt/boot/efi

## Install Arch

    pacstrap -K /mnt base base-devel linux linux-firmware

Load the file table 

    genfstab -U -p /mnt > /mnt/etc/fstab

chroot into your installation

    arch-chroot /mnt /bin/bash

## Configuring

### Text Editor
Install a text editor (I prefer neovim, but you can use nano or another text editor of your choice).

    pacman -S neovim

### Decrypting Volumes
Open up mkinitcpio.conf

    nvim /etc/mkinitcpio.conf

add `encrypt` and `lvm2` into the hooks

    HOOKS=(... block encrypt lvm2 filesystems fsck)

install lvm2

    pacman -S lvm2

### Bootloader
Install grub and eifbootmgr

    pacman -S grub efi bootmgr

Set up grub on efi partition

    grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub --removable

Obtain your lvm partition device UUID and copy it.

    blkid /dev/<partition 3>

Open the grub file.

    nvim /etc/default/grub

Add the following kernel parameters.

    rootwait root=/dev/mapper/arch-root cryptdevice=UUID=<uuid>:luks_lvm

### Keyfile

    mkdir /secure

Root keyfile

    dd if=/dev/random of=/secure/root_keyfile.bin bs=512 count=8

Change permissions

    chmod 000 /secure/*

Add to partitions

    cryptsetup luksAddKey /dev/<partition 3> /secure/root_keyfile.bin

    nvim /etc/mkinitcpio.conf

    FILES=(/secure/root_keyfile.bin)

### Grub
Create grub config

    grub-mkconfig -o /boot/grub/grub.cfg
    grub-mkconfig -o /boot/efi/EFI/arch/grub.cfg

## System Configuration

### Timezone

    ln -sf /usr/share/zoneinfo/America/Indianapolis /etc/localtime

### NTP

    nvim /etc/systemd/timesyncd.conf

Add in the NTP servers

    [Time]
    NTP=0.arch.pool.ntp.org 1.arch.pool.ntp.org 2.arch.pool.ntp.org 3.arch.pool.ntp.org
    FallbackNTP=0.pool.ntp.org 1.pool.ntp.org

Enable timesyncd

    systemctl enable systemd-timesyncd.service

## Locale

    nvim /etc/locale.gen

Uncomment the UTF8 lang you want

    en_US.UTF-8 UTF-8

    locale-gen

    nvim /etc/locale.conf

    LANG=en_US.UTF-8

### Hostname

    echo "<machinename>" > /etc/hostname

### Create users

Secure root user with a password

    passwd

Install preferred shell

    pacman -S bash

Add a new user

    useradd -m -G wheel -s /bin/bash <username>

Set user password

    passwd <username>

Add wheen group to sudoers

    EDITOR=nvim visudo

Uncomment

`%wheel ALL=(ALL:ALL) ALL`

### Network Connectivity

    pacman -S networkmanager
    systemctl enable NetworkManager

### Display Manager

Choose a desktop environment and install it along with a display manager. Below are examples for Gnome, Cinnamon, and KDE Plasma.

    pacman -S gnome
    systemctl enable gdm

    pacman -S cinnamon lightdm lightdm-gtk-greeter
    systemctl enable lightdm

    pacman -S xorg plasma plasma-wayland-session kde-applications
    systemctl enable sddm.service
    systemctl enable NetworkManager.service

### Microcode

For AMD

    pacman -S amd-ucode

For Intel

    pacman -S intel -ucode

    grub-mkconfig -o /boot/grub/grub.cfg
    grub-mkconfig -o /boot/efi/EFI/arch/grub.cfg

## Reboot

    exit
    umount -R /mnt
    reboot now

Remove the bootable media as the machine is restarting. You should be prompted to log in.
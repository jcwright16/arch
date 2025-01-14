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
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


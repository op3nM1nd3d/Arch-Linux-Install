# Arch installation from Live Bootable Disk


## Check if you booted into uefi mode

`ls /sys/firmware/efi/efivars`

_if this directory exists then you are otherwise, you’re in bios_

## Connect to a network

_Run this to check the names of your interfaces “wl” ones are wireless “e” is ethernet (e.g. wlp2s0)_

`ip link`

`wifi-menu -o`		_(-o: obscure mode)_

## Set ntp and check if its active

`timedatectl set-ntp true`

`timedatectl status`

## Partitioning disk for UEFI mode

`cfdisk -z`

_Now use “gpt” on the dialog box and at the next screen create 3 partitions (or more)_
1. /dev/sdX1 for EFI System
2. /dev/sdX2 for Linux SWAP
3. /dev/sdX3 for Linux Filesystem

## Format the partitions and mount root and efi partitions

`mkfs.fat -F32 /dev/sdX1`

`mkfs.ext4 /dev/sdX3`

`mkswap /dev/sdX2`

`swapon /dev/sdX2`

_Replace X with whatever is your device name, most probably sda_

## Mounting the partitions

`mount /dev/sdX3 /mnt`

_Everything else is mounted within this directory, this will become / after we chroot_

## Edit the mirrorlist and pacman.conf files (recommended)

`nano /etc/pacman.d/mirrorlist`

_When the file opens just uncomment mirrors closest to your location_

`nano /etc/pacman.conf`

_When the file opens find [multilib] block and uncomment that_

## Now install the base packages

`pacstrap -i /mnt base base-devel`

`nano /etc/pacman.d/mirrorlist`

## Generate “fstab” file 

`genfstab -U /mnt >> /mnt/etc/fstab`

## Change to root (basically now we are in the newly installed environment)

`arch-chroot /mnt /bin/bash`

## Set correct timezone and sync hardware clock

`ln -sf /usr/share/zoneinfo/your_Region/your_City /etc/localtime`

`hwclock –systohc`

## Edit “locale.gen” file to pick locale

`nano /etc/locale.gen`

_Uncomment “en_US.UTF-8” or any other UTF-8 locale save+exit and run:-_

`locale-gen`

## Create “locale.conf” file or just echo the locale you chose 

`echo LANG=en_US.UTF-8 > /etc/locale.conf`

## Network Configuration and hostname

`nano /etc/hostname`

_Now just enter the name you want to give to your PC and save+exit_

_Create another file “hosts” with contents_

`nano /etc/hosts`

_Content_

>127.0.0.1 &nbsp;&nbsp;&nbsp;&nbsp; localhost

>::1 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; localhost

>127.0.1.1 &nbsp;&nbsp;&nbsp;&nbsp; your_hostname.localdomain your_hostname 

## Install some packages for network management 

`pacman -S networkmanager wireless_tools iw wpa_supplicant dialog`

_pacman is the package manager we'll use from now on_

## Install bootloader

`pacman -S grub efibootmgr intel-ucode`

_intel-ucode is for intel processors, there's one for amd as well, look it up_

## Mount the boot partition now

`mkdir /boot/efi`

`mount /dev/sdX1 /boot/efi/`

## Now generate “grubx64.efi” file

`grub-install –target=x86_64-efi –bootloader-id=GRUB –efi-directory=/boot/efi/`

## Generate config file for grub

`grub-mkconfig -o /boot/grub/grub.cfg`

## Exit this environment by running

`exit`

## Editing “fstab” file to include boot partition

_Find UUID and mounting point with following commands_

`lsblk`

`blkid`

_Now open the file and add the info you just found_

`vim /mnt/etc/fstab`

_you can use nano instead of vim editor_

_Add these line_

>\# /dev/sdX1

>UUID=????-????	mount_point	vfat	defaults,noatime	0 2

_Replace ? marks with UUID you found with blkid command_

## Now unmount all partitions under “/mnt”

`umount -R /mnt`

## Now reboot the system and while system is rebooting remove the installation media(usb etc.)

`reboot`   

## Set root password 

_Once system reboots, you’ll be in tty screen just type “root” and hit enter_ 

`passwd`

_enter the password when prompted_

## Create a new user and setup a password 

`useradd -m -g users -G wheel -s /bin/bash your_user_name`

`passwd your_user_name`

## Give your user proper access

`exit`

_login as root_

`EDITOR=nano visudo`

_Find “%wheel ALL=(ALL) ALL” line and uncomment it_

_After this log back in as your newly created user_

# Minimal Install Finished!!!
# Now we have two options
*	Desktop Environment:

_We can install a DE like (GNOME 3, KDE Plasma, Cinnamon etc.)_
*	Window Manager:

_We can opt for a much more minimalistic approach with WMs but setting them up takes some time._ 
_They are extremely light on RAM usage._

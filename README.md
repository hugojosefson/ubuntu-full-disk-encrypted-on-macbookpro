# Full-disk encrypted Ubuntu, alongside MacOSX

If you, like me, prefer to use Ubuntu on any hardware you use, this is for you.

These are instructions on how to install Ubuntu 14.04 or later with full disk encryption, alongside MacOSX on a MacBook Pro.

This has been specifically tested on a MacBook Pro 10,1 (mid 2012) with the following OS combinations:

  * Ubuntu GNOME 15.10 (Wily), alongside OSX 10.11 (El Capitan)
  * Ubuntu 14.10 (Utopic), alongside OSX 10.10 (Yosemite)
  * Ubuntu 14.10 (Utopic), alongside OSX 10.9 (Mavericks)
  * Ubuntu 14.04 LTS (Trusty), alongside OSX 10.10 (Yosemite)
  * Ubuntu 14.04 LTS (Trusty), alongside OSX 10.9 (Mavericks)
 
It should work the same on many more Apple computers and OS versions.

_With much help from this: https://bugs.launchpad.net/ubuntu/+source/cryptsetup/+bug/1237556_


## Preparations in OSX

Boot up your existing OSX and run Disk Utility.

Resize your existing OSX partition to make room for Ubuntu. Leave as much space as you want for Ubuntu, including your RAM size for swap. Remember that OSX counts gigabytes the way hard drive manufacturers do, and I and Ubuntu count the way computers do :) So make up to 10 % more room than it looks like you'll get, to be on the safe side.

### Encrypt OSX (recommended)

If you want your shrunken OSX partition encrypted too, do that now with File Vault in System Preferences. If you try it after having installed Ubuntu alongside, File Vault will probably not let you. Let it finish encrypting and optimizing. Then reboot to check that everything is in order so far.

### Create bootable USB

Create an Ubuntu install USB using a 64bit image of your choice. Official instructions: [How to create a bootable USB stick on OS X](http://www.ubuntu.com/download/desktop/create-a-usb-stick-on-mac-osx). Most Ubuntu images can be downloaded from [cdimage.ubuntu.com](http://cdimage.ubuntu.com/). I prefer the latest release of Ubuntu GNOME.

## Preparations in Ubuntu

Boot up the Ubuntu install USB.

Start `gparted` and create one small partition with ext4 labelled `/boot`. Create another large one, for encryption.

Lets say the boot partition happens to be `/dev/sda4` and the large encryption partition is `/dev/sda5`.

In a terminal, issue these commands:

    sudo cryptsetup luksFormat /dev/sda5
    sudo cryptsetup luksOpen /dev/sda5 the_encrypted_stuff

    sudo pvcreate /dev/mapper/the_encrypted_stuff
    sudo vgcreate VG-encrypted /dev/mapper/the_encrypted_stuff

    sudo lvcreate --name LV-swap --size 16G VG-encrypted     # Or however much RAM you have

    sudo vgdisplay VG-encrypted                              # Take note of available number of free extents
    sudo lvcreate --name LV-root --extents ... VG-encrypted  # Using number of free extents instead of ...


## Install

Start the installer.

When it asks about erasing installations, installing alongside, or something else, choose "Something else".

Use `/dev/sda4` for `/boot`, the encrypted `LV-root` for `/`, and the encrypted `LV-swap` for swap.

When the installation is finished, DO NOT CLICK Continue, and DO NOT CLICK Reboot!


## Fix boot

### cryptsetup

In a terminal, issue these commands:

    sudo blkid | grep ^/dev/sda5                                  # Take note of the UUID="..." string

    for i in /dev /dev/pts /proc /sys /run; do sudo mount -B $i /target$i; done
    sudo chroot /target

    echo the_encrypted_stuff UUID=... none luks > /etc/crypttab   # Using the UUID from above instead of ...
    update-initramfs -u -k all
    lsinitramfs /boot/initrd* | grep cryptsetup

You should see that `cryptsetup` is indeed included in the initrd image you just built.

Leave the chroot:

    exit

### refind

To enable dual-boot, you can install `refind`:

	sudo apt-add-repository ppa:rodsmith/refind
	sudo apt-get update
	sudo apt-get install -y efibootmgr refind
	
This should get your hard drive (SSD?!) set up for letting you choose operating system on boot.

## Done!

Go ahead; reboot and enjoy!

You might want to check out my [ubuntu-install-scripts](https://github.com/hugojosefson/ubuntu-install-scripts) to get off to a running start, once you're inside your new Ubuntu installation.

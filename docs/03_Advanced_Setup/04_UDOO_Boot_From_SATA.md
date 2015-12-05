This guide will show you how to boot your UDOO from an attached SATA drive with the help of a small SD card. NOTE: It is currently not possible to boot without an SD card, it is hoped that this feature will be added in the future.

## Prerequisites

* A SATA drive
* A SD card
* A spare PC
* A Micro USB cable such as one for charging a smart phone

This guide also assumes you have a working Linux install for UDOO.

## Preparing the drive

You may partition and format the drive however you want, the current builds of U-Boot support up to ext3, newer builds may support ext4. A single ext3 partition will be used as an example. There are several methods of preparing the SATA drive, some are listed below. NOTE: you only need to do ONE of the following methods.

* Using an existing Linux system
* Extracting a FS tarball
* Using dd to flash an image
* Using win32diskimager from windows

### Using an existing Linux system

Connect your SD and SATA drive to a Linux system. During this write up, a USB SATA adapter was used and mounted with the SD at /usb and /sd  for clarity, but this is up to you.

Adjust commands appropriately.

```bash

# cp -rvp /sd/* /usb/

```

This will copy your SD filesystem to the SATA disk.

Unmount both. Once that is complete, connect the SATA disk to the UDOO and reinsert the SD card.

### Extracting a FS tarball

You may use the filesystem tarball from the binaries tab on [udoo.org](http://www.udoo.org/downloads/) or use your own. Place this tarball on the SATA disk. You can transfer this over the network or wget the file straight onto it.

Run the following commands (Assuming /dev/sda1 is mounted at /mnt):

```bash

# cd /mnt
# tar xvzf tarball_you_just_grabbed.tar

```

### Using dd to flash an image

You can also flash the filesystem using DD. You will need a filesytem image such as the ones provided by UDOO. Make sure your SATA disk is NOT mounted.

Flash by doing the following:

```bash

# dd if=path/to/fs.img of=/dev/sdb

```

### Using win32diskimager from windows

You can flash any image from windows with win32diskimager. You can get it from its SourceForge page [http://sourceforge.net/projects/win32diskimager/](http://sourceforge.net/projects/win32diskimager/).

<a name="prepareuboot"></a>

## [Prepare U-Boot](#prepareuboot)

This is a temporary guide, since a new version of u-boot with a reorganization of the environment variables is scheduled and will be released asap.

You will need to connect the UDOO to a PC with a micro USB cable. Make sure to use the micro USB port closest to the corner (CN6).

Then open a serial terminal to the new COM port on your PC with a baud of 115200.

Reset the UDOO and press any key over serial when prompted to cancel the autoboot. If you miss the prompt, you can press reset on the UDOO or run the reboot command to reboot.

With the SATA hard drive connected run the command:

```bash

sata part

```

It will display some information:

```bash

Partition Map for SATA device 0  --   Partition Type: DOS
Part    Start Sector    Num Sectors     UUID            Type
 1        16065           14185395     000c356e-01       83

```

Take note of the device n° and Part n° information.

Now run the following commands:

```bash

setenv bootcmd "if run loadbootscript; then run bootscript; else run sataboot; run mmcboot; run netboot; fi;"

setenv mmcloaduimage ext2load mmc ${mmcdev}:${mmcpart} ${loadaddr} ${uimage}

setenv mmcboot "if mmc rescan; then echo Booting from mmc ...; run mmcloaduimage; run mmcargs; bootm; else mmc boot failed; fi;"

setenv satadev <device n° previously noted e.g. setenv satadev 0>

setenv satapart <part n° previously noted e.g. setenv satapart 1>

setenv sataroot /dev/sda${satapart} rootwait rw

setenv sataargs setenv bootargs console=${console},${baudrate} root=${sataroot} ${hdmi_patch} fbmem=24M video=mxcfb0:dev=hdmi,1920x1080M@60,bpp=32

setenv sataloaduimage ext2load sata ${satadev}:${satapart} ${loadaddr} ${uimage}

setenv sataboot "if sata init; then echo Booting from sata ...; run sataloaduimage; run sataargs; bootm; else sata boot failed; fi;"

saveenv

```

In this way, if you have a connected SATA hard drive, the boot will be from SATA, otherwise it will occur from SD card.

Now run the following to continue booting:

```bash

boot

```

## Done!

You should now be booting into the system on the SATA drive. If all goes well you can safely remove almost everything on the SD card.

## NOTE!

If you used the udooupdate script to update the U-Boot version and kernel, you'll probably need to do this on both the SD card that triggers the boot as well as the SATA drive. Use the instructions here to change which filesystem is the root, download and run the update from each.

DO the SATA drive FIRST!!!!

If you only do the SATA drive, you probably will not be able to reboot into the SATA drive until you also update the SD card to match.

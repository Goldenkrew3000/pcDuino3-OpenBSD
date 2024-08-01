# OpenBSD on a pcDuino3 (Sunxi A20 Board)
Note: This guide is working with OpenBSD 7.5 and U-Boot 2024.10 <br>

## Step 1 - U-Boot Preparation
P.S. Host system is Debian 12 amd64 <br>
Clone the U-Boot Source tree, and install gcc-none-eabi (ArmV7 GCC Cross compiler) <br>
The default configuration works fine for this, although does not have NAND support by default (#1) <br>
So basically the only commands required are: <br>
```
CROSS_COMPILE=arm-none-eabi- make Linksprite_pcDuino3_defconfig
CROSS_COMPILE=arm-none-eabi- make -j12
```
And the required file is: u-boot-sunxi-with-spl.bin

## Step 2 - Prepare the Device Tree
OpenBSD does not utilize any device tree from the Rootfs, even if you install the 'dtb' package. It purely uses the FDT that U-Boot provides. <br>
You are able to use the board without a proper device tree, but ethernet (and probably more stuff) do not work. <br>
Also with this board, the ethernet (and in linux, HDMI) do not work properly with the pcDuino3 dtb, you need to use the pcDuino3-Nano dtb. <br>
Anyway, so either get the device tree (Labeled pcDuino3 but is actually the pcDuino3-Nano dtb) from this repository or extract it from the 'dtb' package in the OpenBSD repository. <br>
The reason it is renamed is due to what U-Boot expects the filename to be. You could also modify this in U-Boot instead but renaming is easier. <br>
Note: Ways of installing OpenBSD without a functional ethernet port are to (a) use a USB ethernet card or (b) Create a 4.4BSD UFS formatted USB with another install of OpenBSD, and put the required tarballs onto it.

## Step 3 - Prepare Boot Media
Requirements: An MMC card that is 8+ GB (But not more than 32GB, it seems to hang when formatting a 128GB card) <br>
Download the ArmV7 Rootfs: https://ftp.openbsd.org/pub/OpenBSD/7.5/armv7/miniroot-am335x-75.img <br>
Now run the following commands to flash the install media to the MMC card. <br>
```
sudo dd if=miniroot-am335x-75.img of=/dev/sdx bs=1M
sudo dd if=u-boot-sunxi-with-spl.bin of=/dev/sdx bs=1024 seek=8
```
Now mount the 1st partition (/dev/sdx1) and copy the device tree, named EXACTLY 'sun7i-a20-pcduino3.dtb', to the root of the partition.

## Step 4 - Boot into OpenBSD
Now connect a USB TTL Serial converter to the DEBUG pins. The board rate is 115200 bps and the board voltage is 3.3v. <br>
The board should autoboot into the OpenBSD installer without any help. <br>
Note: The $loadaddr is 0x42000000 <br>
Note: You can load an EFI File from external media by the following: <br>
```
load usb 0 0x42000000 /file.efi
bootefi 0x42000000
```

## Step 5 - Install OpenBSD
This should be easy. Networking (ethernet is dwge0) should just work with the proper device tree. <br>
The only stage here that needs help is after the install when it boots back into U-Boot, it has created a new EFI partition, and therefore has deleted the device tree. <br>
All you have to do is copy that device tree file to the root of the EFI partition again, and U-Boot should automatically load it on boot.

## Step 6 - Success
![alt text](https://github.com/goldenkrew3000/pcduino3-openbsd/neofetch.png?raw=true)
Note: The board says that it's a pcDuino3 Nano, but this does not matter since the difference between the two is extremely minimal.

## Notes
#1 - U-Boot and NAND Support <br>
The original U-Boot install (on the nand) does not have MMC support, and the default U-Boot configuration as of 2024.10 does not have NAND support enabled for the pcDuino3. It might be possible to enable it as easily as turning on a configuration option but I am not sure. <br>
P.S. The Sunxi A20 SOCs connect to their flash IC directly with data lines, no SPI / I2C involved.

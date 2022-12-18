# Introduction to STM32 and Yocto

## STM32
+ Reference: https://www.youtube.com/watch?v=4H5eOOuPVyY
    - Overview:
        - STM32MP1 is the first step for STM to go to microprocessor systems!
            * STM32MP1 is build with an ARM Cortex M4
            * Build around the stm32 ecosystem, so the code that is already developed for the STM32 can be reused under the stm32mp1's linux OS
        - Very high connectivity --> PWM, Ethernet, ...
        - GPU with Linux support
        - Yocto environment
        - Focus on extensibility
        - There are chips for the connectivity (ethernet, audio-jack/audio-codec, power management) and the STM32MP157 itself
            * On the top of the board are raspberry pi GPIOs
            * On the bottom of the board are ardurino gpio connectors
    - Software development
        - To develop, we need a native or virtual linux machine (e.g. Ubuntu 20.04 LTS)
+ Getting started:
    - Reference: https://forum.digikey.com/t/debian-getting-started-with-the-stm32mp157/12459

# Setup
+ Reference for custom setup: https://forum.digikey.com/t/debian-getting-started-with-the-stm32mp157/12459
+ Setup with the official STM guide:

https://wiki.st.com/stm32mpu/wiki/Getting_started/STM32MP1_boards/STM32MP157x-DK2

## Activating the cross compiler
+ Download the cross compiler (ideally one folder above the root of this repository):
	- `$ wget -c https://mirrors.edge.kernel.org/pub/tools/crosstool/files/bin/x86_64/11.3.0/x86_64-gcc-11.3.0-nolibc-arm-linux-gnueabi.tar.xz
` 
+ Unpack the downloaded cross compiler
	- `$ tar -xf x86_64-gcc-11.3.0-nolibc-arm-linux-gnueabi.tar.xz
`
+ Export the path to CC to the cross compiler
	- `$ export CC=`pwd`/gcc-11.3.0-nolibc/arm-linux-gnueabi/bin/arm-linux-gnueabi-
`
+ Check if everything went as expected:
	- `$ ${CC}gcc --version
`
## Further requirements
+ You need to have the device tree compiler `dtc` installed on your system!
	- `$ sudo apt install device-tree-compiler`
+ Furter to compile the u-boot bootloader, you need its dependencies installed: `sudo apt install build-essential swig
`
+ To compile the kernel, you need to have the unpacking command `$ lzop` installed on your system (`$ sudo apt install lzop`), as well as the following packages:
	- `$ sudo apt-get install lzma gettext libmpc-dev u-boot-tools libncurses5-dev:amd64 
`
+ To be able to flash the linux image, firmware and bootloader to the SD card, we need `sgdisk` to format the drive

## Compiling the bootloader, firmware and linux kernel:
+ While we compile the bootloader, firmware and the linux kernel, we need to have `${CC}` exported to the cross compiler, to compile the code for the targeted device! --> Before you compile something, export `${CC}gcc --version` to the cross compiler

## Burning the linux system to the micro SD card
+ ***Tipp: Use a fixed, exported variable for the targeted device file to avoid typos that can damage your machine!***

+ To reset/overwrite the partition table of an SD card, we need to write `0x0` to every memory location of the former partition table!
    - Do this with `$ sudo dd if=/dev/zero of=${DISK} bs=1M count=10`
        * `dd`: _Disk dump_ - copies data on byte basis, which means it ignores partitions or file systems while copying
        * `/dev/zero` is a special linux device file that returns as many `0x0` values as you read from it
        * `${DISK}` is determined by `$ lsblk` and then `$ export DISK=/dev/sdb` (or whatever the device file is that the external drive is set to)
        * `bs`: Blocks with the given size are read/write by dd 
        * `count`: Defines how many blocks of the size of `bs` should be written with the given data 
+ To create a bootable setup, we use a GPT (_GUID Partition Table_) as the partition table for our system (in contrast to the deprecated MBR - _Master Boot Record_, which we do not use!)
    - Partition tables explained: https://www.youtube.com/watch?v=ZOjicsRClic
    - GPT properties:
        * Hardware needs to support UEFI (unified extended firmware interface)
    - Delete the old partition table: 
        * `$ sudo sgdisk -o ${DISK}`
            - Clear all partition data on the given disk - to avoid damage to your drive, you need to first wipe all data on it with the `-Z` option or with the command `$ sudo dd if=/dev/zero of=${DISK} bs=1M count=10`, like it is shown above
    - Create new partition table:
    ```
    $ sudo sgdisk --resize-table=128 -a 1 \
            -n 1:34:545    -c 1:fsbl1   \
            -n 2:546:1057  -c 2:fsbl2   \
            -n 3:1058:5153 -c 3:ssbl    \
            -n 4:5154:     -c 4:rootfs  \
            -p ${DISK}
    ```
	* After this, you need to reboot your host system in order to let it detect the newly created partitions on the flash drive
        * Where do these partition settings come from?
		- https://www.st.com/content/ccc/resource/training/technical/product_training/group1/ac/45/53/56/1f/0b/47/68/STM32MP1-Software-Platform_boot_BOOT/files/STM32MP1-Software-Platform_boot_BOOT.pdf/_jcr_content/translations/en.STM32MP1-Software-Platform_boot_BOOT.pdf page 32
		- See https://www.youtube.com/watch?v=KU8_HMfpv0s for explaination
	* `-a`: Setting the start of the partition sectors to this value
        * `-n`: Stands for new partition with a `partition_number:start_sector:end_sector` specified after it
        * `-c`: Change the name of the aforementioned partition number
        * `-p`: Print partition summary to terminal
+ Also, we need to set the legacy BIOS flag to true, to make the rootfs bootable (even if its not a legacy boot on it)
	* sudo sgdisk -A 4:set:2 ${DISK}
+ After this, we need to hardcopy the firmware and the bootloader to the partitions 1, 2 and 3 (1 and 2 are the same, since the STM32MP1 needs two arm trusted firmware partitions, since the second is the fallback firmware)
	* `sudo dd if=./arm-trusted-firmware/build/stm32mp1/release/tf-a-stm32mp157a-dk1.stm32 of=${DISK}1`
	* `sudo dd if=./arm-trusted-firmware/build/stm32mp1/release/tf-a-stm32mp157a-dk1.stm32 of=${DISK}2`
	* `sudo dd if=./u-boot/u-boot.stm32 of=${DISK}3`
### Creating the linux root filesystem and install the kernel on it
+ At first, we need to format the _rootfs_ partition to ext4, which is needed for the linux filesystem and kernel
	* `sudo mkfs.ext4 -L rootfs ${DISK}4`
+ Mount the rootfs to the host machine:
	* `sudo mount ${DISK}4 /media/rootfs/`
+ Copy the filesystem to the disk
	* `sudo tar xfvp ./debian-*-*-armhf-*/armhf-rootfs-*.tar -C /media/rootfs/`
		- `-x`: extract
		- `-f`: use this file to extract
		- `-v`: verbose - more terminal printout
		- `-p`: also extract information about file permissions - if you do not use this, it defaults to root user

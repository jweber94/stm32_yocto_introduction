# Device Tree Patch with Yocto
+ This tutorial is based on this youtube video: https://www.youtube.com/watch?v=srM6u8e4tyw&list=PLEBQazB0HUyTpoJoZecRK6PpDG31Y7RPB&index=6
    - The written tutorial can be found here: https://www.digikey.com/en/maker/projects/intro-to-embedded-linux-part-3-flash-sd-card/a1a20abb6b144b008310a69b79034f1b
+  To update the device tree, we need to know about what periferals and busses are existing on our board. Therefore, we need to reference manual of our board. In the case of the stm32mp157d-dk1, [this](https://www.st.com/resource/en/reference_manual/DM00327659-.pdf) is the datasheet/reference manual
+ All manuals of the stm32mp1 can be found here: https://wiki.st.com/stm32mpu/wiki/STM32MP15_resources
    - The *user manual* is the most important document since it shows us the periferials and connection possabilities
        * Here are i.e. the gpio listings printed down!
    - Within the reference manual (https://www.st.com/resource/en/reference_manual/DM00327659-.pdf), we are able to lookup the physical address of the periferials that we can find on the user manual

## Basic knowlage about device files of busses in linux
+ Within the folder `/sys/bus/<busname>` are the device files for bus devices. These are linked to the hardware device files, which commonly contain the addresses on the SOC in their names (commonly under `/sys/devices/platform/soc`)

## I2C bus on the stm32mp1
+ The stm32mp1 has a i2c controller as an inter-integrated circuit. That means, we have a piece of hardware that is able to send and receive i2c messages. Therefore, the processor needs to know where this i2c controller is to be found. (On which hardware addresses). *This is the responsablility of the device tree!*
    - The concrete i2c controller of the stm32mp1 can handle up to 6 i2c connections, but by default, only 2 of them are activated with the poky and meta-stm32 yocto layer. Both of them are needed to let the board function correctly --> To have a free i2c port on our board, we need to enable it by enhancing the device tree.
    - The device tree enhancement will be done by our custom layer from the previous tutorial
    - ***CAUTION***: _Be aware that we are talking about hardware and therefore unchangeable hardware wireings. I.e. we only can use defined i2c ports of our i2c controller to connect to the gpio pins that you can lookup at page 40 in https://www.st.com/resource/en/user_manual/dm00663230-.pdf_ 

## Device trees
+ To learn more about device tree, see https://bootlin.com/blog/device-tree-101-webinar-slides-and-videos/
+ In yocto, the device tree comes from the board support package. (Only for raspberry pi and beaglebone black, that device tree is included to the poky reference distribution) To find the device tree that is used in your build (to patch it afterwards), go to the `/path/to/yocto_ws/build_kirkstone` directory and search for the name of your board with a `.dts` (device tree source) suffix
    - `$ find . -name stm32mp157d-dk1.dts`
    - We are looking for the device tree that is used by the linux kernel. Therefore, we want to device tree that comes from a folder with `./*/arch/arm/boot/dts/<name>.dts` and NOT the `uboot-source` or `tfa-source` device tree files! (Also the arm trusted firmware and the uboot bootloader needs a device tree, since device trees are a common format that is not proprietary to linux)
+ Within the `stm32mp157d-dk1.dts` file, there are commonly multiple includes of `.dtsi` files (device tree includes, which are device tree configurations that are not describing a full device tree but parts of it.)
    - Have a look at `.dts` and all the `.dtsi` files and search for the periferial that you are looking for (in our case, this is `i2c5`)
    - ***If a periferial is not enabled by default, it will be within the `dts` or `dtsi` files anyway. Therefore we need to search for it and patch the original `dts` file such that the needed periferial is enabled afterwards!*** 

## Patching the device tree
+ The first step is to copy the `stm32mp157d-dk1.dts` from our build folder to a safe place where we can work on it
    - `$ cp /path/to/arch/arm/boot/dts/stm32mp157d-dk1.dts ~/tmp_dts_patch`
+ Then we need to create a second version of it
    - `$ cd ~/tmp_dts_patch && cp stm32mp157d-dk1.dts stm32mp157d-dk1.dts.orig`
+ Add the code to the `stm32mp157d-dk1.dts` such that the i2c5 is enabled.
    - _This is derived from the inspection of the i2c5 definition within the device tree of the build folder where it was disabled!_
+ Then, we need to generate a patchfile. We can use git for this, since git works also with patchfiles under to hood
    - `$ git --no-index stm32mp157d-dk1.dts.orig stm32mp157d-dk1.dts`
    - ***CAUTION***: The order is very important, since if we generate a patch from the `dts` to the `dts.orig`, we would generate an invalid patch which will result in a bitbake error while it trys to apply the patch
    	* ***Tipp***: If you edited your patchfile by hand and you get an error message by bitbake that tells you that your patchfile is malformated, you might want to use a tool like `rediff` (from `$ sudo apt install patchutils`) that reformats your patchfile correctly!
    - ***After the patch file is generated, we need to change the path and name definitions of it to `/arch/arm/boot/dts/stm32mp157d-dk1.dts` for a and b!***
+ Now, we need to add the patch file to our custom layer. Therefore, we need to enhance the device tree generation recipe of our board support package.

### Appending to bitbake recipes
+ To append to a bitbake recipe in another layer (i.e. our custom layer), we need to create the correct path for it. (See https://docs.yoctoproject.org/kernel-dev/common.html#creating-the-append-file how we update a recipe from another layer) Within the layer repository
    - In case of the device tree, it is a part of the kernel recipe. Therefore, we need to create the path in our `meta-custom-jens` repository: `/recipes-kernel/linux/`
        * The `.bbappend` file needs to be in the same folder path that the original recipe (with the `.bb` ending) is in the original layer
        * To know which is the correct path, google for it!
            - In case of the device tree recipe, it can be found within the `recipes-kernel/linux/<name_board>_<kernelversion>.bb`
            - Just go to the board support package layer and then search for `$ find . -name linux-stm32mp1_*`

### Applying a patch file
+ The official instruction how to apply patch files can be found here: https://docs.yoctoproject.org/1.6/kernel-dev/kernel-dev.html#modifying-an-existing-recipe

### Updated overwrite syntax
+ Be careful if you program your `.bbappend` files, since the overwrite syntax has changed from yocto dunfell to kirkstone!
    - See https://docs.yoctoproject.org/migration-guides/migration-3.4.html
    - If you use the wrong overwrite syntax, bitbake will moan at you while building!
+ ***Tipp***: If you want to apply a bitbake append (`.bbappend`) to an arbitary recipe (if multiple are in the same folder), you can use `<name>_%.bbappend` instead of fully nameing the `.bbappend` file according to the recipe that you want to append to!
    * This is especially common for kernel recipes, since the files are named according to the kernel version (e.g. `linux-stm32mp1_5.15.bb` and `linux-stm32mp1_5.13.bb` for different kernel. But we only compile for a specific kernel but want to apply the device tree update to all kernel versions, we can name our file `linux-stm32mp1_%.bb`)

### `.bbappend` syntax
+ At first, we need to type
```
FILESEXTRAPATHS:prepend := "${THISDIR}:"
```
    - This adds the directory where the patch is located to the layer
+ Then, we need to append the patch file to `SRC_URI`
```
SRC_URI:append = " file://0023-add-i2c5-userspace-dts.patch"
```
    - This URI can come from whereever it is available. It can be from an http server or from an ftp server, or from a local file like in this case

### Important notes
+ Refernce from yocto to apply patches to existing yocto recipes: https://docs.yoctoproject.org/1.6/kernel-dev/kernel-dev.html

+ ***CAUTION***: Make sure, you use the new overwrite syntax, if you are using yocto kirkstone or newer, since the `bitbake` overwrite-syntax changed: https://docs.yoctoproject.org/migration-guides/migration-3.4.html

+ If you change the device tree, you ***must*** flash the arm trusted firmware with its metdata and the u-boot partition with it, since they also use the device tree

+ Also, you need to adjust the bootfs `/path/to/mount/bootfs/mmc0_extlinux/extlinux.conf` and the `/path/to/mount/bootfs/mmc0_extlinux/stm32mp157d-dk1_extlinux.conf`, because it defines something with the device tree blob (The compiled device tree binary) 

## Adding i2cdetect to the custom image
+ Since changes to the build configuration that are made with the `$ bitbake -C menuconfig busybox` are only applied to the local build, we want to add the `i2cdetect` application to our custom layer from `README.custom_layer.md`.
    - Since `i2cdetect` is not a standalone package, we need to add `IMAGE_INSTALL:append = " i2c-tools"` to our custom image definition, because `i2cdetect` is a part of the `i2c-tools` package.
    - See https://forums.balena.io/t/building-own-image-apt-get/357599/37?page=2

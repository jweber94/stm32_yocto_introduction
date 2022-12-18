# Intro to Yocto

## What is Yocto?
+ Yocto is a umberella project over open-embedded that develops tools to build your own custom taylored linux distributions for custom and already known chips 
+ FooBar
+ A good intro can be found here:
	- https://docs.yoctoproject.org/2.5/overview-manual/overview-manual.html
	- TBD


## Requirements
+ In order to use bitbake and the open-embedded tools, we need to install the following packages on our build machine:
	- `$ sudo apt install -y build-essential chrpath cpio diffstat gawk git texinfo wget gdisk libssl-dev`
+ Also we need to set an alias to python3 while calling python
	- E.g. by adding `alias python=python3` to our `~/.bashrc`

## Terminology
+ OpenEmbedded: Is a community driven ***project*** that maintains a build system and tools (especially bitbake) that should help to create custom linux distributions without the need to do a linux from scratch
+ Yocto Project: Is an official linux foundation ***project***. It uses the OpenEmbedded tools and the bitbake buildsystem. OpenEmbedded is also a collaborator within the yocto project. The main purpose of yocto is to support the most common architectures for the OpenEmbedded toolset.
+ Metadata: Defined by OpenEmbedded! Metadata are all information about configurations of the build process and where to download the source code
+ Recipe: Is a specific type of metadata that defines instructions that should be executed to do the build
+ Layers: Are logically grouped metadata in a folder, commonly orchestated within repositorys. The nameing convention is that a folder that contains the layer metadata must have a name that starts with _meta-_
+ Board Support Package (BSP): Is a special type of layer that contains metadata information how to compile the linux kernel/firmware and bootloader for a specific piece of hardware. BSPs are usually supported by the vendor of a board. Mostly it contains metadata that tells bitbake where to download specific drivers for your board.
+ Distribution: Is a specific implementation of linux (kernel, rootfs, kernel-modules, ...). By using the OpenEmbedded tools, you are creating your own distribution of linux. A distribution is primarily independend of specific hardware. It is the linux configuration itself! Shortly explained: A distribution is an (linux) operating system made from a specific collection of software components.
+ Machine: Defines the architecture, pins, BSPs, buses, ...
+ Image: Is the output of the build process of your created distribution. It is a concrete realization of the distribution configuration for a specific hardware (since it is compiled for the hardware)

## Bitbake
+ Bitbake is a core component of the yocto project and is used as a build execution engine in open-embedded
	- Bitbake itself is maintained by open-embedded which is totally separated from the yocto project, even if yocto uses bitbake heavyly
+ Bitbake on its own is a generic task execution engine that can execute python and bash scripts! It is optimized to run scripts in parallel and to work with complex inter-task dependencies and constrains
+ Overall, Bitbake is a build engine that works with a specific format of recipies

## OE-Core (OpenEmbedded-Core)
+ OE-Core is a common layer of metadata (recipes, classes and associated files)
+ OE-Core is maintained by OpenEmbedded together with the Yocto project as its maintainer
+ OE-Core is used by all OpenEmbedded derived systems!
	- ***One OE-derived system is Yocto!***
+ OE-Core in combination with BitBake is the ***OpenEmbedded Build System***

## Poky
+ Poky enhances the ***OpenEmbedded Build System*** with some meta-data to create a linux distribution out of the OE-Core layer
+ Poky is the Yocto reference distribution! Poky is not Yocto and Yocto is not Poky!
	- By reference distribution, we mean a collection of known and validly to use meta-data, layers, board support packages, bitbake and various open embedded tools.
	- Poky is a good starting point to develop a custom linux distribution based on your own requirements without starting completly from scratch
	- Based on poky, we can add our own layers for our board to create a custom linux image
+ Poky can be downloaded with git from here: https://www.yoctoproject.org/software-item/poky/

## Yocto releases
+ Like always in software engineering, version compatibility is key when it comes to interaction of various software components! Therefore, yocto has a release cycle with LTS and non-LTS versions that you should use to compile a linux distribution with properly interacting software components!
	- See https://wiki.yoctoproject.org/wiki/Releases for the different releases
	- If you download repositorys, layers, BSPs or meta data from e.g. github, make sure that you are on the branch that you choosen yocto release codename is!
		- In this tutorial, we use the dunfell LTS yocto release to build our linux image!
	- LTS is _long term support_ for commonly 2 years
+ The most vendors that support yocto will have corresponding release codename branches on their repositorys!

## Project workspace recommendation
+ Commonly, we use a workspace folder, e.g. `/path/to/yocto_ws` and clone all required repositorys to this workspace folder
+ After we have choosen a release of the yocto project, we need to checkout on all downloaded repositorys the branch with the choosen codename, like it was explained under the _yocto releases_ heading.


## STM32MP1
+ CAUTION: STM has its own STM32MP1 reference distribution that builds upon the Yocto projects tooling! This is https://github.com/STMicroelectronics/meta-st-openstlinux , which includes packages like QT and other stuff you might not need for your use case. So we do not want to use this as a reference distribution!
	- We use Poky and customize it with the BSP of the STM32MP157D and the required layers for our application!
	- You could use the openstlinux reference Yocto distribution and build upon this, but we want to learn the basics about yocto, so we use Poky!

### Downloading the board support package
+ To download the BSP for the STM32MP157D-DK1, we download the meta-st-stm32mp repository
	- `$ git clone https://github.com/STMicroelectronics/meta-st-stm32mp.git`
	- After downloading, we need to checkout the release codename branch of our choosen yocto release: `$ git checkout dunfell`
	- Within the repository are multiple STM32MP1 BSPs! One of them is the STM32MP157D! It is a collection of meta data and BSP layers for multiple STM32MP1 products of STM
	- *We need to download the board support package from STM's github page, because the STM32MP1 is not supported by the poky reference distribution!*
		* CAUTION: Some other boards like the beaglebone black are supported by poky directly, so we do not need to download an extra BSP for these already supported boards separatly!
+ *Some/Most board support packages need other OE-layers to work correctly! To know which layers you also need to include to your distribution conclumerate, read the README of your boards support package and check if there are any layer dependencies!*
	- In case of the _meta-st-stm32mp_ layer, we have layer dependencies to the _openembedded-core_ layer as well as the _bitbake_ layer. But you can also use the poky reference distribution. Since we already decided to use poky, we do not need to download any depended layers in order to use the meta-st-stm32mp layer in our project!
	- Also, we have a dependency to the _meta-openembedded_ layer, which is not part of the poky reference distribution and therefore, we need to download this layer and checkout the dunfull branch!
		* `$ git clone https://github.com/openembedded/meta-openembedded.git`
		* _meta-openembedded is a suppliment OE-Core layer with additional packages that are not required by the bare minimum OE-Core layer!_
+ ***CAUTION: On every downloaded OE conform layer, we need to check the README to look for depended other layers! We need to manually resolve all layer dependencies within out yocto_ws!***
	- If you forget a depended layer, bitbake will moan at you if you try to build your distribution! ;-)

### Downloading other layers to customize your linux distribution
+ You can search for different functionallity and official supported vendor layers/BSPs on the following OpenEmbedded site: https://layers.openembedded.org/layerindex/branch/master/layers/

### Building your arranged distribution
+ One of the main advantages of yocto is, that it keeps track of already compiled and made configurations. Therefore, if you need to rebuild your image or build another image with some identical packages, you do not have compile or configure everything from scratch!
	- Bitbake parses the project and checks it with the configuration of what was build before
+ Within the poky repository there is a bash script that adds bitbake to your `$PATH` and adds some other yocto project tooling to your environment as well as creating a build folder under the current working directory!
	- Type `$ cd /path/to/yocto_ws && source poky/oe-init-build-env <buildfolder_name>`
+ The `$ source poky/oe-init-build-env <buildfolder_name>` creates a build folder with a folder `/path/to/yocto_ws/<buildfolder_name>/conf` within it. In this `/conf` folder, we have a `bblayers.conf` file, where all poky (!) layers are included. To include all other layers of your own yocto project, we need to add the paths to the downloaded other layers/BSPs/meta-data within the `yocto_ws`!
	- Example:
```
# POKY_BBLAYERS_CONF_VERSION is increased each time build/conf/bblayers.conf
# changes incompatibly
POKY_BBLAYERS_CONF_VERSION = "2"

BBPATH = "${TOPDIR}"
BBFILES ?= ""

BBLAYERS ?= " \
  /home/jens/Desktop/intro_to_embedded_linux/yocto_ws/poky/meta \
  /home/jens/Desktop/intro_to_embedded_linux/yocto_ws/poky/meta-poky \
  /home/jens/Desktop/intro_to_embedded_linux/yocto_ws/poky/meta-yocto-bsp \
  /home/jens/Desktop/intro_to_embedded_linux/yocto_ws/meta-st-stm32mp \
  /home/jens/Desktop/intro_to_embedded_linux/yocto_ws/meta-openembedded/meta-oe \
  /home/jens/Desktop/intro_to_embedded_linux/yocto_ws/meta-openembedded/meta-python \
  "
```
	- To check if all layers are captured correctly by bitbake, type `$ bitbake-layers show-layers`
		* The complete project will be parsed and all layers will therefore shown up within the terminal
	- The main reason, why this script tells us, which layers we want in our project and where these layers are located, is the setting of the `BBLAYERS` environment variable. This tells us the used layers in our workspace/project
	- If we have something like `meta-openembedded` which is a collection of multiple layers, we have to add the `meta-*` layer within this repository.
		- A hint where to find such a layer in a layer that has a dependency that we need to include, can be found within the `/path/to/yocto_ws/<layer>/conf/machine/<boardname>.conf` under the variable name `#@NEEDED_BSPLAYERS`
+ Besides the additional layers that should be compiled on top of poky, we need to choose the machine for which we want to compile our distribution. Therefore, we have another file, called `local.conf` wthin the `/path/to/yocto_ws/<buildfolder_name>/conf` folder. Here we can set different options on the compilation process. One of the options is `MACHINE`, where we can set the targeted machine `stm32mp1`
	- By default, the target machine is qemu (Quick Emulator - a system emulator under linux)
	- We can lookup a valid machine name by the BSP-layer that you might have downloaded (in our example the `meta-st-stm32mp` layer/repository) under the file `/path/to/yocto_ws/meta-st-stm32mp/conf/<boardname>.conf` under the variable `#@NAME` (directly under the variable `#@TYPE: Machine`)
+ Now we are ready to compile our distribution and/or to use `menuconfig` to make some configuration like activating the real time patch
	- Opening menuconfig: `$ cd /path/to/yocto_ws/<buildfolder_name> && bitbake -c menuconfig virtual/kernel`
	- Compiling the image: `$ bitbake <imagename>`, e.g.: `$ bitbake core-image-minimal`
+ ***CAUTION***: If you open a new terminal, we need to add bitbake and the other yocto tooling to our `$PATH` variable. Therefore, we need to type again `$ cd /path/to/yocto_ws && source poky/oe-init-build-env <buildfolder_name>`.
	- If we already changed the configurations under `/path/to/yocto_ws/<buildfolder_name>/conf`, the custom configuration will *not* be overwritten. Only the yocto tooling is sources within the terminal

### Yocto images to build against
+ Images are a core concept in yocto. The aim of a yocto project is to create an image that is custom tailored to your board and application. Poky brings a lot of reference images with it. These are `.bb` files that can be found under an `/images` folder of the individual layer.
	- The reference images from poky are meant to use as a reference that you can build upon to create your own image that is tailored to your application. You ***should not*** edit the already existing images!
	- The most common starting point is the `core-image-base.bb` image, which can be found within `/path/to/yocto_ws/poky/meta/recipes-core/images/core-image-base.bb`
+ A good reference how to create your own layer is: https://hub.mender.io/t/how-to-create-custom-images-using-yocto-project/902

### Cleaning up the build folder:
+ If you want to recompile everything, just run `$ rm -rf /tmp` within the build folder
	- Within the `/path/to/yocto_ws/<buildfolder>` folder, all binarys/compiled code is placed into the `/tmp` folder!
+ If you want to just delete the images but you do not want to compile everything again, run `$ bitbake -c cleanall <imagename>`

## Inspecting Yocto distribution repositorys
+ A good overview of the distribution gives the _distribution config_ file. This can be found under `/path/to/yocto_ws/poky/meta-poky/conf/distro/poky.conf`. The following parts are interesting to look up for a first glance at the distribution:
	- `SANITY_TESTED_DISTORS`: Already existing linux distributions that the build with poky (or your distro) was successfully tested on
		* If you do not build your self compiled yocto distribution, you will probably get an error, but this does not mean it will not build. It is only a check that you build on a distro that was not already tested by the yocto maintainers!
	- `DISTRO`: Name of the distribution that should be build
+ Yocto has an overwrite policy, so if a higher abstracted layer (indicated by a number that every layer needs) overwrites the configuration of a lower layer, it will just overwrite it!
	- Another possability is that you can use `.bbappend` files to append something to another confuguration


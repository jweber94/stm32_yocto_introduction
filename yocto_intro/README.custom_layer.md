# Custom Layers in Yocto
+ Remember: A layer in yocto is just a collection of interrelated meta data
+ One of the most important features of yocto while developing your own linux distribution is the ability of creating your own custom layers, such that you can add custom functionallity to your distribution (e.g. by adding specific software packages or drivers or your own/self-written applications)

## Layer repository structure
+ Commonly (not necessary), a layer needs to have the prefix `meta-` (for meta data) as its foldername

### `.bbclass`-files
+ Basically, `.bbclass`-files contain a interrelated functionallity/set of variables for a specific function.
    - The name of the `<name>.bbclass`-file from a layer(-repository) tells us which basic idea stands behind a specific class
    - `.bbclass`-files can be used to inherit from them within
        * other `.bbclass`-files
        * `.bb`-files (which define images)
    - ***Not*** all `.bbclass` files of a layer are included by other bbclasses or image definitions! They can stand alone and exist to inherit from them within your custom layers!
        * Building on that: for an linux image, not all defined classes of a layer need to be used
+ Within the meta folder (which indicates metadata), we have two main folders:
    - `classes`: Contains all classes of the layer
    - `recipes-*`: Contains recipes which correspond to the name (indicated by the * in the name of the folder)
+ `.bbclass`-Files:
    - A `bbclass` is like an (abstract) base class for other derived classes/objects. It basically contains a templated recipe (reminder: A recipe is a special type of metadata that defines what should be executed during the build process)
        * If we import bbclasses to our custom layer recipe, we can enhance their definition or overwrite them! Therefore you can think about them as _abstract_ base classes
        * Basically, a `.bbclass` file is a python-like script (with some parts written in python and some in a bash-like fashion) that defines some variables and executes functions to do the build process
            - The defines variables are used by bitbake to create the image and higher oder layers can overwrite or append to defined variables!
        * A central keyword within `.bbclass` files is the `inherit` keyword:
            - This makes are variables and scripts available within that bbclass-file!
    - Within `.bb`-Files, we inherit from `.bbclass` files/classes in order to call them with bitbake and create a linux image from them!
        * e.g. `/path/to/yocto_ws/poky/meta/recipes-core/images/core-image-minimal.bb` has the line `inherit core-image` within it, which inherits/includes everything from `/path/to/yocto_ws/poky/meta/classes/core-image.bbclass`
        * `bitbake` knows every file and class, because it parses all files that are within the defines folders (and subfolders) that you have configured within the `/path/to/yocto_ws/build_kirkstone/conf/bblayers.conf` file
+ Important to note here is that we need to set some packagegroups for the `CORE_IMAGE_BASE_INSTALL` to get a booting linux!
    - This is not a part of `/path/to/yocto_ws/poky/meta/classes/image.bbclass`, which is the basic poky image class from which all other classes (and images) are derived. --> `image.bbclass` generates us ***NOT*** a usable linux distribution without further refinement!
    - See the last lines of `/path/to/yocto_ws/poky/meta/classes/core-image.bbclass` to see the minimal necessary linux packages that you need to create a running linux
+ Another very important variable is the `CORE_IMAGE_EXTRA_INSTALL`, which commonly comes from the board support packages, that defines hardware specific packages, that run enables linux to run on our particular board!
+ ***Commonly, the `CORE_IMAGE_BASE_INSTALL` and the `CORE_IMAGE_EXTRA_INSTALL` are concatinated to the `IMAGE_INSTALL` variables within the `.bb`-file, which defines ALL packages, that our concrete image should contain after the build***
+ Commonly, we place the image definitions within our repository within the folder `/path/to/yocto_ws/meta-custom-layer/recipes-core/images`
    - Here are the `.bb`-files for the images are stored
### `.bb`-Files
+ Here we can finally overwrite or append to variables that were defined before
+ We also can define variables that we want to set only for our image (i.e. `IMAGE_ROOTFS_SIZE` which defines the size of the rootfs)
+ This is also the place where we can add users, change passwords or make some other configurations for our final linux 

## How Layers are created
+ We do ***NOT*** want to modify classes/recipes of poky! We can use them as base classes or as a base implementation for our own recipes/classes
+ Create a custom layer:
    * First, you need to `$ source /path/to/yocto_ws/poky/oe-init-build-env build_kirkstone`, to have bitbake available within your `$PATH`
    * Then, `$ cd /path/to/yocto_ws && bitbake-layers create-layer meta-custom-jens`, to create a layer (aka folder) under the path `/path/to/yocto_ws/meta-custom-jens`
        - The last command creates a basic yocto layer structure with some example recipes in it
    * This layer needs to be added to `/path/to/yocto_ws/build_kirkstone/conf/bblayers.conf` --> now, bitbake knows about the new layer
+ Creating an image within our custom layer:
    * Create the folder `$ mkdir -p /path/to/yocto_ws/meta-custom-jens/recipes-core/images/` with the file `$ touch /path/to/yocto_ws/meta-custom-jens/recipes-core/images/custom-jens.bb`
    * Copy the basic content from `/path/to/yocto_ws/poky/meta/recipes-core/images/core-image-minimal.bb` to have a start for the custom layer
+ Its best practice to have your custom layer folder under version control (e.g. git). Commonly, we name the branch after the yocto release that it is tested against/used with

## Tipps by creating custom layers and images
+ By defining and append/overwrite variables within your layer, you tweak/modify your image and/or distribution

### Images
+ It is good practice to make the rootfs only as big as it needs to be. This enhances the security, since a hacker is not able to install mallicious software on it that needs much filesystem space
+ To add useres to your linux image, you can inherit from the `/path/to/yocto_ws/poky/meta/classes/extrausers.bbclass`
    - Here, the `useradd` command for the `.bb`-file is defined (here, the corresponding yocto command is defined)  
### Working with bitbake
+ Since bitbake parses at the beginning all recipes and metadata, it knows all variables in advance. You can look at variables with the command `$ bitbake -e | grep <variablename>`

## Example
+ An example for a custom layer for the STM32MP157D-DK1 can be found here: https://github.com/jweber94/meta-custom-jens
    - This is based on the tutorial but for the yocto kirkstone release.
    - To add a password to the root user, it is no longer possible to safe it in plain text within the recipe. Therefore, you need an escaped version of a sha256 hash of your demanded password and safe it with the lower case `-p` command (instead of the captial letter `-P` command of the tutorial video)
+ ***CAUTION***: You can set priorities to layers, which determents which layers/scripts get called first and therefore which configuration gets more priority over others, if they are set in different layers


## Homework
+ Add a user with its own userspace filesystem that is writeable for the user to our custom yocto image
    - Since the userfs is the only fs that the user can change, it is a security feature to create a userfs
+ Build for stm32mp-disco
# Adding a custom application to your yocto image
+ Reference tutorial: 
    * https://www.youtube.com/watch?v=DVlQZnFjO9o&list=PLEBQazB0HUyTpoJoZecRK6PpDG31Y7RPB&index=7
    * https://www.digikey.com/en/maker/projects/intro-to-embedded-linux-part-6-add-custom-application-to-yocto-build/509191cec6c2418d88fd374f93ea5dda

## Yocto SDK
+ To develop software for yocto images, yocto has a software development kit (SDK), which has some tooling (which is part of the yocto project) to develop and test applications. If you create more complex applications, use these tools for development, since it makes it much easier instead of doing everything by hand!
    - Helps to debug and integrate your application to a custom image
    - Good for application and kernel development
    - To learn the basics and what these development tool are doing behind the screnes, we do in this tutorial everything manually
    - See https://docs.yoctoproject.org/2.1/sdk-manual/sdk-manual.html as a starting point and reference for the yocto SDK
+ Besides the SDK approach, you can setup an development image for your board and develop on the device itself to make sure that everything works

## Integration of the application to the yocto image
+ ***We assume that you have developed a custom application (e.g. in C) that we just need to integrate to our custom image!***
+ Commonly, applications are stored in `/path/to/yocto_ws/meta-custom-layer/recipes-apps/<appname>` with the following folder structure
    - `<appname>`
        * `<appname>_0.0.bb`: Bitbake recipe which defines how to build and install your custom application
        * `./files/src`: Folder where to store your custom application code
            - `<applicationname>.c`: The application itself. This could be a complete project with a build system like cmake
            - ***Instead of storeing the custom application code within the layer definition itself, we can use a link to a remote (e.g. github) repository within the application recipe, which is downloaded while the system is gona build*** 
                * _This is commonly done, if your application is used on multiple platforms/boards/processor-types!_

### Build Step
+ To define how the application should be build, we define our `<appname>_0.0.bb`, where `_0.0` is the version number of our application!
    - ***CAUTION***: The name of this recipe (which is the case for *ALL* bitbake recipes) must not contain underscores! You can use - instead! (Since underscores are only used to define the version number by bitbake!)
        * Instead of `my_recipe_0.0.bb`, use `my-recipe_0.0.bb`! If you do not do this, bitbake will moan at you while it is trying to execute your image recipe! (With an error like `To many underscores`)
    - Example for a recipe:
```
SUMMARY = "Get temperature recipe"
DESCRIPTION = "Custom recipe to build get_i2c_data.c application"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"

# Where to find source files (can be local, GitHub, etc.)
SRC_URI:append = " file://src"

# Where to keep downloaded source files (in tmp/work/...)
S = "${WORKDIR}/src"

# Pass arguments to linker
TARGET_CC_ARCH += "${LDFLAGS}"

# Cross-compile source code
do_compile() {
    ${CC} -o get_i2c_data get_i2c_data.c
}

# Create /usr/bin in rootfs and copy program to it
do_install() {
    install -d ${D}${bindir}
    install -m 0755 get_i2c_data ${D}${bindir}
}
```
- ***CAUTION***: Bitbake/Yocto always needs to have a licence which checks it with a md5sum hash!
    * To check for default licences, see `/path/to/yocto_ws/poky/meta/files/commmon-licences/<your-license>`! These can be used as the `LICENSE` entry in your recipe if you are using poky!
    * You can add your own licence or make it closed source!
    * `LIC_FILES_CHKSUM` is used to verify that your licence is corretly set (and not changed after you decided which licence you wanted to use when you developed your application!)
- `SRC_URI:append`:
    * Explaination, what SRC_URI does: http://www.embeddedlinux.org.cn/OEManual/src_uri_variable.html
    * If you use a local source file or folder, you need to have it placed within a `./files` folder, relative to the path to the recipe where it is included (the path is then set by bitbake!)
- `${S}`: Defines the location where unpacked recipe source code should be stored!
    * This needs to be set for every recipe that uses a SRC_URI
    * See https://docs.yoctoproject.org/3.2.3/ref-manual/ref-variables.html#term-S for details
        - Here is also an example how to add a git repo as a source!
- `TARGET_CC_ARCH`: Flags for the linker - You need to assign something, even if its an empty variable! If you do not do this, bitbake will fail
- `do_compile()`: Defines steps/a script that is executed when bitbake calls the do_compile steps in order to build your application
    * You can use the bitbake variables (like ${CC} in the example, which defines the cross compiler)
- `do_install()`: Defines the install steps while bitbake executes the install step for you application (it is like the `do_compile` function)
    * In our case, we use the [install](https://man7.org/linux/man-pages/man1/install.1.html) function of linux

### Integration Step
+ After everything was configured and build correctly, we just can include our custom application by defining it as an `IMAGE_INSTALL += " <appname>"` (without the version- and `.bb` suffix)
    - Then call `$ cd /path/to/yocto_ws && source poky/oe-init-build-env build_kirkstone && bitbake custom-jens`
    - After everything builds correctly, we can flash our device/SD-Card and your application runs on the board if everything worked correctly

## TODOS
+ ## Development of the i2c read out application for the GY-68
+ Read about SRC_URI
+ Read about the `${S}`, `${PN}`, ... variables
+ Read about the SDK
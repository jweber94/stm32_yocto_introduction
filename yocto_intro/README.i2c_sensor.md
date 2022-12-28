# Using the i2c5 port to connect to the Bosch GY-68 temperature-pressure sensor
+ We have enabled our i2c5 port by patching the device tree in `README.device_tree_patch.md`.
+ Pin-Configuration on the board (https://www.st.com/resource/en/user_manual/dm00591354-discovery-kits-with-stm32mp157-mpus-stmicroelectronics.pdf page 31):
    - i2c5_SDA: pin 3
    - i2c5_SDL: pin 5
    - 3.3V: pin 1
    - GND: pin 9
+ After patching the device tree, we can lookup the newly added i2c device/bus on our linux image by
    - `ls -l /sys/bus/i2c/devices/`
        * Here, we need to look out for the i2c5 address which you can look up in the stm32mp1 reference manual (https://www.st.com/resource/en/reference_manual/DM00327659-.pdf) at page 165. In that way, we can see which `/dev/i2c-x` device file is linked to the newly enabled i2c port
        * In case of i2c5, we are looking for the HEX-address `0x40015000` which is mapped to `/dev/i2c-1`
+ Detecting the i2c devices:
    - `$ i2cdetect -y 1`, where the `1` is for `/dev/i2c-1` in that case
    - In my case, I received the following output:
```
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:                         -- -- -- -- -- -- -- -- 
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
70: -- -- -- -- -- -- -- 77 
```

## Application development
+ Makeing it the normal way: Use the yocto SDK: https://docs.yoctoproject.org/2.1/sdk-manual/sdk-manual.html
    - Good for application AND kernel development - ***Highly recommended to use!***
+ Doing it the hard way: Develop on the target machine (you need to use an image with development packages installed) or develop on a similar device with development tools installed (e.g. a raspberry pi). This is want we want to do in this tutorial.

### Datasheet for the i2c interaction
+ Our sensor is the Bosch GY-68 BMP180. Its datasheet can be found here: https://cdn-shop.adafruit.com/datasheets/BST-BMP180-DS000-09.pdf

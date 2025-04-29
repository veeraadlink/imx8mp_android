# Android 14 for ADLINK LEC-iMX8MP

## Contents
```
1. Hardware Details
2. Software Details
3. Package structure
4. Flashing the image and booting
   4.1. SD boot
   4.2. eMMC boot
5. Peripheral testing
   5.1. USB type A ports
   5.2. Micro USB (Device mode)
   5.3. HDMI
   5.4. UART - Console
   5.5. UART - RS232
   5.6. WM8960 Audio codec
   5.7. GPIO on expansion connector
   5.8. PWM GPIO
   5.9. SPI on expansion connector
   5.10. CAN interface
   5.11. RTC
   5.12. Wifi/BT
   5.13. MIPI DSI display
   5.14. LVDS display
   5.15. MIPI CSI camera
   5.16. PCIe
   5.17. ETHERNET
         5.17.1. Ethernet in U-boot
         5.17.2. Ethernet in Android
6. OTA Update
   6.1. Host Preparation
         6.1.1. Install Apache Server
   6.2. Build OTA Package
   6.3. OTA Update
```
## 1 Hardware Details

| Base | I-Pi SMARC Plus |
|:----------------|:-----------|
| **Module** | **LEC-iMX8MP** |

## 2 Software Details
|   Android   |   Ver  14   |
|:-----------:|:-----------:|
| **Kernel**  | **6.6.36** |
| **U-Boot**  | **2024.04** |
| **Host OS** | **Ubuntu 22.04.4** |


## 3 Package structure

 ```
  |---adlink-lec-imx8mp-android-upsidedowncake_V2_R1_241118
     |--- android images
     |--- README.md
 ```
- Download Android release (adlink-lec-imx8mp-android-upsidedowncake_V2_R1_241118.zip) and extract it.



## 4 Flashing the Image and Booting

### 4.1 SD Boot

#### Host preparation

1. On a linux host machine, insert the micro SD card (through an USB adapter).

2. Check the device node of the micro SD card using dmesg command.

3. Move into android release directory ```adlink-lec-imx8mp-android-upsidedowncake_V2_R1_241118```

   ```
   $ sudo cp tools/lib64/libc++.so /lib/x86_64-linux-gnu
   $ sudo chmod +x /lib/x86_64-linux-gnu/libc++.so
   ```

   ```
   $ sudo cp tools/bin/make_f2fs tools/bin/simg2img /usr/bin
   $ sudo chmod +x /usr/bin/make_f2fs /usr/bin/simg2img
   ```

#### Flash image to SD card

* Execute the following command for a 32 GB Micro-SD card.
   ```
   $ sudo ./imx-sdcard-partition.sh -f imx8mp -c 28 -s <N>gb -u dual /dev/sdX
   ```
* Replace N with size of RAM installed in the module

* /dev/sdX need to be changed to actual device node of the micro SD card

* For more details, please refer: https://www.nxp.com/docs/en/user-guide/ANDROID_USERS_GUIDE.pd

  Note: First boot from SD card can be slow,subsequent boot will be faster


### 4.2 eMMC Boot

#### Download uuu utility

 - Download uuu utility and copy to /usr/bin

 - https://github.com/nxp-imx/mfgtools/releases/download/uuu_1.4.182/uuu

   ```
   $ sudo cp ~/Downloads/uuu /usr/bin
   $ sudo chmod +x /usr/bin/uuu
   ```

#### Boot into Recovery Mode

 * Set the boot switch into recovery mode.
 * Connect USB OTG cable to host.
 * Power on the board.

#### Flash image to eMMC

* Execute the following command to start flashing Android image to eMMC.

   ```
   $ sudo ./uuu_imx_android_flash.sh -f imx8mp -e -c 28 -s <N>gb -u dual
   ```

* Replace N with size of RAM installed in the module.
* Once flashing completed, power off the board and change boot settings to eMMC mode.
* Power on the board to boot Android from eMMC.
* A detailed guide on using UUU tool is available in: https://www.ipi.wiki/pages/imx8mplus-docs?page=HowToFlashImageeMMCUsingUUUTool.html


## 5 Peripheral testing
### 5.1 USB Type A

* All USB type A ports are validated.
* Any storage device connected on these ports will be mounted at "/mnt/media_rw" location.
* Device can also be accessed from Android GUI.


### 5.2 Micro USB (Device mode)

 * Connect micro USB cable to access device in client mode.
 * Devices can be accessed using adb commands.

 #### Running adb
 * After Board boots up, connect the micro USB port on the ADLINK board with the Host system using a Micro USB cable.
 * On a Ubuntu machine, install adb using below commands.
   ```
   $ sudo adb usb
   $ sudo adb shell ls <to list the current directory on the Android board>
    ```

### 5.3 HDMI

HDMI function is enabled by default.

### 5.4 UART1 - Console

* Connect UART cable to CN1001 expansion connector to get android boot logs.
* Console UART works at TTL level. Use TTL compatible USB Serial adapter to get logs.

Pin connection:

| Pin  | Function |
|:----:|:--------:|
| 10 | UART RX |
| 8  | UART TX |
| 6  | GND     |


### 5.5 UART2 - RS232

 Connect RS232 compatible UART cable to CN1609 expansion connector.

 Pin connection:

| Pin  | Function |
|:----:|:--------:|
| 1 | UART RX |
| 3 | UART TX |
| 5 | GND     |

#### UART Tx Test
* Open minicom, 115200 baudrate with no hardware flow control setting.
* Run the below commands from adb shell to transmit data to Minicom.
   ```
   $ stty -F /dev/ttymxc2 115200 cs8 -cstopb -parenb
   $ echo 'ADLINK' > /dev/ttymxc2
   ```
   'ADLINK' string will be displayed in minicom

#### UART Rx Test
* Run below command in adb shell
   ```
   $ cat /dev/ttymxc2
   ```
   Type some data and press enter in Minicom.
   The data will be received in adb shell.

### 5.6 WM8960 Audio Codec

------

Connect headphone on the jack, execute the following command to play on headphone:

```
$ tinyplay <wav file> -D 1
```

To play the wav file on HDMI, execute the command below:
```
$ tinyplay <wav file> -D 0
```
***NOTE:***
* HDMI audio device has higher priority over other audio devices.
When HDMI display is connected, android will use HDMI as playback device.

* To make WM8960 as system playback device, disconnect HDMI and use LVDS/MIPI panel as primary display device.
Steps for flashing android image with LVDS/MIPI enabled are mentioned in section 5.13 and 5.14

* WM8960 can always be tested irrespective of connected display from adb shell using tinyplay command mentioned above.

### 5.7 GPIO on Expansion Connector

 GPIO on expansion connector (CN1001) can be accessed using following commands:

```
$ cd /sys/class/gpio/
$ echo GPIO_NUM > export
$ cd gpio<GPIO_NUM>
$ echo out > direction ("out" is to enable pin as ouput, "in" for input)
$ echo 1 > value       ("1" to drive high, "0" to drive low)
```

 The GPIO_NUM mentioned above are mapped to following pin numbers:

| Pin on expansion | Kernel Gpio number |
|:----------------:|:------------------:|
|      7        |      514    |
|      12       |      515    |
|      11       |      516    |
|      13       |      517    |
|      15       |      518    |
|      16       |      519    |
|      18       |      520    |
|      22       |      521    |
|      29       |      546    |
|      31       |      547    |
|      32       |      548    |
|      33       |      549    |
|      35       |      550    |
|      36       |      551    |
|      37       |      552    |
|      38       |      553    |
|      40       |      554    |

### 5.8 PWM GPIO's

* Few gpio on expansion connector has additional PWM feature via SX1509 I/O expander.
* Following pins support pwm:

    29, 31, 32, 33, 35, 36, 37, 38, 40

* ```adlink_pwm``` utility can be used to generate pwm signal on above gpio.
* Use below command to turn on pwm:

   ```shell
   $ adlink_pwm <pin> <state> <duty>
   ```

#### Example:

* To enable 30%  duty cycle on pin 29:

   ```shell
   $ adlink_pwm 29 ON 30
   ```

* To turn off pwm on pin 29:

   ```shell
   $ adlink_pwm 29 OFF
   ```

### 5.9 SPI on expansion connector
 Two instances of SPI are available for user.

 Follow below procedure to perform loop-back test:

#### SPI1 Loopback test
* Connect Pin 19 and 21 in CN1001 connector
* Run below command to send and receive data over SPI1
    ```
    $ spidevtest -D /dev/spidev1.0 -p "12345" -N -v
    ```

 #### SPI2 Loopback test
* Connect Pin 27 and 28 in CN1602 connector
* Run below command to send and receive data over SPI2
    ```
    $ spidevtest -D /dev/spidev2.0 -p "12345" -N -v
    ```

### 5.10 CAN interface (CN1602)

Setup CAN0 & CAN1 Loopback: Connect Pins (13 - 14) and (15 - 16) in CN1602 connector.

Sender should execute below commands:

1. Configure the CAN0 ports as
   ```
   $ ip link set can0 type can bitrate 500000
   $ ip link set can0 up
   ```

2. Configure the CAN1 ports as
   ```
   $ ip link set can1 type can bitrate 500000
   $ ip link set can1 up
   ```

3. Dump CAN data on can0:
   ```
   $ candump can0 &
   ```

4. Send data over can1:
   ```
   $ cansend can1 01a#11223344AABBCCDD
   ```

    Now, data sent from CAN1 will be dumped back on CAN0.


### 5.11 RTC

* Android will update date/time from internet when connected to network. Date/time settings in android available under:
  Settings -> System -> Date & time

* Once time is updated, remove network cable and power off the board.
  Wait for some time and power on the board, system will now sync time from RTC.

### 5.12 Wifi/BT

WiFi/BT supported in Android and functionalities can be realised by using Android Settings.

### 5.13. MIPI DSI display

* DSI display feature can be enabled by flashing corresponding DTB file during SD/eMMC boot media preparation.
* Append "-d mipi-panel' to SD/eMMC flash command to enable DSI feature.
* Android will support HDMI + DSI dual display feature with this flash command.

Example:

* Run ```$ sudo ./imx-sdcard-partition.sh -f imx8mp -c 28 -s 2gb -u dual -d mipi-panel /dev/sdX```
to prepare SD card with DSI feature enabled for 2G module

* Run ```$ sudo ./uuu_imx_android_flash.sh -f imx8mp -e -c 28 -s 4gb -u dual -d mipi-panel```
to flash image to eMMC with DSI feature enabled for 4G module

### 5.14. LVDS display

* LVDS feature can be enabled by adding '-d lvds-panel' to flash command.
* Android will support HDMI + LVDS dual display feature with this flash.

Example:

* Run ```$ sudo ./imx-sdcard-partition.sh -f imx8mp -c 28 -s 4gb -u dual -d lvds-panel /dev/sdX```
to prepare SD card with LVDS feature enabled for 4G module

* Run ```$ sudo ./uuu_imx_android_flash.sh -f imx8mp -e -c 28 -s 2gb -u dual -d lvds-panel```
to flash image to eMMC with LVDS feature enabled for 2G module


### 5.15. MIPI CSI camera
* OV5640 camera is enabled by default.
* Connect camera to 2-lane MIPI CSI connector.
* Camera App can be used to stream video from camera.

### 5.16. PCIe
* Run ```lspci``` command from adb shell to list connected PCI devices.
* Vendor ID details can be extracted by running lspci command with '-n' argument.

### 5.17 Ethernet

#### 5.17.1 Ethernet in u-boot
 * Press any key to break into U-Boot command prompt.
 * Execute the below commands to configure u-boot network (The following are provided as an example, please change appropriately)
   ```
   u-boot=> setenv ipaddr 192.168.1.126
   u-boot=> setenv serverip 192.168.1.5
   u-boot=> setenv netmask 255.255.255.0
   ```
##### ETH0 - FEC
* Execute the below commands to ping from ETH0 port.

      u-boot=> setenv ethact eth0
      u-boot=> ping 192.168.1.1

##### ETH1 - DWMAC
* Execute the below commands to ping from ETH1 port.

      u-boot=> setenv ethact eth1
      u-boot=> ping 192.168.1.1

## 6 OTA Update

### 6.1 Host Preparation

#### 6.1.1 Install Apache Server

* Apache server can be installed as below
  ```
  $ sudo apt install apache2 ufw
  $ sudo ufw allow 'Apache'
  $ sudo mkdir /var/www/html/downloads
  $ sudo vi /etc/apache2/sites-available/000-default.conf
  ```
* Edit apache configuration file
  ```
  $ sudo vi /etc/apache2/sites-available/000-default.conf
  ```
  add below content inside ```<VirtualHost>``` block

  ```
	//-----------------000-default.conf------------------
	<Directory /var/www/html/downloads>
	    Options +Indexes
	    AllowOverride None
	    Require all granted
	</Directory>
	//---------------------------------------------------
* Execute ```sudo systemctl restart apache2``` command to restart Apache server.
* Run ```hostname -I``` command to get IP address of server.


### 6.2 Build OTA Package

1. Modify "BOARD_OTA_BOOTLOADERIMAGE" variable in "device/nxp/imx8m/lec_imx8mp/SharedBoardConfig.mk" according to size of RAM installed in the module:

	```BOARD_OTA_BOOTLOADERIMAGE := bootloader-imx8mp-<N>gb-dual.img```

	Replace N with actual RAM size: ```2``` ```4``` or ```8```
2. Execute below commands to build OTA package
   ```
   $ export OTA_SERVER="http://<serverip>/downloads/lec_imx8mp-ota.zip"
   $ ./imx-make.sh -j20 otapackage
   ```
3. Modify ```serverip``` in "OTA_SERVER" variable according to apache server installation
4. Copy generated OTA file to Apache downloads directory
   ```
   $ sudo cp out/target/product/lec_imx8mp/lec_imx8mp-ota.zip /var/www/html/downloads
   ```
5. Open ```http://<serverip>/downloads``` in a browser and check if ```lec_imx8mp-ota.zip``` is successfully shown.

***NOTE:***
* OTA update can also be realised using prebuild OTA binary available in release package.
* To use prebuilt OTA, skip steps 1-3. Copy ```lec_imx8mp-<N>gb-ota.zip``` and ```lec_imx8mp-<N>gb-ota.json``` from release package and rename them to ```lec_imx8mp-ota.zip``` and ```lec_imx8mp-ota.json```
* Update "url" section in lec-imx8mp-ota.json file with actual server address
   ```
   "url": "http://<serverip>/downloads/lec_imx8mp-ota.zip"
   ```
* Continue from step 4 and use updated prebuilt OTA binary instead of ```out/target/product/lec_imx8mp/**OTA-FILES**```

### 6.3 OTA Update

* Execute below command in HOST to copy OTA json file
   ```
   $ adb push out/target/product/lec_imx8mp/lec_imx8mp-ota.json /data/local/tmp
   ```
* Prepare TARGET for OTA update by running below commands in ```adb shell```
   ```
   $ mkdir -m 777 -p /data/user/0/com.example.android.systemupdatersample/files
   $ mkdir -m 777 -p /data/user/0/com.example.android.systemupdatersample/files/configs
   $ cp /data/local/tmp/*.json /data/user/0/com.example.android.systemupdatersample/files/configs
   $ chmod 777 /data/user/0/com.example.android.systemupdatersample/files/configs/*.json
   ```
* Launch ```SystemUpdaterSample``` app in TARGET.

* Tap ```VIEW CONFIG``` and check details of the update
* Tap ```APPLY``` to start OTA update process
* Execute ```logcat``` command in ```adb shell``` to monitor OTA progress
* Wait for OTA update to complete. Update usually takes 15-20 mins time.
* Check ```Engine Error``` status after update is complete and it should show 'Success'
* Reboot the board. Android will use installed OTA update in next boot.

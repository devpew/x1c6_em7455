

# Lenovo X1 Carbon 6th + EM7455

## INTRO

I have Lenovo X1 Carbon 6th (x1c6) but in my notebook configuration it was without LTE modem. And couple months it doesn't supported any LTE modems, but about 3 months ago it started to support two modems Mobile broadband Fibocom L850-GL and Mobile broadband Sierra EM7455.

I recently bought Mobile broadband Sierra EM7455 on aliexpress and tried to install it. 

I bought just a modem, but it came with standard antennas.

But it was with a bunch of problems.

I always got an error that i have no SIM card in the tray. 

So, I thought that I have a problems with SIM tray, espessially when i read this post

https://forums.lenovo.com/t5/ThinkPad-X-Series-Laptops/X1-Carbon-6th-Gen-WWAN-SIM-not-detected/m-p/4202613/highlight/true#M90211

And I spent couple hours to attach some peace of plastic to my SIM tray

But i still got an error that SIM card is not inserted

Then I decided that I have software problems. 

And that's how you can fix it if you have the same problems:

# FIX


First of all we need to check if modem is connected

```
lsusb
```

If everything is ok we should stop modemmanager


```
sudo systemctl stop ModemManager
```

After that we should download and unzip latest firmware from https://source.sierrawireless.com/resources/airprime/minicard/74xx/airprime-em_mc74xx-approved-fw-packages/

Also you could download another things there - https://source.sierrawireless.com/devices/em-series/em7455/

And update the firmware

```
cd /path/to/extracted/firmware
```

```
sudo qmi-firmware-update -d 1199 -u *.cwe *.nvu
```

>  If you have an error on this stage (I had it too) you should see how to fix it at the bottom of this page.

Change the modem's USB composition to enable AT command ports:

```
sudo qmicli -d /dev/cdc-wdm1 --dms-swi-set-usb-composition=8
```

Power-cycle the modem as advised by qmicli: 

```
sudo qmicli -d /dev/cdc-wdm1 --dms-swi-get-usb-composition
```

Verify that the three serial ports `/dev/ttyUSB0`, `/dev/ttyUSB1` and `/dev/ttyUSB2` are now available (assuming you do not have any other USB-serial converters plugged in):

```
# ls -l /dev/ttyUSB*
crw-rw---- 1 root uucp 188, 0 Feb 14 20:11 /dev/ttyUSB0
crw-rw---- 1 root uucp 188, 1 Feb 14 20:11 /dev/ttyUSB1
crw-rw---- 1 root uucp 188, 2 Feb 14 20:11 /dev/ttyUSB2
```

After that we should connect to our modem via screen or picocom or tia 

```
sudo screen /dev/ttyUSB2 115200
```

```
sudo picocom /dev/ttyUSB2 115200
```

And after that we should use AT commands

Enable command echo (if echo is initially disabled, you won't see this command as you type it):

```
ATE1
OK
```

Unlock engineering commands:

```
AT!ENTERCND="A710"
OK
```

Check customization options (these are the author's options): 

```
AT!CUSTOM?
!CUSTOM: 
             GPSENABLE          0x01
             GPSSEL             0x01
             IPV6ENABLE         0x01
             SIMLPM             0x01
             SINGLEAPNSWITCH    0x01


OK
```

Configure *USB fast enumeration* (swap `2` for `0` if you want to play it safe):

```
AT!CUSTOM?
!CUSTOM: 
             GPSENABLE          0x01
             GPSSEL             0x01
             IPV6ENABLE         0x01
             SIMLPM             0x01
             FASTENUMEN         0x02
             SINGLEAPNSWITCH    0x01


OK
```

Configure the modem to ignore W_DISABLE:

```
AT!PCOFFEN=2
OK
```

Verify:

```
AT!PCOFFEN?
2
   
OK
```

Reset the modem:

```
AT!RESET
OK
```

## Problem with update firmware

I have nothing on the first step when i tried to flash new firmware.

But i found this guide

https://github.com/danielewood/sierra-wireless-modems#official-sierra-documentsfirmwares-may-require-free-sierra-account

with this commands

```
AT!USBVID=1199
```

```
AT!USBPID=9079,9078
```

```
AT!USBPRODUCT="Sierra Wireless EM7455 Qualcomm Snapdragon X7 LTE-A"
```

```
AT!PRIID="9904609","002.026","Lenovo-Storm"
```

```
AT!RESET
```

And only after that I successfully flashed new firmware. 

```
sudo qmi-firmware-update --update -d 1199:9079 SWI9X30C_02.33.03.00.cwe SWI9X30C_02.33.03.00_GENERIC_002.072_000.nvu
```


```
sudo systemctl start ModemManager
```

And my modem is working now

## Config

Btw, you should create config file to connect to LTE

```
sudo vim /etc/qmi-network.conf
```

with this content (for MTS)

```
APN=internet.mts.ru
APN_USER=mts
APN_PASS=mts
```


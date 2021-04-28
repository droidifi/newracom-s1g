![Silex SX-NEWAH EVK](https://github.com/droidifi/newracom-s1g/blob/master/SX-NEWAH-new-driver.jpg)

# Description
Newracom driver and compoments for Linux native S1G / 802.11ah

The components needed for native S1G suport are:

Linux kernel 5.10.x 

Wpa_supplicant & hostapd

Wireless-regdb & CRDA

Newracom driver, dtbo file and cli_app utility application

The components can be checked out directly from the droidifi repositories

The branch name is "linux-5.10.x-S1G" in each of the wireless-regdb, linux,
hostap and nrc7292_sw_pkg repos

Or the original repositories can be checked out and the patchfiles in this repo
can be applied

A ready-to-go SD card image for a Raspberry Pi model 3 or model 4 is available at:

https://www.droidifi.com/2021-04-28-raspios-buster-armhf_newracom_8GB.img.zip 

The SD card image automatically loads the mac80211.ko and nrc.ko modules. The cli_app is included. 
After booting assign an IP address to the wlan1 interface and run either wpa_supplicant or hostapd.

The default country is 'EU'. If you would like to use another country code, edit the file in
/mnt/rootfs/etc/modprobe.d/nrc.conf and change the two letter country to one of [US, EU, JP, KR, TW, CN] 
in the "fw_country=EU" module parameter. You will also need to edit the 
/etc/wpa_supplicant/sta_halow_sae_s1g.conf or /etc/hostapd/ap_halow_sae_s1g.conf file. 
If using an EU country, actual country codes such as DE, FR, SE, GB, etc must be used for wpa_supplicant and hostapd. 
They will automatically be translated to 'EU' for the Newracom firmware. Otherwise us US, JP, KR, TW or CN directly.

# Building

Clone the following repositories:

git://git.kernel.org/pub/scm/linux/kernel/git/mcgrof/crda.git

https://github.com/droidifi/wireless-regdb.git

https://github.com/droidifi/hostap.git

https://github.com/droidifi/linux.git

https://github.com/droidifi/nrc7292_sw_pkg.git

Make sure you checkout the linux-5.10.x-S1G branch.

## wireless-regdb

Run "make && make install"

## crda

Copy the public keys from the wireless-regdb repo

```
cp [path]/wireless-regdb/*.key.pub.pem [path]/crda/pubkeys/
```
Run "make && make install"

## hostapd & wpa_supplicant

Change to the hostap/hostapd directory

Run "make && make install"

Change to the hostap/wpa_supplicant directory

Run "make && make install"

## linux kernel for Raspberry Pi
You will have to cross-compile the Pi 64-bit kernel on an x86_64 machine

Install the cross-compiler via "apt install gcc-aarch64-linux-gnu"

Change to the linux directory

Export the cross-compile variables
```
export ARCH=arm64
export CROSS_COMPILE=aarch64-linux-gnu-
export CFLAGS="-march=armv8-a+crc -mtune=cortex-a72"
export INSTALL_MOD_PATH=[path to mounted SD card]
```
Run "make brcm2711_defconfig"

Run "make -j4 Image modules dtbs"

Run "make modules_install"

Copy the kernel to [sd card mount path]/boot
```
cp arch/arm64/boot/Image [path]/boot/kernel8.img
```
Copy the *.dtb files to [sd card mount path]/boot
```
cp arch/arm64/boot/dts/broadcom/*.dtb [path]/boot/
```
Copy the *.dtbo overlay files to [path]/boot/overlays/
```
cp arch/arm64/boot/dts/overlays/*.dtbo [path]/boot/overlays/
```
Edit the [path]/boot/config.txt file and add the line "arm64_bit=1"

Boot the SD card and run "uname -a" to make sure you are running a 64-bit kernel

## Newracom module, dtbo, and cli_app

After cloning the nrc7292_sw_pkg repo, change to package/host/nrc_driver/source/nrc_driver/nrc

Make sure you have the same variables exported used to compile the kernel

Export the KDIR variable set to the path to the kernel source directory
```
export KDIR=[path to kernel source]
```
Run "make clean && make && make modules_install"

This will install the newracom nrc.ko module to the [path]/lib/modules/[kernel version] directory

Please see the README file in the nrc7292_sw_pkg repo for instructions on compiling dtbo and cli_app and
installing the firmware in the /lib/firmware directory

The newracom.dtbo file should be copied to the [path]/boot/overlays directory.

# Running
Load the modules
```
sudo modprobe mac80211
sudo insmod /lib/modules/$(uname -r)/extra/nrc.ko fw_name=nrc7292_cspi.bin hifspeed=16000000 fw_country=[US,EU,JP,TW,KR,CN]
```
Adjust the driver Wi-Fi parameters (optional)
```
sudo cli_app set maxagg 1 8
sudo cli_app set txpwr 23
sudo cli_app set gi short
```
Running the hostapd soft Access Point
```
ip addr add 192.168.200.1/24 dev wlan1
hostapd /etc/hostapd/ap_halow_sae_s1g.conf
```
Running the wpa_supplicant Wi-Fi client (on a different device than hostapd)
```
ip addr add 192.168.200.2/24 dev wlan1
wpa_supplicant -iwlan1 -Dnl80211 -c /etc/wpa_supplicant/sta_halow_sae_s1g.conf 
```

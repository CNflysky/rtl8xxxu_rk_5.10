# rtl8xxxu_rk_5.10
rtl8xxxu usb wifi driver, for rockchip rk3588 bsp kernel 5.10.  
Cause bsp kernel lacks support of `RTL8188F` usb wifi module, so I have backported it from `6.2`.  
[简体中文/Simplified Chinese](./README_zh.md)

# Integrate to existing BSP tree
Copy all sources to `drivers/net/wireless/realtek/rtl8xxxu`.  
`make menuconfig`，enable `CONFIG_RTL8XXXU`.  
Then simply build your kernel.

# Build out-of-tree module on your board 
Make sure you have kernel headers installed;  
`Armbian` users can install it using following command:  
```bash
sudo apt install -y linux-headers-legacy-rk35xx
```
build the module:
```bash
make CONFIG_RTL8XXXU=m -C /lib/modules/`uname -r`/build M=`pwd` modules
```
`rtl8xxxu.ko` will appears in current directory.  
You can load it using `insmod rtl8xxxu.ko`.  
If you wish autoload module when USB wifi adapter plugged in, you can run the following command:
```bash
sudo cp rtl8xxxu.ko /lib/modules/`uname -r`/kernel/drivers/net/wireless/rtl8xxxu
sudo depmod
```
`reboot`, that's it.  

# Installing firmware
Copy `rtl8188fufw.bin` file to `/lib/firmware/rtlwifi/` directory.

# Testing
My USB WiFi adapter is `RTL8188FTV USB Dongle`，use `lsusb` to inspect it:
```bash
Bus 007 Device 002: ID 0bda:f179 Realtek Semiconductor Corp. RTL8188FTV 802.11b/g/n 1T1R 2.4G WLAN Adapter
```
Then plug it into your board, `dmesg` prints:
```bash
[   50.883071] usb 7-1: new high-speed USB device number 2 using xhci-hcd
[   51.024006] usb 7-1: New USB device found, idVendor=0bda, idProduct=f179, bcdDevice= 0.00
[   51.024037] usb 7-1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[   51.024057] usb 7-1: Product: 802.11n
[   51.024076] usb 7-1: Manufacturer: Realtek
[   51.024095] usb 7-1: SerialNumber: ************
[   51.205709] usb 7-1: Vendor: Realtek
[   51.205718] usb 7-1: Product: 802.11n
```
when module loaded, `dmesg` prints:
```bash
[   51.205723] usb 7-1: rtl8188fu_parse_efuse: dumping efuse (0x200 bytes):
# massive efuse registers
[   51.205991] usb 7-1: RTL8188FU rev B (SMIC) 1T1R, TX queues 2, WiFi=1, BT=0, GPS=0, HI PA=0
[   51.205996] usb 7-1: RTL8188FU MAC: **:**:**:**:**:**
[   51.206000] usb 7-1: rtl8xxxu: Loading firmware rtlwifi/rtl8188fufw.bin
[   51.210598] usb 7-1: Firmware revision 4.0 (signature 0x88f1)
[   52.313334] usbcore: registered new interface driver rtl8xxxu
[   52.327916] rtl8xxxu 7-1:1.0 wlx************: renamed from wlan0
```
you can use `ip addr` command to see the wireless interface:
```bash
6: wlx************: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether **:**:**:**:**:** brd ff:ff:ff:ff:ff:ff
```
Finally you can use your favorite wireless management tool(`wpa_supplicant`/`nmcli`/`nmtui`/`Advanced Network Configuration`) to connect WiFi.

# Enable HT40 Support
`/etc/modprobe.d/rt8xxxu.conf`:
```bash
options rtl8xxxu ht40_2g=1
```
After enable this param on my hardware, Transfer speed increased from `3MB/s` to `5MB/s` in my local network.
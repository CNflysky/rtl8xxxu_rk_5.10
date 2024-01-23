# rtl8xxxu_rk_5.10
适用于RK3588 BSP `5.10` 的 `rtl8xxxu USB WiFi`内核驱动。  
由于BSP内核缺少对`RTL8188F`芯片的支持，因此由`6.2`内核反向移植了该驱动。


# 集成到现有内核源码
先将仓库下所有源码复制到内核源码的drivers/net/wireless/realtek/rtl8xxxu目录下。  
随后使用`make menuconfig`命令，启用`CONFIG_RTL8XXXU`内核配置项。  
最后编译内核即可。

# 直接在开发板上编译树外模块
首先确保你的系统上安装了对应的内核头文件；  
如果你使用`armbian`，输入以下命令安装内核头文件：
```bash
sudo apt install -y linux-headers-legacy-rk35xx
```
安装完成后,使用以下命令编译模块：
```bash
make CONFIG_RTL8XXXU=m -C /lib/modules/`uname -r`/build M=`pwd` modules
```
编译完成后，当前目录下会出现`rtl8xxxu.ko`文件，使用`insmod`命令加载模块即可。   
如果要使得模块在USB网卡插入时自动加载，只需要输入以下命令：
```bash
sudo cp rtl8xxxu.ko /lib/modules/`uname -r`/kernel/drivers/net/wireless/rtl8xxxu
sudo depmod
```
重启后，插入USB网卡，模块将会自动加载。
# 安装固件
将当前目录下的`rtl8188fufw.bin`文件复制到`/lib/firmware/rtlwifi`目录下即可。

# 测试
使用淘宝5.9元购买的`RTL8188FTV`USB WiFi，`lsusb`检查信息如下：
```bash
Bus 007 Device 002: ID 0bda:f179 Realtek Semiconductor Corp. RTL8188FTV 802.11b/g/n 1T1R 2.4G WLAN Adapter
```
插入设备后，`dmesg`输出以下信息：
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
加载模块后`dmesg`输出以下信息：
```bash
[   51.205723] usb 7-1: rtl8188fu_parse_efuse: dumping efuse (0x200 bytes):
# 大量efuse信息
[   51.205991] usb 7-1: RTL8188FU rev B (SMIC) 1T1R, TX queues 2, WiFi=1, BT=0, GPS=0, HI PA=0
[   51.205996] usb 7-1: RTL8188FU MAC: **:**:**:**:**:**
[   51.206000] usb 7-1: rtl8xxxu: Loading firmware rtlwifi/rtl8188fufw.bin
[   51.210598] usb 7-1: Firmware revision 4.0 (signature 0x88f1)
[   52.313334] usbcore: registered new interface driver rtl8xxxu
[   52.327916] rtl8xxxu 7-1:1.0 wlx************: renamed from wlan0
```
此时使用ip addr命令已经可以看到新增的无线网络接口：
```bash
6: wlx************: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether **:**:**:**:**:** brd ff:ff:ff:ff:ff:ff
```
然后你就可以使用你喜欢的方式（`wpa_supplicant`/`nmcli`/`nmtui`/`高级网络管理器`），连接到无线网络了。

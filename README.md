#### RTL8812au驱动移植

- 版本：4.3.20
- 下载：`git clone git@github.com:fangzr/rtl8812au.git`
- 内核版本：4.90

##### 编译步骤
1. 解压源码包，在此路径打开终端，修改`Makefile`两处地方，如下所示：
```
#其余的Platform必须为n
CONFIG_PLATFORM_GENERIC = y

ifeq ($(CONFIG_PLATFORM_GENERIC), y)
EXTRA_CFLAGS += -DCONFIG_LITTLE_ENDIAN
EXTRA_CFLAGS += -DCONFIG_IOCTL_CFG80211 -DRTW_USE_CFG80211_STA_EVENT
#SUBARCH := $(shell uname -m | sed -e s/i.86/i386/ -e s/armv.*/arm/)
ARCH :=arm
CROSS_COMPILE :=arm-linux-gnueabihf-
KVER  := 4.90
#这里填写你的内核源码路径
KSRC := /.../linux-digilent
MODULE_NAME :=wlan0
endif
```
2. `make`开始编译，编译完成后得到`wlan0.ko`文件
3. 将`wlan0.ko`上传到文件系统某个文件夹待用


---

### 网络配置
把交叉编译得到的可执行文件、头文件、库文件都上传到开发板对应位置后，可以开始实现WiFi热点的最后一步。

1. 加载RTL8812au驱动
`insmod /etc/wlan1.ko`
打印消息：
```
root@Zybo:~# insmod /etc/wlan0.ko                                               
wlan0: loading out-of-tree module taints kernel.                                
RTL871X: module init start                                                      
RTL871X: rtl8812au v4.3.20_16317.20160108                                       
RTL871X: build time: Apr 28 2019 20:04:05                                       
usbcore: registered new interface driver rtl8812au                              
RTL871X: module init ret=0 
```
2. 插上无线网卡，打印消息
```                         
usb 1-1: new high-speed USB device number 3 using ci_hdrc                       
RTL871X: rtw_ndev_init(wlan0) if1 mac_addr=00:0f:00:73:76:b3  

```
3. 编写hostapd配置文件
`vim /etc/hostapd.conf`
我们配置一个最简单的无线热点，不加密
```
interface=wlan0
driver=nl80211
ssid=WiFi_AP_Name
channel=6
hw_mode=g
wpa=0
bridge=br0
```

4.输入以下命令
```
ifconfig wlan0 up
brctl addbr br0
brctl addif br0 eth0 
ifconfig br0 up
hostapd -dd /etc/hostapd.conf

```
此时你应该可以用手机连接热点了。

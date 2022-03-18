WiFi dongle: rtl8822bu 0bda:b82c

Openwrt Linux : 21.02.1
Linux Version : 5.4.154

Openwrt platform : mt7688 LinkIt Smart 7688

Driver version : v5.8.7.1_35809.20191129_COEX20191129-7777

Change from : 

Caution:
	(1)
	This package supports 2 lan interface.
	Remvoe the pkg Makefie argument of "-DCONFIG+_CONCURRET_MODE" and supports 1 lan interface
	
	(2)
	If you USB dongle come without bluetooth (RTL881x), change src Makefle argument from
		CONFIG_BT_COEXIST = y
	to
		CONFIG_BT_COEXIST = n

	change pkg Makefile argument from 
		FILES:=$(PKG_BUILD_DR)/88x2bu.ko
	to
		FILES:=$(PKG_BUILD_DR)/8812bu.ko

#make 88x2bu.ko 
#pacakge build 

#merge rtl8822bu.tar.*
cat rtl8822bu.tar.* > rtl8822bu.tar
untar 
copy rtl8822bu/ to openrt/package/kernel 

select M at 
	Kernel modules
	Wireless Drivers
	kmod-rtl8822bu

make openrt/package/kernel/rtl8822bu/compile V=s

rtl88x2bu.ko at openwrt/build_dir/target-mipsel_musl/linux-ramips_mt76x8/rtl8822bu/

------------
#verify

[Lab] WiFi bridge

[PC] --- [waln2:AP:greatcat7 | mt7688 | wlan1:station] --- [AP:greatcat5G:192.168.2.1]

#For supporting WiFi brdige mode, openwrt need to modify cfg80211.ko
#Refer to Realtek's document of Quick_Start_Guide_for_Bridge.pdf

cat /etc/hostapd.conf

interface=wlan2
driver=nl80211
ssid=greatcat7
hw_mode=g
channel=1
bridge=br0


echo 1 > /proc/net/rtl88x2bu/log_level

ifconfig -a
ifconfig wlan1 up
iwlist wlan1 scan
iw wlan1 connect -w "greatcat5G"


brctl addbr br0 
brctl addif br0 wlan1 
brctl setfd br0 0 
hostapd -d -B /etc/hostapd.conf 
ifconfig br0 up 
brctl show


change bridge status form
brctl show                                             
bridge name     bridge id               STP enabled     interfaces              
br-lan          7fff.9c65f920a914       no              eth0

to
brctl show                                             
bridge name     bridge id               STP enabled     interfaces              
br-lan          7fff.9c65f920a914       no              eth0                    
br0             8000.bcec4342ebd9       no              wlan2                   
                                                        wlan1
-------
#test
#PC links to AP named greatcat7 and gets dhcp IP 192.168.2.x from greatcat5G
ping 192.168.2.1

-------
If the document help you, how about buy street cats a fish can ?

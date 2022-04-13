# asuswrt_newifi
Notes to build the asus RT-AC1200HP firmware for newifi

https://gist.github.com/KyonLi/6fa5d6e391d82856e35168e851303784

# [GUIDE] Build AsusWRT for newifi mini on Ubuntu 14.04 32bit

## Manually preparing the build environment

install packages
```bash
apt-get install libncurses5 libncurses5-dev m4 bison gawk flex libstdc++6-4.4-dev g++-4.4 g++ \
git gitk zlib1g-dev autoconf autopoint libtool shtool autogen mtd-utils intltool sharutils \
docbook-xsl-* libstdc++5 texinfo dos2unix xsltproc make pkg-config
```

prepare source to, ex, ～/asuswrt
```bash
cd ~
wget https://dlcdnets.asus.com/pub/ASUS/wireless/RT-AC1200HP/GPL_RT_AC1200HP_30043808457.zip
unzip GPL_RT_AC1200HP_30043808457.zip
tar zxvf GPL_RT-AC1200HP_3.0.0.4.380.8457-*.tgz
```

setup development system
```bash
cd asuswrt/
cp -r tools/brcm /opt/
tar -jxvf tools/buildroot-gcc342.tar.bz2 -C /opt/
export PATH=$PATH:/opt/brcm/hndtools-mipsel-linux/bin:/opt/brcm/hndtools-mipsel-uclibc/bin:/opt/buildroot-gcc342/bin
```

## Newifi mini adaptation

```bash
cd release/
nano src/router/shared/sysdeps/ralink/mt7620.c
```

search for `RTAC1200HP` edit it as follows
```shell
#elif defined(RTAC1200HP)
enum {
        WAN_PORT=4,
        LAN1_PORT=1,
        LAN2_PORT=0,
        LAN3_PORT=2,
        LAN4_PORT=3,
        P5_PORT=5,
        CPU_PORT=6,
        P7_PORT=7,
};
```

```bash
nano src/router/rc/init.c
```

search for `RTAC1200HP` edit it as follows
```shell
case MODEL_RTAC1200HP:
......
		nvram_set_int("btn_rst_gpio", 11|GPIO_ACTIVE_LOW);
		//nvram_set_int("btn_wps_gpio", 61|GPIO_ACTIVE_LOW);
		nvram_set_int("led_usb_gpio", 52|GPIO_ACTIVE_LOW);
		nvram_set_int("led_pwr_gpio", 9|GPIO_ACTIVE_LOW);
		nvram_set_int("led_wps_gpio", 9|GPIO_ACTIVE_LOW);
		nvram_set_int("led_5g_gpio", 50|GPIO_ACTIVE_LOW);
		nvram_set_int("led_2g_gpio", 72|GPIO_ACTIVE_LOW);
		//nvram_set_int("led_all_gpio", 10|GPIO_ACTIVE_LOW);
		nvram_set_int("led_lan_gpio", 55|GPIO_ACTIVE_LOW);
		nvram_set_int("led_wan_gpio", 51|GPIO_ACTIVE_LOW);
......
```

```bash
nano src-ra-mt7620/linux/linux-2.6.36.x/drivers/net/raeth/raether.c
```

search for `RTAC1200HP` edit it as follows
```shell
#if defined(RTAC1200HP)
	"2", "1", "3", "4", "", "x" /* RT-AC1200HP, P0P1P2P3P4P5 map to LAN/WAP port*/ 
#else
```

## Fix 5GHz Wi-Fi

```bash
nano src-ra-mt7620/linux/linux-2.6.36.x/drivers/net/wireless/rlt_wifi_ap/Makefile
```

edit first line as follows
```shell
obj-m += rlt_wifi.o
```

## Enable TProxy

```bash
nano src-ra-mt7620/linux/linux-2.6.36.x/config_base
```

set y to these three config blow:
```shell
CONFIG_NETFILTER_TPROXY=y
CONFIG_NETFILTER_XT_TARGET_TPROXY=y
CONFIG_NETFILTER_XT_MATCH_SOCKET=y
```

## Build firmware

```bash
cd src-ra-mt7620/
make distclean
make RT-AC1200HP
```

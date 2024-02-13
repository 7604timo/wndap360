# wndap360
Netgear WNDAP360 OpenWRT

After upgrade, LAN MAC addr will be hardcoded in different place in ART. Default MAC dhcp entry:
```
host ap_temp {
 hardware ethernet 00:03:7F:E0:00:96;
 fixed-address 192.168.1.12;
}
```
DTS: ```target/linux/ath79/dts/ar7161_netgear_wndap360.dts```

# Edit Lan MAC address
Edit DTS to comment out ART partition read-only flag. Figure out new LAN MAC address:
```ifconfig -a | grep HWaddr```
Pick an unused middle entry..

```cat /proc/mtd
mtd unlock art
dd if=/dev/mtd7 of=/tmp/art
# 00:8E:??:??:??:??
printf "\x00\x8E\x??\x??\x??\x??" | dd conv=notrunc of=/tmp/art bs=1 seek=$((0x00))
mtd write /tmp/art art
reboot
```
# Make sure LAN port comes up at 1Gbps
Edit DTS to make sure eth0 section looks like this:
```
&eth0 {
        status = "okay";

        phy-mode = "rgmii";
        phy-handle = <&phy1>;

        nvmem-cells = <&macaddr_art_0>;
        nvmem-cell-names = "mac-address";

        pll-data = <0x11110000 0x00001099 0x00991099>;
};
```

# Make sure console port stays at 9600
Edit DTS top section (above 'aliases'):
```
        chosen {
                bootargs = "console=ttyS0,9600";
        };
```

# Make power LED running state 'green'
Modify DTS to add green-led definition to 'leds' section:
```
led_power_green: power_green {
                        label = "green:power";
                        gpios = <&gpio 2 GPIO_ACTIVE_LOW>;
                };
```
Change 'aliases' section to make 'led-running' the new colour:
```
        aliases {
                led-boot = &led_power_orange;
                led-failsafe = &led_power_orange;
                led-running = &led_power_green;
                led-upgrade = &led_power_orange;
        };
```

# Make sure the owl-loader kmod is loaded:
Edit ```target/linux/ath79/image/generic.mk``` and search for ```wndap360```, then add in the ```kmod-owl-loader``` package to the DEVICE_PACKAGES list:
```
define Device/netgear_wndap360
  $(Device/netgear_generic)
  SOC := ar7161
  DEVICE_MODEL := WNDAP360
  DEVICE_PACKAGES := kmod-leds-reset kmod-owl-loader
  IMAGE_SIZE := 7744k
  BLOCKSIZE := 256k
  KERNEL := kernel-bin | append-dtb | gzip | uImage gzip
  KERNEL_INITRAMFS := kernel-bin | append-dtb | uImage none
  IMAGES := sysupgrade.bin
  IMAGE/sysupgrade.bin := append-kernel | pad-to 64k | append-rootfs | pad-rootfs | \
        check-size | append-metadata
endef
```

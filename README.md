# wndap360
Netgear WNDAP360 OpenWRT

After upgrade, LAN MAC addr will be hardcoded in different place in ART. Default MAC dhcp entry:
```
host ap_temp {
 hardware ethernet 00:03:7F:E0:00:96;
 fixed-address 192.168.1.12;
}
```

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

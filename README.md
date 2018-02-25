# archon on tplink hs110

![](board.jpg)


### working with the prototype setup

![](protosetup.jpg)

for magnetics we use the [omega2 ethernet shield](https://github.com/OnionIoT/Onion-Hardware/blob/master/Schematics/Omega-Ethernet-Expansion.pdf)

flashing can be done by interrupting uboot

```
Hit any key to stop autoboot:  0 
ar7240> 
```

stock flash layout is as follows

```
0x000000000000-0x000000010000 : "u-boot"
0x000000010000-0x000000110000 : "uImage"
0x000000110000-0x0000003c0000 : "rootfs"
0x0000003c0000-0x0000003d0000 : "TPHead"
0x0000003d0000-0x0000003e0000 : "config"
0x0000003e0000-0x0000003f0000 : "log"
0x0000003f0000-0x000000400000 : "ART"
0x000000000000-0x0000003d0000 : "firmware"
```

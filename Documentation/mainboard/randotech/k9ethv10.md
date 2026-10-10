# R&O Industrial Technology Co., Limited K9-ETH v1.0

The R&O Industrial Technology Co., Limited K9-ETH v1.0 is a PCIe addon card-formated
 desktop board. It was used as GPU mining rig and (later on) NAS management board.

## Technology

```{eval-rst}
+------------------+--------------------------------------------------+
| Southbridge      | Intel H67 (called bd82x6x in coreboot code)      |
+------------------+--------------------------------------------------+
| CPU socket       | LGA 1155                                         |
+------------------+--------------------------------------------------+
| RAM              | 1 x SO-DIMM DDR3-1600                            |
+------------------+--------------------------------------------------+
| SuperIO          | Nuvoton NCT5532D                                 |
+------------------+--------------------------------------------------+
| Audio            | None                                             |
+------------------+--------------------------------------------------+
| Network          | Realtek RTL8111 Gigabit Ethernet                 |
+------------------+--------------------------------------------------+
```

There is no serial port. Serial console output is possible by soldering
to a point at the corresponding Super I/O pin and patching the
mainboard-specific code accordingly.

## Status

### Working

Tests were done with SeaBIOS 1.14.0 and slackware64-live from 2019-07-12
(linux-4.19.50).

+ Intel Xeon E3-1245v2
+ Only SO-DIMM slots at 1600 MHz (tested 1x8GB)
+ Integrated graphics (libgfxinit)
+ VGA port
+ USB (2 internal, 2 external)
+ mSATA slot
+ Onboard Ethernet
+ Poweroff

### Not working


### Untested

+ PCI-e 'slot'
+ S3 suspend/resume
+ Super I/O
+ Wake-on-LAN
+ SATA port

## Flashing coreboot

```{eval-rst}
+---------------------+------------+
| Type                | Value      |
+=====================+============+
| Socketed flash      | no         |
+---------------------+------------+
| Model               | W25Q32JV   |
+---------------------+------------+
| Size                | 4 MiB      |
+---------------------+------------+
| In circuit flashing | sort of?   |
+---------------------+------------+
| Package             | SOIC-8     |
+---------------------+------------+
| Write protection    | No         |
+---------------------+------------+
| Dual BIOS feature   | No         |
+---------------------+------------+
| Internal flashing   | yes        |
+---------------------+------------+
```

The flash is divided into the following regions, as obtained with
`ifdtool -f rom.layout backup.rom`:
```
00000000:00000fff fd
00180000:003fffff bios
00003000:0017ffff me
00001000:00002fff gbe
00fff000:00000fff pd
```

In general, flashing is possible internally. It might be necessary
 to specify the chip type: `W25Q32JV` is the correct one.

### Internal flashing

The SPI flash can be accessed internally using [flashrom].
It looks like BIOS come 'unlocked' from a factory (tweaked Intel ME, allowed write
 to IFD, ME and GBE regions), so firmware can be flashed to all regions

```bash
     $ flashrom -p internal -c "W25Q32JV" -w coreboot.rom
```

Of course, you can flash just BIOS region

```bash
     $ flashrom -p internal -c "W25Q32JV" --ifd -i bios -w coreboot.rom --noverify-all
```

In addition to the information here, please see the
<project:../../tutorial/flashing_firmware/index.md>.

### External flashing

ISP (in circuit programming) seems to be quite possible on this board but 3.3VDC
 is shared with whole motherboard, so the only 'working' solution is applying
 power to 6pin connector and attaching to SPI chip via group of pogo pins
 (often referred as 'chip burning needle', 'SOIC-8 208 mil spacing 5.8' in this case)
 since mSATA standoff is blocking usage of pomona clip.

**DISCLAIMER**: even with this setup XGecu T48 only able to read/write every **second** time!
If this sounds too risky for you -- don't try ISP!

## Variants

As for now, the only existing variant is K9-ETH v1.1, differences are unknown.

[flashrom]: https://flashrom.org/

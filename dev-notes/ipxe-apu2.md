# iPXE on APU2

## Prerequisites
- APU2 with serial connection to host PC

## Building iPXE

>NOTE: Prebuilded binary is available on
[cloud.3mdeb.com](https://cloud.3mdeb.com/index.php/s/4RoyYJTCnxmZJ76)

Clone 3mdem/ipxe repository and checkout to `skinit_lz` branch:

```
$ git clone https://github.com/3mdeb/ipxe.git
$ cd ipxe
$ git checkout skinit_lz
```

Build iPXE:
```
$ cd src
$ make
```

after a few minutes, you should see similar output:

```
===========================================================

To create a bootable floppy, type
    cat bin/ipxe.dsk > /dev/fd0
where /dev/fd0 is your floppy drive.  This will erase any
data already on the disk.

To create a bootable USB key, type
    cat bin/ipxe.usb > /dev/sdX
where /dev/sdX is your USB key, and is *not* a real hard
disk on your system.  This will erase any data already on
the USB key.

To create a bootable CD-ROM, burn the ISO image
bin/ipxe.iso to a blank CD-ROM.

These images contain drivers for all supported cards.  You
can build more customised images, and ROM images, using
    make bin/<rom-name>.<output-format>

===========================================================
```

Now you can see contents of `bin` folder to verify building process.

`$ ls bin | grep ipxe`

We are looking for `ipxe.lkrn`.

## Booting to MBOOT2 iPXE

Enter to the `bin` folder and run simple HTTP file server:

`$ python3 -m http.server`

Power on APU2 device and enter to the stock iPXE shell by typing `Ctrl+B`. Inside
shell run:

```
# dhcp
# chain http://<YOUR_HOST_PC_IP>:8000/ipxe.lkrn
```

Excepted output:

```
iPXE> dhcp
Configuring (net0 00:0d:b9:4e:0e:00)...... ok
iPXE> chain http://<YOUR_HOST_PC_IP>:8000/ipxe.lkrn
http://<YOUR_HOST_PC_IP>8000/ipxe.lkrn... ok
iPXE initialising devices...ok



iPXE 1.20.1+ (g8151b) -- Open Source Network Boot Firmware -- http://ipxe.org
Features: DNS HTTP iSCSI TFTP SRP AoE ELF MBOOT MBOOT2 PXE bzImage Menu PXEXT
```

Now you can enter to the shell of second iPXE by typing again `Ctrl+B`

## Troubleshooting

**Problem:**

```
util/zbin.c:7:10: fatal error: lzma.h: No such file or directory
    7 | #include <lzma.h>
      |          ^~~~~~~~
```

**Solution:**

Install `liblzma-dev` on host machine:

`$ apt-get install liblzma-dev`

------------------


**Problem:**

```
util/genfsimg: could not find isolinux.bin
make: *** [arch/x86/Makefile.pcbios:61: bin/ipxe.iso] Error 1
rm bin/version.ipxe.dsk.o bin/ipxe.lkrn.zbin bin/ipxe.dsk.zbin bin/version.ipxe.lkrn.o bin/ipxe.lkrn.bin bin/ipxe.dsk.bin bin/ipxe.lkrn.zinfo bin/ipxe.dsk.zinfo
```

Install `isolinux` on host machine:

`$ sudo apt install isolinux`

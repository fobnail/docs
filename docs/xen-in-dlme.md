# Running Xen in DLME

During test we are using [meta-fobnail](https://github.com/fobnail/meta-fobnail)
which is our custom Yocto layer. meta-fobnail has been described more in-depth
in [meta-fobnail in DLME](./meta-fobnail-in-dlme.md). Here we are using Yocto as a
base OS because it already provides TrenchBoot's GRUB and Landing Zone which are
required to boot Xen and Linux itself to test whether Xen is working.

Build Yocto image and flash it to SDcard.

```shell
$ mkdir yocto
$ cd yocto
$ git clone https://github.com/fobnail/meta-fobnail
$ kas-container build meta-fobnail/kas-debug.yml
```

Build latest Xen version

```shell
$ git clone https://xenbits.xen.org/git-http/xen.git
$ cd xen
$ git co RELEASE-4.16.1
$ ./configure
$ make -j$(nproc)
```

Copy `xen/xen` file to Yocto boot partition and add following entry to
`/boot/grub/grub.cfg`.

```shell
menuentry 'XEN' {
  slaunch skinit
  slaunch_module (hd0,msdos1)/lz_header.bin
  multiboot2 /xen loglvl=all guest_loglvl=all com1=115200,8n1 console=com1
  module2 /bzImage root=/dev/mmcblk0p2 console=ttyS0,115200 earlyprintk=serial,ttyS0,115200 rootwait
}
```

When booting platform, Landing Zone starts first, then Xen, and then Linux.

```
shasum calculated:
0x001001dc: d0 8d a8 6f 97 cc 6b 6b 99 9b 1c e1 e7 0b 60 3c   ...o..kk......`<
0x001001ec: db a3 2d ac cc cc cc cc cc cc cc cc cc cc cc cc   ..-.............
shasum calculated:
0x001001f0: 56 2b de 09 b5 8b 8a 9b d5 40 1c 73 52 0e 37 74   V+.......@.sR.7t
0x00100200: 58 3e 44 0c 9b a1 9e aa 35 a5 b5 be ba e3 b3 d2   X>D.....5.......
PCR extended
shasum calculated:
0x001001dc: 0c d1 e3 3c e3 f1 21 44 13 87 3d 88 b7 1e c5 6b   ...<..!D..=....k
0x001001ec: 35 fc 40 b0 56 2b de 09 b5 8b 8a 9b d5 40 1c 73   5.@.V+.......@.s
shasum calculated:
0x001001f0: 45 5b 59 f4 f8 8d 08 95 a6 c9 dc 3e 2c 28 c3 c7   E[Y........>,(..
0x00100200: 76 dc 0a 4f 55 89 54 a5 a9 bb 71 bf 02 97 da 8b   v..OU.T...q.....
PCR extended
kernel_size 0x00278bd0:
shasum calculated:
0x001001dc: 8f d0 9f b3 cf 33 c1 e0 e0 47 8c cf d0 8b 2c c9   .....3...G....,.
0x001001ec: b8 b6 1a a4 45 5b 59 f4 f8 8d 08 95 a6 c9 dc 3e   ....E[Y........>
shasum calculated:
0x001001f0: bc f4 aa 0d 2e 10 72 12 9d 42 ef bf 57 25 ad 56   ......r..B..W%.V
0x00100200: de 40 86 45 16 11 75 3d 7e 3d 7e 4e 78 ba ae 61   .@.E..u=~=~Nx..a
PCR extended
Module 'root=/dev/mmcblk0p2 console=ttyS0,115200 earlyprintk=serial,ttyS0,115200' [0x00111000: 0x00939e80: ]
shasum calculated:
0x001001dc: fb 15 05 fd bc b7 09 e3 62 11 8e 13 06 7e c0 0b   ........b....~..
0x001001ec: 53 45 4b d1 bc f4 aa 0d 2e 10 72 12 9d 42 ef bf   SEK.......r..B..
shasum calculated:
0x001001f0: 78 57 2d b4 2d 41 95 0a f8 2e 8c f1 5d 1c f5 55   xW-.-A......]..U
0x00100200: 7c 6b 03 ac 90 c6 27 25 10 d0 23 1d 79 8d 25 17   |k....'%..#.y.%.
PCR extended
pm_kernel_entry:
0xcf800000: e9 5a 11 1c 00 0f 1f 00 02 b0 ad 1b 03 00 00 00   .Z..............
0xcf800010: fb 4f 52 e4 c2 c2 c2 c2 d6 50 52 e8 00 00 00 00   .OR......PR.....
0xcf800020: 88 00 00 00 a2 ae ad 17 01 00 00 00 10 00 00 00   ................
0xcf800030: 04 00 00 00 06 00 00 00 06 00 00 00 08 00 00 00   ................
0xcf800040: 0a 00 01 00 18 00 00 00 00 00 20 00 ff ff ff ff   .......... .....
0xcf800050: 00 00 20 00 02 00 00 00 04 00 01 00 0c 00 00 00   .. .............
0xcf800060: 02 00 00 00 c2 c2 c2 c2 05 00 01 00 14 00 00 00   ................
0xcf800070: 00 00 00 00 00 00 00 00 00 00 00 00 c2 c2 c2 c2   ................
0xcf800080: 07 00 01 00 08 00 00 00 09 00 01 00 0c 00 00 00   ................
0xcf800090: 5e 10 3c 00 c2 c2 c2 c2 00 00 00 00 08 00 00 00   ^.<.............
0xcf8000a0: 0f 01 15 19 fe 1b 00 b9 00 00 00 00 8e d9 8e c1   ................
0xcf8000b0: 8e e1 8e e9 8e d1 48 c7 c1 a0 00 00 00 0f 22 e1   ......H.......".
0xcf8000c0: 48 8b 25 3d 6f 27 00 6a 00 9d 68 08 e0 00 00 48   H.%=o'.j..h....H
0xcf8000d0: 8d 05 03 00 00 00 50 48 cb 85 db 74 4c e8 9e c2   ......PH...tL...
0xcf8000e0: 12 00 85 c0 74 3c b9 a2 06 00 00 31 d2 0f 30 b9   ....t<.....1..0.
0xcf8000f0: a0 00 80 00 0f 22 e1 a8 01 74 27 48 89 e2 48 81   ....."...t'H..H.
zero_page:
0x00110018: 20 03 00 00 00 00 00 00 15 00 00 00 0c 00 00 00    ...............
0x00110028: 00 00 80 cf d0 1f d7 cf 01 00 00 00 41 00 00 00   ............A...
0x00110038: 6c 6f 67 6c 76 6c 3d 61 6c 6c 20 67 75 65 73 74   loglvl=all guest
0x00110048: 5f 6c 6f 67 6c 76 6c 3d 61 6c 6c 20 63 6f 6d 31   _loglvl=all com1
0x00110058: 3d 31 31 35 32 30 30 2c 38 6e 31 20 63 6f 6e 73   =115200,8n1 cons
0x00110068: 6f 6c 65 3d 63 6f 6d 31 00 67 72 75 62 5f 72 65   ole=com1.grub_re
0x00110078: 02 00 00 00 16 00 00 00 47 52 55 42 20 32 2e 30   ........GRUB 2.0
0x00110088: 34 7e 72 63 31 00 6f 63 0a 00 00 00 1c 00 00 00   4~rc1.oc........
0x00110098: 02 01 00 f0 37 d7 00 00 00 f0 00 f0 03 00 f0 ff   ....7...........
0x001100a8: f0 ff f0 ff 73 69 00 67 03 00 00 00 59 00 00 00   ....si.g....Y...
0x001100b8: 00 10 11 00 80 9e 93 00 72 6f 6f 74 3d 2f 64 65   ........root=/de
0x001100c8: 76 2f 6d 6d 63 62 6c 6b 30 70 32 20 63 6f 6e 73   v/mmcblk0p2 cons
0x001100d8: 6f 6c 65 3d 74 74 79 53 30 2c 31 31 35 32 30 30   ole=ttyS0,115200
0x001100e8: 20 65 61 72 6c 79 70 72 69 6e 74 6b 3d 73 65 72    earlyprintk=ser
0x001100f8: 69 61 6c 2c 74 74 79 53 30 2c 31 31 35 32 30 30   ial,ttyS0,115200
0x00110108: 00 5f 65 6e 64 00 67 72 06 00 00 00 e8 00 00 00   ._end.gr........
0x00110118: 18 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   ................
0x00110128: 00 fc 09 00 00 00 00 00 01 00 00 00 00 00 00 00   ................
0x00110138: 00 fc 09 00 00 00 00 00 00 04 00 00 00 00 00 00   ................
0x00110148: 02 00 00 00 00 00 00 00 00 00 0f 00 00 00 00 00   ................
0x00110158: 00 00 01 00 00 00 00 00 02 00 00 00 00 00 00 00   ................
0x00110168: 00 00 10 00 00 00 00 00 00 20 d8 cf 00 00 00 00   ......... ......
0x00110178: 01 00 00 00 00 00 00 00 00 20 e8 cf 00 00 00 00   ......... ......
0x00110188: 00 e0 17 00 00 00 00 00 02 00 00 00 00 00 00 00   ................
0x00110198: 00 00 00 f8 00 00 00 00 00 00 00 04 00 00 00 00   ................
0x001101a8: 02 00 00 00 00 00 00 00 00 00 d4 fe 00 00 00 00   ................
0x001101b8: 00 50 00 00 00 00 00 00 02 00 00 00 00 00 00 00   .P..............
0x001101c8: 00 00 00 00 01 00 00 00 00 00 00 2f 00 00 00 00   .........../....
0x001101d8: 01 00 00 00 00 00 00 00 00 00 00 2f 01 00 00 00   .........../....
0x001101e8: 00 00 00 01 00 00 00 00 02 00 00 00 00 00 00 00   ................
0x001101f8: 09 00 00 00 b4 00 00 00 04 00 00 00 28 00 00 00   ............(...
0x00110208: 02 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   ................
0x00110218: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   ................
0x00110228: 00 00 00 00 00 00 00 00 00 00 00 00 01 00 00 00   ................
0x00110238: 01 00 00 00 07 00 00 00 00 00 20 00 80 00 00 00   .......... .....
0x00110248: d0 8b 27 00 00 00 00 00 00 00 00 00 40 00 00 00   ..'.........@...
0x00110258: 00 00 00 00 07 00 00 00 03 00 00 00 00 00 00 00   ................
0x00110268: 00 00 11 00 f0 8c 27 00 18 00 00 00 00 00 00 00   ......'.........
0x00110278: 00 00 00 00 01 00 00 00 00 00 00 00 11 00 00 00   ................
0x00110288: 07 00 00 00 00 00 00 00 80 0d 1c 00 00 0e 1c 00   ................
lz_base:
0x00100000: 90 02 58 80 00 80 ef be ad de cc cc cc cc cc cc   ..X.............
0x00100010: cc cc cc cc cc cc cc cc cc cc cc cc cc cc cc cc   ................
0x00100020: cc cc cc cc cc cc cc cc cc cc cc cc cc cc cc cc   ................
0x00100030: cc cc cc cc cc cc cc cc cc cc cc cc cc cc cc cc   ................
0x00100040: cc cc cc cc cc cc cc cc cc cc cc cc cc cc cc cc   ................
0x00100050: cc cc cc cc cc cc cc cc cc cc cc cc cc cc cc cc   ................
0x00100060: cc cc cc cc cc cc cc cc cc cc cc cc cc cc cc cc   ................
0x00100070: cc cc cc cc cc cc cc cc cc cc cc cc cc cc cc cc   ................
0x00100080: cc cc cc cc cc cc cc cc cc cc cc cc cc cc cc cc   ................
0x00100090: cc cc cc cc cc cc cc cc cc cc cc cc cc cc cc cc   ................
0x001000a0: f2 78 71 c6 77 66 b0 f2 d1 76 81 41 da 26 10 00   .xq.wf...v.A.&..
0x001000b0: 78 1d 7b a3 e5 53 97 52 cb 4e 13 61 ba 30 10 00   x.{..S.R.N.a.0..
0x001000c0: 00 00 00 00 38 00 00 00 ac 3b 10 00 0d 33 10 00   ....8....;...3..
0x001000d0: 40 00 00 00 80 36 10 00 e0 00 10 00 40 00 00 00   @....6......@...
0x001000e0: 65 ce fe de 0d 48 a1 2c e0 3b 10 00 e0 3b 10 00   e....H.,.;...;..
0x001000f0: 6e 01 10 00 21 3c 10 00 21 3c 10 00 5e 2a 10 00   n...!<..!<..^*..
bootloader_data:
0x00108058: 0f 04 5a 00 21 24 0b 00 da e5 42 df ec 6e 2e 3b   ..Z.!$....B..n.;
0x00108068: fa 74 4f 97 ba 7d c5 d6 03 18 1d cd 8b 68 e1 a5   .tO..}.......h..
0x00108078: 5c e0 37 e7 d7 e7 02 ee 21 18 04 00 f3 18 60 07   \.7.....!.....`.
0x00108088: 4e 6a 9f 9e 6a 76 c4 a7 d9 49 12 da 20 4a 32 73   Nj..jv...I.. J2s
0x00108098: 11 0e 18 00 11 00 00 00 80 cf 00 00 00 00 20 0a   .............. .
0x001080a8: 00 40 e8 cf 00 10 00 00 00 02 80 01 40 00 80 01   .@..........@...
TPM event log:
0xcfe84000: 00 00 00 00 03 00 00 00 00 00 00 00 00 00 00 00   ................
0xcfe84010: 00 00 00 00 00 00 00 00 00 00 00 00 39 00 00 00   ............9...
0xcfe84020: 53 70 65 63 20 49 44 20 45 76 65 6e 74 30 33 00   Spec ID Event03.
0xcfe84030: 00 00 00 00 00 02 00 02 02 00 00 00 04 00 14 00   ................
0xcfe84040: 0b 00 20 00 14 00 40 e8 cf 00 00 00 00 00 10 00   .. ...@.........
0xcfe84050: 00 00 00 00 00 63 02 00 00 11 00 00 00 02 05 00   .....c..........
0xcfe84060: 00 02 00 00 00 04 00 f3 18 60 07 4e 6a 9f 9e 6a   .........`.Nj..j
0xcfe84070: 76 c4 a7 d9 49 12 da 20 4a 32 73 0b 00 da e5 42   v...I.. J2s....B
0xcfe84080: df ec 6e 2e 3b fa 74 4f 97 ba 7d c5 d6 03 18 1d   ..n.;.tO..}.....
0xcfe84090: cd 8b 68 e1 a5 5c e0 37 e7 d7 e7 02 ee 06 00 00   ..h..\.7........
0xcfe840a0: 00 53 4b 49 4e 49 54 12 00 00 00 02 05 00 00 02   .SKINIT.........
0xcfe840b0: 00 00 00 04 00 d0 8d a8 6f 97 cc 6b 6b 99 9b 1c   ........o..kk...
0xcfe840c0: e1 e7 0b 60 3c db a3 2d ac 0b 00 56 2b de 09 b5   ...`<..-...V+...
0xcfe840d0: 8b 8a 9b d5 40 1c 73 52 0e 37 74 58 3e 44 0c 9b   ....@.sR.7tX>D..
0xcfe840e0: a1 9e aa 35 a5 b5 be ba e3 b3 d2 23 00 00 00 4d   ...5.......#...M
0xcfe840f0: 65 61 73 75 72 65 64 20 62 6f 6f 74 6c 6f 61 64   easured bootload
0xcfe84100: 65 72 20 64 61 74 61 20 69 6e 74 6f 20 50 43 52   er data into PCR
0xcfe84110: 31 38 12 00 00 00 02 05 00 00 02 00 00 00 04 00   18..............
0xcfe84120: 0c d1 e3 3c e3 f1 21 44 13 87 3d 88 b7 1e c5 6b   ...<..!D..=....k
0xcfe84130: 35 fc 40 b0 0b 00 45 5b 59 f4 f8 8d 08 95 a6 c9   5.@...E[Y.......
0xcfe84140: dc 3e 2c 28 c3 c7 76 dc 0a 4f 55 89 54 a5 a9 bb   .>,(..v..OU.T...
0xcfe84150: 71 bf 02 97 da 8b 17 00 00 00 4d 65 61 73 75 72   q.........Measur
0xcfe84160: 65 64 20 4d 42 49 20 69 6e 74 6f 20 50 43 52 31   ed MBI into PCR1
0xcfe84170: 38 11 00 00 00 02 05 00 00 02 00 00 00 04 00 8f   8...............
0xcfe84180: d0 9f b3 cf 33 c1 e0 e0 47 8c cf d0 8b 2c c9 b8   ....3...G....,..
0xcfe84190: b6 1a a4 0b 00 bc f4 aa 0d 2e 10 72 12 9d 42 ef   ...........r..B.
0xcfe841a0: bf 57 25 ad 56 de 40 86 45 16 11 75 3d 7e 3d 7e   .W%.V.@.E..u=~=~
0xcfe841b0: 4e 78 ba ae 61 1a 00 00 00 4d 65 61 73 75 72 65   Nx..a....Measure
0xcfe841c0: 64 20 4b 65 72 6e 65 6c 20 69 6e 74 6f 20 50 43   d Kernel into PC
0xcfe841d0: 52 31 37 11 00 00 00 02 05 00 00 02 00 00 00 04   R17.............
0xcfe841e0: 00 fb 15 05 fd bc b7 09 e3 62 11 8e 13 06 7e c0   .........b....~.
0xcfe841f0: 0b 53 45 4b d1 0b 00 78 57 2d b4 2d 41 95 0a f8   .SEK...xW-.-A...
0xcfe84200: 2e 8c f1 5d 1c f5 55 7c 6b 03 ac 90 c6 27 25 10   ...]..U|k....'%.
0xcfe84210: d0 23 1d 79 8d 25 17 48 00 00 00 72 6f 6f 74 3d   .#.y.%.H...root=
0xcfe84220: 2f 64 65 76 2f 6d 6d 63 62 6c 6b 30 70 32 20 63   /dev/mmcblk0p2 c
0xcfe84230: 6f 6e 73 6f 6c 65 3d 74 74 79 53 30 2c 31 31 35   onsole=ttyS0,115
0xcfe84240: 32 30 30 20 65 61 72 6c 79 70 72 69 6e 74 6b 3d   200 earlyprintk=
0xcfe84250: 73 65 72 69 61 6c 2c 74 74 79 53 30 2c 31 31 35   serial,ttyS0,115
0xcfe84260: 32 30 30 00 00 00 00 00 00 00 00 00 00 00 00 00   200.............
0xcfe84270: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   ................
...
lz_main() is about to exit
 Xen 4.16.1
(XEN) Xen version 4.16.1 (akowalski@) (gcc (Ubuntu 9.4.0-1ubuntu1~20.04.1) 9.4.0) debug=n Fri May  6 12:41:07 CEST 2022
(XEN) Latest ChangeSet: Tue Apr 12 14:21:23 2022 +0200 git:f265444922
(XEN) build-id: a5224ddd598819a4ba78754a6e0bd7c15f9983e1
(XEN) Bootloader: GRUB 2.04~rc1
(XEN) Command line: loglvl=all guest_loglvl=all com1=115200,8n1 console=com1
(XEN) Xen image load base address: 0xcf600000
(XEN) Video information:
(XEN)  No VGA detected
(XEN) Disc information:
(XEN)  Found 1 MBR signatures
(XEN)  Found 1 EDD information structures
(XEN) CPU Vendor: AMD, Family 22 (0x16), Model 48 (0x30), Stepping 1 (raw 00730f01)
(XEN) Xen-e820 RAM map:
(XEN)  [0000000000000000, 000000000009fbff] (usable)
(XEN)  [000000000009fc00, 000000000009ffff] (reserved)
(XEN)  [00000000000f0000, 00000000000fffff] (reserved)
(XEN)  [0000000000100000, 00000000cfe81fff] (usable)
(XEN)  [00000000cfe82000, 00000000cfffffff] (reserved)
(XEN)  [00000000f8000000, 00000000fbffffff] (reserved)
(XEN)  [00000000fed40000, 00000000fed44fff] (reserved)
(XEN)  [0000000100000000, 000000012effffff] (usable)
(XEN)  [000000012f000000, 000000012fffffff] (reserved)
(XEN) ACPI: RSDP 000F3830, 0024 (r2 COREv4)
(XEN) ACPI: XSDT CFE950E0, 0074 (r1 COREv4 COREBOOT        0 CORE 20200925)
(XEN) ACPI: FACP CFE96E10, 0114 (r6 COREv4 COREBOOT        0 CORE 20200925)
(XEN) ACPI: DSDT CFE95280, 1B87 (r2 COREv4 COREBOOT    10001 INTL 20200925)
(XEN) ACPI: FACS CFE95240, 0040
(XEN) ACPI: SSDT CFE96F30, 01EF (r2 COREv4 COREBOOT       2A CORE 20200925)
(XEN) ACPI: MCFG CFE97120, 003C (r1 COREv4 COREBOOT        0 CORE 20200925)
(XEN) ACPI: TPM2 CFE97160, 004C (r4 COREv4 COREBOOT        0 CORE 20200925)
(XEN) ACPI: APIC CFE971B0, 007E (r3 COREv4 COREBOOT        0 CORE 20200925)
(XEN) ACPI: HEST CFE97230, 01D0 (r1 COREv4 COREBOOT        0 CORE 20200925)
(XEN) ACPI: SSDT CFE97400, 48A6 (r2 AMD    AGESA           2 MSFT  4000000)
(XEN) ACPI: SSDT CFE9BCB0, 07C8 (r1 AMD    AGESA           1 AMD         1)
(XEN) ACPI: DRTM CFE9C480, 007C (r1 COREv4 COREBOOT        0 CORE 20200925)
(XEN) ACPI: HPET CFE9C500, 0038 (r1 COREv4 COREBOOT        0 CORE 20200925)
(XEN) System RAM: 4078MB (4176004kB)
(XEN) No NUMA configuration found
(XEN) Faking a node at 0000000000000000-000000012f000000
(XEN) Domain heap initialised
(XEN) DMI present.
(XEN) Using APIC driver default
(XEN) ACPI: PM-Timer IO Port: 0x818 (32 bits)
(XEN) ACPI: v5 SLEEP INFO: control[0:0], status[0:0]
(XEN) ACPI: SLEEP INFO: pm1x_cnt[1:804,1:0], pm1x_evt[1:800,1:0]
(XEN) ACPI: 32/64X FACS address mismatch in FADT - cfe95240/0000000000000000, using 32
(XEN) ACPI:             wakeup_vec[cfe9524c], vec_size[20]
(XEN) ACPI: Local APIC address 0xfee00000
(XEN) ACPI: IOAPIC (id[0x04] address[0xfec00000] gsi_base[0])
(XEN) IOAPIC[0]: apic_id 4, version 33, address 0xfec00000, GSI 0-23
(XEN) ACPI: IOAPIC (id[0x05] address[0xfec20000] gsi_base[24])
(XEN) IOAPIC[1]: apic_id 5, version 33, address 0xfec20000, GSI 24-55
(XEN) ACPI: INT_SRC_OVR (bus 0 bus_irq 0 global_irq 2 dfl dfl)
(XEN) ACPI: INT_SRC_OVR (bus 0 bus_irq 9 global_irq 9 low level)
(XEN) ACPI: IRQ0 used by override.
(XEN) ACPI: IRQ2 used by override.
(XEN) ACPI: IRQ9 used by override.
(XEN) Enabling APIC mode:  Flat.  Using 2 I/O APICs
(XEN) ACPI: HPET id: 0x10228201 base: 0xfed00000
(XEN) PCI: MCFG configuration 0: base f8000000 segment 0000 buses 00 - 3f
(XEN) PCI: MCFG area at f8000000 reserved in E820
(XEN) PCI: Using MCFG for segment 0000 bus 00-3f
(XEN) HEST: Table parsing has been initialized
(XEN) Using ACPI (MADT) for SMP configuration information
(XEN) SMP: Allowing 4 CPUs (0 hotplug CPUs)
(XEN) IRQ limits: 56 GSI, 776 MSI/MSI-X
(XEN) CPU0: 1000 (600 ... 1400) MHz
(XEN) xstate: size: 0x340 and states: 0x7
(XEN) CPU0: AMD Fam16h machine check reporting enabled
(XEN) Speculative mitigation facilities:
(XEN)   Hardware hints:
(XEN)   Hardware features:
(XEN)   Compiled-in support: INDIRECT_THUNK SHADOW_PAGING
(XEN)   Xen settings: BTI-Thunk RETPOLINE, SPEC_CTRL: No, Other: BRANCH_HARDEN
(XEN)   Support for HVM VMs: RSB
(XEN)   Support for PV VMs: RSB
(XEN)   XPTI (64-bit PV only): Dom0 disabled, DomU disabled (without PCID)
(XEN)   PV L1TF shadowing: Dom0 disabled, DomU disabled
(XEN) Using scheduler: SMP Credit Scheduler rev2 (credit2)
(XEN) Initializing Credit2 scheduler
(XEN)  load_precision_shift: 18
(XEN)  load_window_shift: 30
(XEN)  underload_balance_tolerance: 0
(XEN)  overload_balance_tolerance: -3
(XEN)  runqueues arrangement: socket
(XEN)  cap enforcement granularity: 10ms
(XEN) load tracking window length 1073741824 ns
(XEN) Platform timer is 14.318MHz HPET
(XEN) Detected 998.136 MHz processor.
(XEN) Freed 1024kB unused BSS memory
(XEN) alt table ffff82d0404525b0 -> ffff82d04045eaf2
(XEN) I/O virtualisation disabled
(XEN) nr_sockets: 1
(XEN) ENABLING IO-APIC IRQs
(XEN)  -> Using new ACK method
(XEN) ..TIMER: vector=0xF0 apic1=0 pin1=2 apic2=0 pin2=0
(XEN) Allocated console ring of 32 KiB.
(XEN) mwait-idle: does not run on family 22 model 48
(XEN) HVM: ASIDs enabled.
(XEN) SVM: Supported advanced features:
(XEN)  - Nested Page Tables (NPT)
(XEN)  - Last Branch Record (LBR) Virtualisation
(XEN)  - Next-RIP Saved on #VMEXIT
(XEN)  - DecodeAssists
(XEN)  - Pause-Intercept Filter
(XEN)  - Pause-Intercept Filter Threshold
(XEN)  - TSC Rate MSR
(XEN) HVM: SVM enabled
(XEN) HVM: Hardware Assisted Paging (HAP) detected
(XEN) HVM: HAP page sizes: 4kB, 2MB, 1GB
(XEN) alt table ffff82d0404525b0 -> ffff82d04045eaf2
(XEN) Brought up 4 CPUs
(XEN) Scheduling granularity: cpu, 1 CPU per sched-resource
(XEN) Adding cpu 0 to runqueue 0
(XEN)  First cpu on runqueue, activating
(XEN) Adding cpu 1 to runqueue 0
(XEN) Adding cpu 2 to runqueue 0
(XEN) Adding cpu 3 to runqueue 0
(XEN) MCA: Use hw thresholding to adjust polling frequency
(XEN) mcheck_poll: Machine check polling timer started.
(XEN) NX (Execute Disable) protection active
(XEN) Dom0 has maximum 680 PIRQs
(XEN) *** Building a PV Dom0 ***
(XEN)  Xen  kernel: 64-bit, lsb
(XEN)  Dom0 kernel: 64-bit, PAE, lsb, paddr 0x1000000 -> 0x282c000
(XEN) PHYSICAL MEMORY ARRANGEMENT:
(XEN)  Dom0 alloc.:   0000000124000000->0000000128000000 (983076 pages to be allocated)
(XEN) VIRTUAL MEMORY ARRANGEMENT:
(XEN)  Loaded kernel: ffffffff81000000->ffffffff8282c000
(XEN)  Phys-Mach map: 0000008000000000->00000080007a0120
(XEN)  Start info:    ffffffff8282c000->ffffffff8282c4b8
(XEN)  Page tables:   ffffffff8282d000->ffffffff82846000
(XEN)  Boot stack:    ffffffff82846000->ffffffff82847000
(XEN)  TOTAL:         ffffffff80000000->ffffffff82c00000
(XEN)  ENTRY ADDRESS: ffffffff825d8180
(XEN) Dom0 has maximum 4 VCPUs
(XEN) Initial low memory virq threshold set at 0x4000 pages.
(XEN) Scrubbing Free RAM in background
(XEN) Std. Loglevel: All
(XEN) Guest Loglevel: All
(XEN) *** Serial input to DOM0 (type 'CTRL-a' three times to switch input)
(XEN) Freed 632kB init memory
mapping kernel into physical memory
about to get started...
[    0.000000] Linux version 5.8.10-yocto-standard (oe-user@oe-host) (x86_64-fobnail-linux-gcc (GCC) 11.2.0, GNU ld (GNU Binutils) 2.37.2021071
[    0.000000] Command line: root=/dev/mmcblk0p2 console=ttyS0,115200 earlyprintk=serial,ttyS0,115200
[    0.000000] x86/fpu: Supporting XSAVE feature 0x001: 'x87 floating point registers'
[    0.000000] x86/fpu: Supporting XSAVE feature 0x002: 'SSE registers'
[    0.000000] x86/fpu: Supporting XSAVE feature 0x004: 'AVX registers'
[    0.000000] x86/fpu: xstate_offset[2]:  576, xstate_sizes[2]:  256
[    0.000000] x86/fpu: Enabled xstate features 0x7, context size is 832 bytes, using 'standard' format.
[    0.000000] Released 0 page(s)
[    0.000000] BIOS-provided physical RAM map:
[    0.000000] Xen: [mem 0x0000000000000000-0x000000000009efff] usable
[    0.000000] Xen: [mem 0x000000000009fc00-0x00000000000fffff] reserved
[    0.000000] Xen: [mem 0x0000000000100000-0x00000000cfe81fff] usable
[    0.000000] Xen: [mem 0x00000000cfe82000-0x00000000cfffffff] reserved
[    0.000000] Xen: [mem 0x00000000f8000000-0x00000000fbffffff] reserved
[    0.000000] Xen: [mem 0x00000000fec00000-0x00000000fec00fff] reserved
[    0.000000] Xen: [mem 0x00000000fec20000-0x00000000fec20fff] reserved
[    0.000000] Xen: [mem 0x00000000fed40000-0x00000000fed44fff] reserved
[    0.000000] Xen: [mem 0x00000000fee00000-0x00000000feefffff] reserved
[    0.000000] Xen: [mem 0x0000000100000000-0x000000012effffff] usable
[    0.000000] Xen: [mem 0x000000012f000000-0x000000012fffffff] reserved
[    0.000000] NX (Execute Disable) protection: active
...
```

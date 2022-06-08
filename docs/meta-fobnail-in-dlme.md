# Running OS in DLME

In this document, we would like to describe the attempts to run a minimal
operating system in the Dynamically Launched Measured Environment. For this
purpose, we have prepared a meta-fobnail layer, which allows you to build a
system image using the Yocto Project. Following this document will allow
building an operating system running in DLME independently.

## Prerequisites

- PC Engines apu2 with a serial connection to host PC
- Host PC that meets the requirements outlined in the Yocto Project
  [documentation](https://docs.yoctoproject.org/brief-yoctoprojectqs/index.html#compatible-linux-distribution)
  (tested on Ubuntu 20.04)
- At least 2GB USB stick to flash the image
- At least 2GB SD card and SD card reader
- `bmaptools` installed; in case of Ubuntu it can be done via the following
  command

```
sudo apt install bmap-tools
```

## Background

To prepare a system that will be capable to run in DLME we want to use
[Trenchboot](https://trenchboot.org/) project. It is a framework that allows
building security engines to perform launch integrity actions for their systems.
The framework builds upon Boot Integrity Technologies (BITs) that establish one
or more Roots of Trust (RoT) from which a degree of confidence that integrity
actions were not subverted.

Because we want to test on PC Engines apu2 we needed that our system consists
of:

* [GRUB SecureLaunch for
  AMD](https://github.com/TrenchBoot/grub/tree/trenchboot_support_2.04)
* [Linux 5.13 SecureLaunch for Intel and
  AMD](https://github.com/TrenchBoot/linux/tree/linux-sl-5.13-amd)
* [Secure Kernel Loader](https://github.com/TrenchBoot/secure-kernel-loader)

The above information was obtained from the website describing the [Trenchboot
repositories](https://trenchboot.org/code/).

It is also crucial that the platform runs at least coreboot `v4.12.0.3`. We
used PC Engines apu2, for which firmware can be found on
[pcengines.github.io](https://pcengines.github.io/).

The last thing is to make sure that IOMMU is enabled. For PC Engines apu2 it can
be checked in setup chosen from the boot menu.

## Building

As mentioned earlier, we prepared a project using Yocto. For layers management
and simplification of the build process, we use
[kas](https://kas.readthedocs.io/en/latest/) tool with
[kas-container](https://github.com/siemens/kas/blob/master/kas-container)
script. All build components are open and public, so everyone can try to
reproduce our effort if only the prerequisites are met.

### Instructions

To complete the build process, please execute the following steps.

1. Download `kas-container` script, place it in PATH

```
$ wget https://raw.githubusercontent.com/siemens/kas/2.6.2/kas-container ~/bin/kas-container
```

2. Make it executable.

```
$ chmod +x ~/bin/kas-container
```

3. Make sure that script can be executed from any place.

```
$ kas-container --help
Usage: /home/tomzy/bin/kas-container [OPTIONS] { build | checkout | shell } [KASOPTIONS] [KASFILE]
       /home/tomzy/bin/kas-container [OPTIONS] for-all-repos [KASOPTIONS] [KASFILE] COMMAND
       /home/tomzy/bin/kas-container [OPTIONS] clean
       /home/tomzy/bin/kas-container [OPTIONS] menu [KCONFIG]

Positional arguments:
build                   Check out repositories and build target.
checkout                Check out repositories but do not build.
shell                   Run a shell in the build environment.
for-all-repos           Run specified command in each repository.
clean                   Clean build artifacts, keep downloads.
menu                    Provide configuration menu and trigger configured build.

Optional arguments:
--isar                  Use kas-isar container to build Isar image.
--with-loop-dev         Pass a loop device to the container. Only required if
                        loop-mounting is used by recipes.
--runtime-args          Additional arguments to pass to the container runtime
                        for running the build.
--docker-args           Same as --runtime-args (deprecated).
-d                      Print debug output.
-v                      Same as -d (deprecated).
--version               print program version.
--ssh-dir               Directory containing SSH configurations.
                        Avoid $HOME/.ssh unless you fully trust the container.
--aws-dir               Directory containing AWScli configuration.
--git-credential-store  File path to the git credential store
--no-proxy-from-env     Do not inherit proxy settings from environment.
--repo-ro               Mount current repository read-only
                        (default for build command)
--repo-rw               Mount current repository writeable
                        (default for shell command)

You can force the use of podman over docker using KAS_CONTAINER_ENGINE=podman.
```

4. Clone `meta-fobnail` repository.

```
$ git clone git@github.com:fobnail/meta-fobnail.git
```

> Note: use `git clone https://github.com/fobnail/meta-fobnail.git` if do not
  have SSH keys in GitHub account.

5. Start build.

```
$ kas-container build meta-fobnail/kas-debug.yml
```

> Note: `kas-debug.yml` configuration will generate a debug version of the
  image, which differs from prod in that it does not have a root password

Running this command will clone every needed Yocto layer and start the build
which last about 2 hours for the first run.

6. Flash image.

Firstly, you need to connect your USB stick to your PC and unmount the device.

```
$ sudo fdisk -l

Device     Boot   Start      End Sectors Size Id Type
/dev/sdb1  *       2048    34815   32768  16M  c W95 FAT32 (LBA)
/dev/sdb2         34816  2131967 2097152   1G 83 Linux
/dev/sdb3       2131968  4229119 2097152   1G 83 Linux
/dev/sdb4       4229120 12617727 8388608   4G 83 Linux

$ sudo umount /dev/sdX*  # in this case /dev/sdb*
```

Now you can flash your image.

```
$ cd build/tmp/deploy/images/fobnail-machine
$ sudo bmaptool copy --bmap fobnail-base-image-debug-fobnail-machine.wic.bmap \
  fobnail-base-image-debug-fobnail-machine.wic.gz /dev/sdX # in this case /dev/sdb
```

## Tests

We checked a couple of configurations to run our minimal OS in DLME. It is
essential to build GRUB, Linux kernel, and Secure Kernel Loader in such a form
that they can work together, which will allow us to achieve our goal. For most
cases, we want to load the following configuration with GRUB.

```
menuentry 'secure-boot'{
  slaunch skinit
  slaunch_module (hd0,msdos1)/skl.bin
  linux /bzImage root=/dev/sda2 console=ttyS0,115200 earlyprintk=serial,ttyS0,115200
}
```

### Test of latest Trenchboot

In the beginning, we prepared a build containing the revisions mentioned on the
Trenchboot website. They are listed in the [background](#background) section.
Such an image can be built by following the steps described in the
[instructions](#instructions) section. The only difference is that the
meta-fobnail repository should be checked out to `tb-latest-bad-format` branch.

Unfortunately, the boot failed and reset while the platform was still in GRUB.
The result was as follows.

```
grub_cmd_slaunch:122: check for manufacturer
grub_cmd_slaunch:126: check for cpuid
grub_cmd_slaunch:136: set slaunch
grub_cmd_slaunch_module:156: check argc
grub_cmd_slaunch_module:161: check relocator
grub_cmd_slaunch_module:170: open file
grub_cmd_slaunch_module:175: get size
grub_cmd_slaunch_module:180: allocate memory
grub_cmd_slaunch_module:192: addr: 0x100000
grub_cmd_slaunch_module:194: target: 0x100000
grub_cmd_slaunch_module:196: add module
grub_cmd_slaunch_module::205: read file
grub_cmd_slaunch_module:215: close file
grub_slaunch_boot_skinit:41: real_mode_target: 0x8b000
grub_slaunch_boot_skinit:42: prot_mode_target: 0xx1000000
grub_slaunch_boot_skinit:43: params: 0xcfe037cBad bootloader data format
Rebooting now..
```

#### 3mdeb GRUB fork

Problems early in booting meant we needed to make some modifications to GRUB. To
test the next build, repeat the steps in the [instructions](#instructions) using
the branch `tb-grub-3mdeb`.

Compared to the previous configuration, we used the [3mdeb fork for
GRUB](https://github.com/3mdeb/grub/tree/tpm_event_log_support) here. The Secure
Kernel Loader on master already uses passing info via MB2 tags. TrenchBoot/GRUB
lacks it.

We were able to push the boot further, but not much further though. The result
was as follows. SKL falls into an endless loop (the observed `Flushing IOMMU
cache` log will be printed indefinitely).

```
shasum calculated:
0x001001dc: e5 7d 2d 07 37 2b 7e 38 ec f7 d4 52 72 c9 6d 63   .}-.7+~8...Rr.mc
0x001001ec: 9e 52 83 76 cc cc cc cc cc cc cc cc cc cc cc cc   .R.v............
shasum calculated:
0x001001f0: 7f dd 92 0a 8c 39 1b 4c 76 ee 1c 31 43 a8 55 9e   .....9.Lv..1C.U.
0x00100200: 6a 6a 41 8f d4 10 17 87 27 fd ea b9 dc d6 57 6a   jjA.....'.....Wj
PCR extended
IOMMU MMIO Base Address = 0xf7f00000:
0x00000000: IOMMU_MMIO_STATUS_REGISTER
0x00106001: IOMMU_MMIO_DEVICE_TABLE_BA
0x00103000: IOMMU_MMIO_COMMAND_BUF_BA
0x00105000: IOMMU_MMIO_EVENT_LOG_BA
0x00000018: IOMMU_MMIO_STATUS_REGISTER
INVALIDATE_IOMMU_ALL
0x00290ad2: IOMMU_MMIO_EXTENDED_FEATURE
0x0000000a: IOMMU_MMIO_STATUS_REGISTER
0x0000000a: IOMMU_MMIO_STATUS_REGISTER
Disabling SLB protection
IOMMU MMIO Base Address = 0xf7f00000:
0x0000000a: IOMMU_MMIO_STATUS_REGISTER
0x00106001: IOMMU_MMIO_DEVICE_TABLE_BA
0x00103000: IOMMU_MMIO_COMMAND_BUF_BA
0x00105000: IOMMU_MMIO_EVENT_LOG_BA
0x0000001a: IOMMU_MMIO_STATUS_REGISTER
INVALIDATE_IOMMU_ALL
0x00290ad2: IOMMU_MMIO_EXTENDED_FEATURE
0x0000000a: IOMMU_MMIO_STATUS_REGISTER
0x0000000a: IOMMU_MMIO_STATUS_REGISTER
Flushing IOMMU cache..........................................................
```

#### W/A patch for SKL

We can solve the above problem with IOMMU in one of two ways:

* turn off the IOMMU in PC Engines apu2 (which we do not want to do),
* add a small workaround for the Secure Kernel Loader.

We decided to implement the second option and add the following change into the
SKL source code.

```
Subject: [PATCH] main.c: IOMMU flushing infinity loop workaround

---
 main.c | 7 ++++---
 1 file changed, 4 insertions(+), 3 deletions(-)

diff --git a/main.c b/main.c
index d2c727a..d125fa6 100644
--- a/main.c
+++ b/main.c
@@ -341,9 +341,10 @@ static void iommu_setup(void)

                iommu_load_device_table(iommu_cap, &iommu_done);
                print("Flushing IOMMU cache");
-               while (!iommu_done) {
-                       print(".");
-               }
+               /* Inifnity loop workaround */
+               //while (!iommu_done) {
+                       //print(".");
+               //}
                print("\nIOMMU set\n");
        }

--
2.25.1
```

It helped us to go even further. To test this changes, repeat the steps in the
[instructions](#instructions) using the branch `tb-grub-3mdeb-iommu`. The start
was promising.

```
shasum calculated:
0x001001dc: 44 b6 98 d6 93 b8 5b 21 9b 87 0a 43 a9 bf 27 af   D.....[!...C..'.
0x001001ec: 8f 58 b7 35 cc cc cc cc cc cc cc cc cc cc cc cc   .X.5............
shasum calculated:
0x001001f0: 4a 9d bd 0c 02 b0 fb 79 c5 79 8a 53 2a 67 eb f7   J......y.y.S*g..
0x00100200: 91 8b 3d 87 6f cb db 03 08 c3 cb 27 f1 92 c4 f4   ..=.o......'....
PCR extended
IOMMU MMIO Base Address = 0xf7f00000:
0x00000000: IOMMU_MMIO_STATUS_REGISTER
0x00106001: IOMMU_MMIO_DEVICE_TABLE_BA
0x00103000: IOMMU_MMIO_COMMAND_BUF_BA
0x00105000: IOMMU_MMIO_EVENT_LOG_BA
0x00000018: IOMMU_MMIO_STATUS_REGISTER
INVALIDATE_IOMMU_ALL
0x00290ad2: IOMMU_MMIO_EXTENDED_FEATURE
0x0000000a: IOMMU_MMIO_STATUS_REGISTER
0x0000000a: IOMMU_MMIO_STATUS_REGISTER
Disabling SLB protection
IOMMU MMIO Base Address = 0xf7f00000:
0x0000000a: IOMMU_MMIO_STATUS_REGISTER
0x00106001: IOMMU_MMIO_DEVICE_TABLE_BA
0x00103000: IOMMU_MMIO_COMMAND_BUF_BA
0x00105000: IOMMU_MMIO_EVENT_LOG_BA
0x0000001a: IOMMU_MMIO_STATUS_REGISTER
INVALIDATE_IOMMU_ALL
0x00290ad2: IOMMU_MMIO_EXTENDED_FEATURE
0x0000000a: IOMMU_MMIO_STATUS_REGISTER
0x0000000a: IOMMU_MMIO_STATUS_REGISTER
Flushing IOMMU cache
IOMMU set

code32_start 0x01000000:
mle_header
0x01000514: 5a ac 82 90 6f 47 a7 74 0f 5c 55 a2 cb 51 b6 42   Z...oG.t.\U..Q.B
0x01000524: 34 00 00 00 02 00 02 00 a0 9c 89 00 00 00 00 00   4...............
0x01000534: 00 00 00 00 34 3f 8a 00 27 02 00 00 00 00 00 00   ....4?..'.......
0x01000544: 00 00 00 00 1f 8b 08 00 00 00 00 00 02 03 ec dd   ................
shasum calculated:
0x001001dc: 2e aa bb 1b 57 52 6c cf 6a 44 4c 3b ed 31 49 19   ....WRl.jDL;.1I.
0x001001ec: c5 78 58 ba 34 3a 20 00 93 12 10 00 00 00 00 00   .xX.4: .........
shasum calculated:
0x001001f0: a6 cd 2c aa c9 34 d6 cf 7b 44 de 87 43 04 33 54   ..,..4..{D..C.3T
0x00100200: 27 70 1c d7 80 97 7a 5e f7 7a 42 50 93 8b 7f 3f   'p....z^.zBP...?
PCR extended
```

But unfortunately, we failed to boot, and the process ended with `slaunch: Error
failed to find TPM event log` error.

```
[    3.998124] slaunch: Error failed to find TPM event log
[    3.998124]  - error: 0xc0008022
[    3.998209] invalid opcode: 0000 [#1] PREEMPT SMP NOPTI
[    3.999030] CPU: 1 PID: 1 Comm: swapper/0 Not tainted 5.13.0-yocto-standard #1
[    3.999030] Hardware name: PC Engines apu2/apu2, BIOS v4.12.0.3 07/30/2020
[    3.999030] RIP: 0010:slaunch_skinit_reset+0x1b/0x1d
[    3.999030] Code: c7 c8 f6 df af e8 10 74 00 00 e9 89 14 4e ff 0f 1f 44 00 00 55 48 89 f2 48 89 fe 48 c7 c7 74 f8 df af 48 89 e5 e8 f0 73 00 00 <0f> 0b 83 c8 03 48 c7 c7 e0 f7 df af 89 05 e9 de c9 00 e8 d9 73 00
[    3.999030] RSP: 0018:ffffb7ed40023e50 EFLAGS: 00010246
[    3.999030] RAX: 0000000000000040 RBX: 0000000000000000 RCX: 0000000000000000
[    3.999030] RDX: 0000000000000000 RSI: 00000000ffffffea RDI: 00000000ffffffff
[    3.999030] RBP: ffffb7ed40023e50 R08: ffffffffb00c17e8 R09: 0000000000000003
[    3.999030] R10: ffffffffb0051800 R11: ffffffffb0051800 R12: ffffffffb0270f67
[    3.999030] R13: ffff9bc980160b40 R14: ffffffffb03b1384 R15: 0000000000000000
[    3.999030] FS:  0000000000000000(0000) GS:ffff9bc9aac80000(0000) knlGS:0000000000000000
[    3.999030] CS:  0010 DS: 0000 ES: 0000 CR0: 0000000080050033
[    3.999030] CR2: 0000000000000000 CR3: 000000005d80a000 CR4: 00000000000406e0
[    3.999030] Call Trace:
[    3.999030]  slaunch_module_init+0x40d/0x501
[    3.999030]  ? slaunch_setup_txt+0x4f4/0x4f4
[    3.999030]  do_one_initcall+0x51/0x220
[    3.999030]  kernel_init_freeable+0x1f2/0x241
[    3.999030]  ? rest_init+0xc3/0xc3
[    3.999030]  kernel_init+0xe/0x10d
[    3.999030]  ret_from_fork+0x22/0x30
[    3.999030] Modules linked in:
[    4.144594] ---[ end trace 6340ff4ba3dd3afb ]---
[    4.149263] RIP: 0010:slaunch_skinit_reset+0x1b/0x1d
[    4.154348] Code: c7 c8 f6 df af e8 10 74 00 00 e9 89 14 4e ff 0f 1f 44 00 00 55 48 89 f2 48 89 fe 48 c7 c7 74 f8 df af 48 89 e5 e8 f0 73 00 00 <0f> 0b 83 c8 03 48 c7 c7 e0 f7 df af 89 05 e9 de c9 00 e8 d9 73 00
[    4.173123] RSP: 0018:ffffb7ed40023e50 EFLAGS: 00010246
[    4.178364] RAX: 0000000000000040 RBX: 0000000000000000 RCX: 0000000000000000
[    4.185507] RDX: 0000000000000000 RSI: 00000000ffffffea RDI: 00000000ffffffff
[    4.192697] RBP: ffffb7ed40023e50 R08: ffffffffb00c17e8 R09: 0000000000000003
[    4.199841] R10: ffffffffb0051800 R11: ffffffffb0051800 R12: ffffffffb0270f67
[    4.206992] R13: ffff9bc980160b40 R14: ffffffffb03b1384 R15: 0000000000000000
[    4.214142] FS:  0000000000000000(0000) GS:ffff9bc9aac80000(0000) knlGS:0000000000000000
[    4.222261] CS:  0010 DS: 0000 ES: 0000 CR0: 0000000080050033
[    4.228021] CR2: 0000000000000000 CR3: 000000005d80a000 CR4: 00000000000406e0
[    4.235214] Kernel panic - not syncing: Attempted to kill init! exitcode=0x0000000b
[    4.236170] Kernel Offset: 0x2dc00000 from 0xffffffff81000000 (relocation range: 0xffffffff80000000-0xffffffffbfffffff)
[    4.236170] ---[ end Kernel panic - not syncing: Attempted to kill init! exitcode=0x0000000b ]---
```

Full boot log can be found in
[meta-fobnail-log.cap](https://github.com/fobnail/docs/blob/main/dev-notes/meta-fobnail-log.cap).

### Working solution

Unfortunately, we had to make a few concessions to get our system running in
DLME. Instead of SKL, we used a landing zone, we had to replace the kernel with
an older version (v5.8), and we had to turn off IOMMU in the platform options.
Ultimately, we used the following components for the build.

* [GRUB](https://github.com/3mdeb/grub/commit/4dfd376a2aac1045f467d7e0d70f37b3a6d82eeb):
  3mdeb fork and branch `tpm_event_log_support`
* [kernel](https://github.com/TrenchBoot/linux/commit/9f24db90772857682193e6d73fbee7c2d05ce7dd):
  version v5.8, branch `linux-sl-5.8amd`
* [landing-zone](https://github.com/TrenchBoot/landing-zone/commit/60bba229ae5dd12f29d205e02197313139d8ae3f):
  instead of Secure Kernel Loader from TrenchBoot

> Update: Since v0.2.2 we managed to use latest versions of GRUB and kernel as
  well as the Secure Kernel Loader instead of landing-zone. We neede the
  following revisions/versions

  * [GRUB](https://github.com/TrenchBoot/grub/pull/4)
  * [kernel](https://github.com/TrenchBoot/linux/commit/7fe9bb33721975fc796e4114b7370bed9afefffe)
  * [SKL](https://github.com/TrenchBoot/secure-kernel-loader/commit/3432f4398652727f402b710c2fea4e3f1efecce6)
    with reverted changes from [IOMMU: Shrink the size of the command
    buffer](https://github.com/TrenchBoot/secure-kernel-loader/commit/266dfcfa6fcfae2a184d0ff840b447308e8e82d0)

In addition, we also had to give up trying to boot our system from a USB stick
and switch to booting from a SD card. Unfortunately, after the LZ application,
the system was unable to detect the USB device while booting. Eventually we
loaded the following configuration into GRUB.

```
menuentry 'secure-boot'{
  slaunch skinit
  slaunch_module (hd0,msdos1)/lz_header.bin
  linux /bzImage root=/dev/mmcblk0p2 console=ttyS0,115200 earlyprintk=serial,ttyS0,115200
}
```

> Update: Since v0.2.2 we use here skl.bin file instead of lz_header.bin.

Using the above, we can boot into a system for which we assume it runs in DLME.
In the later stages of the project, we will add to the system appropriate tools
to check the content of PCR17 and PCR18, which will allow us to confirm this
fact. To test this build, repeat the steps in the [instructions](#instructions).
The changes are merged to `main` branch.

> Update: Since v0.1.1 after login to shell, we can run `tpm2_pcrread` to verify
  PCR17 and PCR18 values. If these registers are not empty (or equal to
  0xFFFF...), we can conclude that the device is running in DLME. The output of
  PCR registers dump should look similar to this:

```
# tpm2_pcrread
sha1:
  0 : 0x3A3F780F11A4B49969FCAA80CD6E3957C33B2275
  1 : 0xE3B26DA646504EF1E716C2D2B6DE920EA37F919D
  2 : 0x53DE584DCEF03F6A7DAC1A240A835893896F218D
  3 : 0x3A3F780F11A4B49969FCAA80CD6E3957C33B2275
  4 : 0x29E503140459F9AE7B6FB502AB0486D941DE948C
  5 : 0x6A0AEEB558ECFBCB56A13C2E525D511412C1B558
  6 : 0x3A3F780F11A4B49969FCAA80CD6E3957C33B2275
  7 : 0x3A3F780F11A4B49969FCAA80CD6E3957C33B2275
  8 : 0x0000000000000000000000000000000000000000
  9 : 0x0000000000000000000000000000000000000000
  10: 0x0000000000000000000000000000000000000000
  11: 0x0000000000000000000000000000000000000000
  12: 0x0000000000000000000000000000000000000000
  13: 0x0000000000000000000000000000000000000000
  14: 0x0000000000000000000000000000000000000000
  15: 0x0000000000000000000000000000000000000000
  16: 0x0000000000000000000000000000000000000000
  17: 0x79827B00FE723A9CDADC1A88AC564F73762752D9
  18: 0x6095E1C7DF27260720C3C73585C8B2C61765E7FD
  19: 0x0000000000000000000000000000000000000000
  20: 0x0000000000000000000000000000000000000000
  21: 0x0000000000000000000000000000000000000000
  22: 0x0000000000000000000000000000000000000000
  23: 0x0000000000000000000000000000000000000000
sha256:
  0 : 0xD27CC12614B5F4FF85ED109495E320FB1E5495EB28D507E952D51091E7AE2A72
  1 : 0xCAC0A9141FA63BCB0027B4C219E9DC19CE164EBB003297D3F2F1AF1877B561D8
  2 : 0xFA8791BB6BCE8EBF4AD7B516ADFBBB9B2F1499A8876E2C909135AEBDCCA2D84C
  3 : 0xD27CC12614B5F4FF85ED109495E320FB1E5495EB28D507E952D51091E7AE2A72
  4 : 0xA7862EE52AA91BFFFBA8E04B7D2EA8660ABFAA610F8B831A2701F03DF9E33493
  5 : 0x0DE1E8677D23636EFBDADEA31E697A862386309383800A8DA695D1E1D4F8D60B
  6 : 0xD27CC12614B5F4FF85ED109495E320FB1E5495EB28D507E952D51091E7AE2A72
  7 : 0xD27CC12614B5F4FF85ED109495E320FB1E5495EB28D507E952D51091E7AE2A72
  8 : 0x0000000000000000000000000000000000000000000000000000000000000000
  9 : 0x0000000000000000000000000000000000000000000000000000000000000000
  10: 0x0000000000000000000000000000000000000000000000000000000000000000
  11: 0x0000000000000000000000000000000000000000000000000000000000000000
  12: 0x0000000000000000000000000000000000000000000000000000000000000000
  13: 0x0000000000000000000000000000000000000000000000000000000000000000
  14: 0x0000000000000000000000000000000000000000000000000000000000000000
  15: 0x0000000000000000000000000000000000000000000000000000000000000000
  16: 0x0000000000000000000000000000000000000000000000000000000000000000
  17: 0xADB1DB1E9EE753FF66EF9AB06BC2192FDAB24C4239F06AA37B5B8D8ECE0A2294
  18: 0xD73B0CA75F4323BC7C7A7397580336096974468114C25CA4CA89E3991CC4634D
  19: 0x0000000000000000000000000000000000000000000000000000000000000000
  20: 0x0000000000000000000000000000000000000000000000000000000000000000
  21: 0x0000000000000000000000000000000000000000000000000000000000000000
  22: 0x0000000000000000000000000000000000000000000000000000000000000000
  23: 0x0000000000000000000000000000000000000000000000000000000000000000
```

## Summary

* When we started working on this issue, we assumed that we would be able to run
  a minimal OS in DLME by integrating the latest revisions of `Trenchboot`
  components in it; unfortunately, we encountered some problems that may be
  possible to solve together.
* At the moment, we have agreed to a satisfactory solution; the later stages of
  the project will allow us to confirm that the OS is running in DLME.
* To start a discussion of the problems we encountered when trying to integrate
  `Trenchboot` into PC Engines apu2, we created an
  [issue](https://github.com/TrenchBoot/trenchboot-issues/issues/6)

### Update

* As proved in previous section, we managed to run latest version of SKL, and
  solve the
  [issue](https://github.com/TrenchBoot/trenchboot-issues/issues/6#issuecomment-1148230965)

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
- `bmaptools` installed; in case of Ubuntu it can be done via the following
  command

```
sudo apt install bmap-tools
```

## Background

In order to prepare a system that will be capable to running in DLME we want to
use [Trenchboot](https://trenchboot.org/) project. It is a framework that allows
to build security engines to perform launch integrity actions for their systems.
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

It is also important that the platform run at least coreboot `v4.12.0.3`. We
used PC Engines apu2 for which firmware can be found on
[pcengines.github.io](https://pcengines.github.io/).

Last thing is to make sure that IOMMU is enabled. For PC Engines apu2 it can be
checked in setup choosed from boot menu.

## Building

As mentioned earlier, we prepared a project using Yocto. For layers management
and simplification of the build process, we use
[kas](https://kas.readthedocs.io/en/latest/) tool with
[kas-container](https://github.com/siemens/kas/blob/master/kas-container)
script. All build components are open and public, so everyone can try to
reproduce our effort if only the prerequisites are met.

### Instructions

To complete the build process please execute the following steps.

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

Firstly, you need to connect your USB stick to your PC and unmount device.

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

We checked couple configurations in order to run our minimal OS in DLME. It is
important to build GRUB, Linux kernel and Secure Kernel Loader in such a form
that they can work together, which will allow us to achieve our goal. For most
cases we want to load the following configuration with GRUB.

```
menuentry 'secure-boot'{
  slaunch skinit
  slaunch_module (hd0,msdos1)/skl.bin
  linux /bzImage root=/dev/sda2 console=ttyS0,115200 earlyprintk=serial,ttyS0,115200
}
```

### Test of latest Trenchboot

At the beginning, we prepared a build containing the revisions mentioned on the
Trenchboot website. They are listed in the [background](#background) section.
Such an image can be built by following the steps described in the
[instructions](#instructions) section, The only difference is that the
meta-fobnail repository should be checked out to `tb-latest-bad-format` branch.

Unfortunately, the boot failed and reset while the platfrom was still in GRUB.
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

Problems so early in booting meant we needed to make some modifications to GRUB.
To test the next build, repeat the steps in the [instructions](#instructions) -
this time using the branch `tb-grub-3mdeb`.

Compared to the previous configuration, we used the [3mdeb fork for
GRUB](https://github.com/3mdeb/grub/tree/tpm_event_log_support) here. The Secure
Kernel Loader on master already uses passing info via MB2 tags. TrenchBoot/GRUB
lacks it.

We was able to push the boot further but not much further though. The result was
as follows. We can see that SKL falls into an endless loop (the observed
`Flushing IOMMU cache` log will be printed indefinitely).

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

Link to TB issue

### Working solution

linux v5.8
landing-zone
3mdeb fork for GRUB

## Summary

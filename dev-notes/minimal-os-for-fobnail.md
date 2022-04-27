# Minimal OS for Fobnail project

The Fobnail Projects aims to build a USB device capable of verifying
trustworthiness of the platform it is connected to. The project is described
more in-depth [here](../docs/index.md).

According to the information presented at TrenchBoot Summit 2021
[Fobnail: Attestation in Your Pocket](https://youtu.be/xZoCtNV8Qs0?t=5062),
the minimum OS, together with the connected Fobnail Token, should allow
determining the state of the platform before the target OS is launched. As we
can see in the presentation, [heads](https://github.com/osresearch/heads) would
be responsible for the platform's trust control, while in this document, we want
to present the results of research on the possibilities of using various
operating systems.

## OS requirements

We pay attention to the drivers supported by each OS. We need the following
drivers:

- TPM drivers
- Drivers required for communicating with Fobnail Token
  - USB host drivers - at minimum EHCI and xHCI support, however UHCI and OHCI
    would be good to have
  - USB EEM driver - emulated Ethernet driver
  - Network stack with IPv4 support

Also we pay attention to OS security, ability to run in DLME, including
portability beetween different hardware (a single binary should be able to boot
on all x86 platforms with support for ACPI) and ability to chainload another,
target OS.

The table below summarises OS features which are a hard requirement, though they
could be implemented by us.

| Requirement             | Description                                                        |
| ----------------------- | ------------------------------------------------------------------ |
| USB host driver         | Required for communication with Fobnail                            |
| USB EEM driver          | Required for communication with Fobnail                            |
| Network Stack           | Required for communication with Fobnail                            |
| TPM driver              | Required to perform attestation                                    |
| Bootloader Capabilities | Required to boot target OS                                         |
| C library   | [fobnail-attester](https://github.com/fobnail/fobnail-attester) is writen in C |
| Bootable by SKL         | Whether OS can be loaded by TrenchBoot SKL without SKL or OS modification |

These are another features which are taken into account (soft requirements).

| Feature                  | Description                                                               |
| ------------------------ | ------------------------------------------------------------------------- |
| Microkernel              | Microkernels are more secure and are preferred                            |
| OS portability           | Required to avoid rebuilding minimal OS for each device                   |
| CPU Architecture support | OS supported architectures, mostly we consider x86, ARM, RISC-V and POWER |

### OS score counting

The table at the bottom of this document contains summary of features of all
OSes together with theirs scores. Each score is computed using the following
rules

- If feature is present then +1
- If feature is not present then 0, or if feature would be hard to implement
  then -1
- If this is a hard requirement then multiple result by 2

## Different OSs propositions

The research effect is presented below. 4 systems were considered:

* Zephyr
* Xous
* seL4
* Linux

Each of them has a short description, an analysis of the launch in DLME, and the
possibilities and potential problems that will have to be addressed for the
Fobnail Token to be functional.

### Zephyr

Zephyr is a well-known and supported embedded RTOS with a monolithic kernel
written in C. It supports various devices ranging from Cortex-M to x86 CPUs.
It's main benefits are IPv4, CoAP, and USB (including USB EEM) support which are
required for communication with Fobnail token. Other benefits are listed
[here](https://www.zephyrproject.org/benefits/).

#### Running in DLME

Secure launch uses a component called
[SKL](https://github.com/TrenchBoot/secure-kernel-loader) (Secure Kernel Loader)
which is responsible for booting OS in DLME (Zephyr in that case). Zephyr boots
using Multiboot1 but SKL does not support it (only Multiboot2 is supported). For
SKL to be able to boot Zephyr, either Zephyr would have to support Multiboot2 or
SKL Multiboot1.

Also, Zephyr may have trouble with running on different hardware configurations.
Some important HW-related configuration is baked into Zephyr during build:

- LAPIC base is selected by `CONFIG_LOAPIC_BASE_ADDRESS`, if target platform has
  LAPIC remapped using `IA32_APIC_BASE` Zephyr will break.

- APIC mode (xAPIC vs x2APIC) is selected when building Zephyr and cannot be
  detected at runtime. Zephyr may not boot if it has different APIC mode set
  than firmware has.

#### Fobnail integration

Zephyr provides most of the drivers needed for integration:

- USB hub drivers
- USB EEM driver
- Network stack with IPv4 and CoAP support

TPM driver is missing, however there is a fairly recent
[PoC implementation](https://github.com/drandreas/zephyr-tpm2-poc) of TPM2
stack.

Zephyr provides good support for standard C library (newlib), which should
simplify [fobnail-attester](https://github.com/fobnail/fobnail-attester)
porting.

### Xous

Xous is a microkernel-based OS written in Rust, currently in Alpha state. Xous
is used as a secure microkernel for [Betrusted](betrusted.io) device and is
focused on RISC-V, which right now is the only architecture supported. However,
x86 already contains
[dummy implementation](https://github.com/betrusted-io/xous-core/blob/30b82b25b100e958790973c129dc49e1acca79ec/kernel/src/arch/x86_64.rs)
which probably will be extended someday.


#### Running in DLME

Xous currently cannot run on x86, and DLME on RISC-V is not covered here. We
have opened an [issue](https://github.com/betrusted-io/xous-core/issues/139)
raising the subject, among others, about x86 support.

#### Fobnail integration

Xous has network support but currently lacks USB host drivers and TPM support.
Also, it lacks C interface for calling the kernel and lacks C library. This
should not be a serious problem because needed support may be added by writing
the userland code in Rust (which is preffered) or C bindings to the Rust code.
It means that getting
[fobnail-attester](https://github.com/fobnail/fobnail-attester) running may
require a some amount of work. Also, Xous needs to gain kexec-like
abilities to chainload target OS.

### seL4

seL4 is a secure L4 family microkernel written in C. It has strong security
guarantees assured by [formal proofs](https://sel4.systems/Info/FAQ/proof.pml).
However these proofs are still incomplete for x86; see
[Supported Platforms](https://docs.sel4.systems/Hardware) for an up-to-date
verification status. Due to its microkernel nature, seL4 provides stronger
isolation. A breach in one of the components (such as a USB driver or the
network stack) wouldn't compromise the entire OS (contrary to monolithic
kernels).

We have opened an [issue](https://github.com/seL4/seL4/issues/832) about seL4
usage as secure bootloader.

#### Running in DLME

It should be runnable, however due to microkernel nature all programs run in
unprivileged mode which complicates booting of the target OS. seL4 would have to
gain kexec-like abilities.

#### Fobnail integration

seL4 provides virtually no drivers, except a few drivers listed here.
According to the page linked above, `libusbdrivers` is inactive and lacks XHCI
support. seL4 has basic support for an old version of musl C library (v1.1.16).

Using seL4 would require a significant amount of work:
- Extending USB drivers
- Implementing USB EEM driver
- Implementing TPM driver
- Porting [fobnail-attester](https://github.com/fobnail/fobnail-attester)
- seL4 runs all processes in unprivileged mode, so the kernel itself would have
  to be modified to allow booting of another kernel

[Genode](https://github.com/genodelabs/genode) is an framework for creating
custom, specialized OSes. It provides USB drivers (including `usb-net` driver
from which should handle USB EEM) and a network stack. Its support for seL4 used
to be incomplete, and many components were broken. However, that might have
changed, and Genode may be an easier way to get software running on seL4. We
have opened an [issue](https://github.com/genodelabs/genode/issues/4480) here
too.

### Linux

In the case of Linux, a minimal distribution will be prepared that meets the
requirements of the project. [Yocto Project](https://www.yoctoproject.org/) will
be used for this, because it gives a lot of freedom in manipulating the elements
that make up the target system.

#### Running in DLME

Ensuring that the operating system works in DLME can be achieved by using the
[TrenchBoot](https://trenchboot.org/) project. For this purpose, a Yocto
[meta-fobnail](https://github.com/fobnail/meta-fobnail) layer has been created
that integrates the necessary elements. The effects of the tests are presented
in a [separate document](./running-os-in-dlme.md).

#### Fobnail integration

At this point, the [fobnail-attester](https://github.com/fobnail/fobnail-attester)
application is strongly dependent on Linux. Mainly due to the fact that its
development took place on this operating system. Therefore, it is important that
the created minimal OS also has an integrated attester application. The main
dependencies of its operation are drivers for USB, USB EEM and TPM, but running
them under Linux will not cause major problems.

## PoC test

Running DLME requires GRUB from TrenchBoot as mainline doesn't have DLME
support. Currently, there is an ongoing discussion how TrenchBoot should be
integrated into Linux. Unless this is solved no investment in TrenchBoot GRUB2
implementation would be made. During PoC we use GRUB from
[here](https://github.com/3mdeb/grub/tree/tb_xen). To reproduce PoC results
please follow the instructions below.

- Clone GRUB source

  ```shell
  $ git clone https://github.com/3mdeb/grub/ --branch tb_xen
  ```

- Prepare container for building GRUB.

  ```shell
  $ cd grub
  $ docker run --rm -it -v $(readlink -f ..):$(readlink -f ..) -w $PWD ubuntu:18.04
  (docker)$ apt update
  (docker)$ apt install gcc-5 automake autoconf make git gettext autopoint \
      pkg-config python bison flex
  ```

- Build GRUB (execute from container while in GRUB directory)

  ```shell
  (docker)$ ./bootstrap
  (docker)$ mkdir ../grub-install build
  (docker)$ cd build
  (docker)$ env CC=gcc-5 CFLAGS=-Wno-error=vla ../configure --prefix=$(readlink -f ../../grub-install)
  (docker)$ make -j$(nproc)
  (docker)$ make install
  ```

- Exit container and prepare boot disk by creating MBR + FAT32 partition.

  ```shell
  $ sudo fdisk /dev/disk/by-id/usb-TS-RDF5_SD_Transcend_000000000039-0\:0 <<EOF
  o
  n
  p
  1


  w
  EOF

  $ sudo mkfs.vfat -F32 /dev/disk/by-id/usb-TS-RDF5_SD_Transcend_000000000039-0\:0-part1
  ```

- Install GRUB onto target device. This will a complete GRUB into target
  devices, including MBR, GRUB, and its modules.

  ```shell
  $ sudo mkdir /mnt/boot
  $ sudo mount /dev/disk/by-id/usb-TS-RDF5_SD_Transcend_000000000039-0\:0-part1 /mnt/boot
  $ cd grub-install
  $ sudo sbin/grub-install --target=i386-pc --modules "part_msdos fat" \
      --boot-directory=/mnt/boot \
      /dev/disk/by-id/usb-TS-RDF5_SD_Transcend_000000000039-0\:0
  ```

- Build SKL (Secure Kernel Loader) from TrenchBoot and copy it to boot
  partition.

  ```shell
  $ git clone https://github.com/TrenchBoot/secure-kernel-loader
  $ cd secure-kernel-loader
  $ make
  $ cp skl.bin /mnt/boot/
  ```

Setup Zephyr build environment. Instructions below are based on Zephyr
[Getting Started Guide](https://docs.zephyrproject.org/latest/develop/getting_started/index.html).

- Install required packages. Ubuntu has too old packages so we have to use
  KitWare repo.

  ```shell
  $ wget https://apt.kitware.com/kitware-archive.sh
  $ sudo bash kitware-archive.sh
  $ sudo apt install --no-install-recommends git cmake ninja-build gperf \
    ccache dfu-util device-tree-compiler wget \
    python3-dev python3-pip python3-setuptools python3-tk python3-wheel xz-utils file \
    make gcc gcc-multilib g++-multilib libsdl2-dev python3-venv
  $ pip3 install --user -U west
  ```

- Obtain Zephyr toolchain

  ```shell
  $ wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.14.1/zephyr-sdk-0.14.1_linux-x86_64.tar.gz
  $ tar xf zephyr-sdk-0.14.1_linux-x86_64.tar.gz -C ~/.local
  $ rm zephyr-sdk-0.14.1_linux-x86_64.tar.gz
  $ ~/.local/zephyr-sdk-0.14.1/setup.sh -c
  ```

- Fetch Zephyr source code.

  ```shell
  $ west init zephyr
  $ cd zephyr
  $ west update
  ```

- Install required packages, use virtual environment to avoid conflict with
  user/system packages.

  ```shell
  $ python3 -m venv .env
  $ pip3 install -r zephyr/scripts/requirements.txt
  ```

- Build Zephyr, we use Qemu as the target, but it works on APU (without DLME).

  ```shell
  $ cd zephyr
  $ west build -p auto -b qemu_x86_64 samples/userspace/hello_world_user/
  $ sudo cp build/zephyr/zephyr.elf /mnt/boot/
  ```

- Configure GRUB, create '/mnt/boot/grub/grub.cfg` with the following contents.

  ```shell
  default 0
  menuentry "Zephyr (DLME)" {
    slaunch skinit
    slaunch_module /skl.bin
    multiboot /zephyr.elf
  }
  menuentry "Zephyr (no DLME)" {
    multiboot /zephyr.elf
  }
  ```

- Umount disk and sync before unplugging it. There were some problems with
  obtaining working binary, so before booting Zephyr on target platform please
  check whether it boots in Qemu (without DLME). When booting Zephyr you should
  see `Hello World` message on serial console, if Zephyr doesn't output anything
  or if it stuck at `Booting Zephyr` message then something went wrong.

  ```shell
  $ qemu-system-x86_64 -nographic -serial mon:stdio -m 512M -cpu qemu64 \
    -drive format=raw,file=/dev/disk/by-id/usb-TS-RDF5_SD_Transcend_000000000039-0\:0
  ```

- Zephyr should normally boot on APU. If DLME variant (from boot menu) is used
  Zephyr will also boot normally. That is because TrenchBoot GRUB executes SKL
  only when using
  [Multiboot2](https://github.com/3mdeb/grub/blob/tb_xen/grub-core/loader/multiboot.c#L192)
  but Zephyr has support only for Multiboot1.

- Booting Zephyr in DLME would require updating `slaunch` support in GRUB and
  bringing Multiboot1 support to SKL.

## Summary

* The above report outlines four operating systems that should be considered
  candidates for Fobnail Token interoperability.

* The basic choice is Linux, the operating system based on it is created with
  the use of Yocto Project. All achievements can be reproduced at any time using
  the [meta-fobnail](https://github.com/fobnail/meta-fobnail) layer.

* As part of the report, the seL4, Xous and Zephyr systems were also checked.
  The possibility of running it in DLME and the integration of the
  [fobnail-attester](https://github.com/fobnail/fobnail-attester) application
  was checked for each of them.

* The description of an attempt to run Zephyr on PC Engines apu2 in order to
  verify the current state of the system to work with Fobnail Token has been
  included. The effects are described in the [PoC test](#poc-test) section.

* The table below summarises all OSes features described
  [above](#os-requirements).

| OS      | USB host driver  | USB EEM driver   | Network stack | TPM driver        | OS portability | Bootloader capabilities | C library | Microkernel | CPU Architecture support | Bootable by SKL | Score |
| ------- | ---------------- | ---------------- | ------------- | ----------------- | -------------- | ----------------------- | --------- | ----------- | ------------------------ | --------------- | ----- |
| Zephyr  | Yes (+2)         | Yes (+2)         | Yes (+2)      | PoC available (0) | Limited (-1)   | No (0)                  | Yes (+2)  | No (0)      | Good (+1) [^4]           | No (0) [^6]     | 8     |
| Xous    | No (0)           | No (0)           | Yes (+2)      | No (0)            | Limited (-1)   | No (0)                  | No (0)    | Yes (+1)    | RISC-V only (-1)         | No (0)          | 1     |
| seL4    | No (0) [^1]      | No (0) [^2]      | Yes (+2)      | No (0)            | Limited (-1)   | No (0)                  | Yes (+2)  | Yes (+1)    | Good (+1) [^3]           | Yes (+2)        | 7     |
| Linux   | Yes (+2)         | Yes (+2)         | Yes (+2)      | Yes (+2)          | Yes (+1)       | Yes (kexec) (+2)        | Yes (+2)  | No (0)      | Good (+1) [^5]           | Yes (+2)        | 16    |

[^1]: seL4 has an old unmaintained driver with no xHCI support. Better driver is
      available only from Genode.

[^2]: available from Genode only

[^3]: supports x86, ARM and RISC-V

[^4]: supports x86, ARM, RISC-V, SPARC and MIPS

[^5]: supports most of the architectures, much more than any other OS listed
      here

[^6]: not bootable due to of lack Multiboot1 support in SKL

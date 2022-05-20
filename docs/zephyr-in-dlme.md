# Running Zephyr in DLME

In addition to conducting the theoretical [analysis](./meta-fobnail-in-dlme.md),
it was decided to include a test report of one of the operating systems in order
to verify the information presented. The choice fell on Zephyr.

Running DLME requires GRUB from TrenchBoot as mainline doesn't have DLME
support. Currently, there is an ongoing discussion how TrenchBoot should be
integrated into Linux - see the following links:

* [Discussion on adding Trenchboot secure dynamic launch Linux kernel
  support](https://groups.google.com/g/trenchboot-devel/c/-RxTtan5H3I/m/vU-yp5kHAgAJ).
* [Discussion on adding Secure Launch SMP bringup
  support](https://groups.google.com/g/trenchboot-devel/c/6ilwQUkwt9w/m/dpyriBCHAgAJ).

Unless this is solved no investment in TrenchBoot GRUB2 implementation would be
made. During PoC we use GRUB from
[here](https://github.com/3mdeb/grub/tree/tb_xen). To reproduce PoC results
please follow the instructions below.

* Clone GRUB source.

  ```shell
  $ git clone https://github.com/3mdeb/grub/ --branch tb_xen
  ```

* Prepare container for building GRUB.

  ```shell
  $ cd grub
  $ docker run --rm -it -v $(readlink -f ..):$(readlink -f ..) -w $PWD ubuntu:18.04
  (docker)$ apt update
  (docker)$ apt install gcc-5 automake autoconf make git gettext autopoint \
      pkg-config python bison flex
  ```

* Build GRUB (execute from container while in GRUB directory).

  ```shell
  (docker)$ ./bootstrap
  (docker)$ mkdir ../grub-install build
  (docker)$ cd build
  (docker)$ env CC=gcc-5 CFLAGS=-Wno-error=vla ../configure --prefix=$(readlink -f ../../grub-install)
  (docker)$ make -j$(nproc)
  (docker)$ make install
  ```

* Exit container and prepare boot disk by creating MBR + FAT32 partition

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

* Install GRUB onto target device, this will put a complete GRUB into target
  device, including MBR, GRUB, and its modules.

  ```shell
  $ sudo mkdir /mnt/boot
  $ sudo mount /dev/disk/by-id/usb-TS-RDF5_SD_Transcend_000000000039-0\:0-part1 /mnt/boot
  $ cd grub-install
  $ sudo sbin/grub-install --target=i386-pc --modules "part_msdos fat" \
      --boot-directory=/mnt/boot \
      /dev/disk/by-id/usb-TS-RDF5_SD_Transcend_000000000039-0\:0
  ```

* Build SKL (Secure Kernel Loader) from TrenchBoot and copy it to boot
  partition.

  ```shell
  $ git clone https://github.com/TrenchBoot/secure-kernel-loader
  $ cd secure-kernel-loader
  $ make
  $ cp skl.bin /mnt/boot/
  ```

Setup Zephyr build environment. Instructions below are based on Zephyr
[Getting Started Guide](https://docs.zephyrproject.org/latest/develop/getting_started/index.html).

* Install required packages, Ubuntu has too old packages so we have to use
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

* Obtain Zephyr toolchain.

  ```shell
  $ wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.14.1/zephyr-sdk-0.14.1_linux-x86_64.tar.gz
  $ tar xf zephyr-sdk-0.14.1_linux-x86_64.tar.gz -C ~/.local
  $ rm zephyr-sdk-0.14.1_linux-x86_64.tar.gz
  $ ~/.local/zephyr-sdk-0.14.1/setup.sh -c
  ```

* Fetch Zephyr source code.

  ```shell
  $ west init zephyr
  $ cd zephyr
  $ west update
  ```

* Install required packages, use virtual environment to avoid conflict with
  user/system packages.

  ```shell
  $ python3 -m venv .env
  $ pip3 install -r zephyr/scripts/requirements.txt
  ```

* Build Zephyr, we use QEMU as the target, but it works on APU (without DLME).

  ```shell
  $ cd zephyr
  $ west build -p auto -b qemu_x86_64 samples/userspace/hello_world_user/
  $ sudo cp build/zephyr/zephyr.elf /mnt/boot/
  ```

* Configure GRUB, create '/mnt/boot/grub/grub.cfg` with the following contents.

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

* Umount disk and sync before unplugging it.
  > There were some problems with obtaining working binary, so before booting
    Zephyr on target platform please check whether it boots in QEMU (without
    DLME). When booting Zephyr you should see `Hello World` message on serial
    console, if Zephyr doesn't output anything or if it stuck at
    `Booting Zephyr` message then something went wrong.

  ```shell
  $ qemu-system-x86_64 -nographic -serial mon:stdio -m 512M -cpu qemu64 \
    -drive format=raw,file=/dev/disk/by-id/usb-TS-RDF5_SD_Transcend_000000000039-0\:0
  ```

* Zephyr should normally boot on APU, if DLME variant (from boot menu) is used
  Zephyr will also boot normally - that is because TrenchBoot GRUB executes SKL
  only when using
  [Multiboot2](https://github.com/3mdeb/grub/blob/tb_xen/grub-core/loader/multiboot.c#L192)
  but Zephyr has support only for Multiboot1.

* Booting Zephyr in DLME would require extending it with Multiboot2 support.

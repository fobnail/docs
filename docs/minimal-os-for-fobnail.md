# Minimal OS for Fobnail Project

The Fobnail Project aims to build a USB device capable of verifying
trustworthiness of the platform it is connected to. The project is described
more in-depth [here](./index.md).

According to the information presented at TrenchBoot Summit 2021
[Fobnail: Attestation in Your Pocket](https://youtu.be/xZoCtNV8Qs0?t=5062),
the minimum OS, together with the connected Fobnail Token, should allow
determining the state of the platform before the target OS is launched. As we
can see in the presentation, [heads](https://github.com/osresearch/heads) (here
presented as a coreboot payload) could be responsible for the platform's trust
control. But in this document, we want to present the results of research on the
possibilities of using various operating systems running as DLME that will
perform the same function.

Also it should be possible to launch minimal OS from an already running target
OS, this is called late launch as opposed to early launch described above.
Minimal OS would communicate with Fobnail Token, perform attestation in order
to determine platform state, and then hand the control back to the main OS.
Currently, only early launch will be supported, late launch is planned as a
future improvement.

## OS requirements

We pay attention to the drivers supported by each OS. We need the following
drivers:

* TPM drivers
* Drivers required for communicating with Fobnail Token:
  - USB host drivers - at minimum EHCI and xHCI support, however UHCI and OHCI
    would be good to have
  - USB EEM driver - emulated Ethernet driver
  - network stack with IPv4 support

Also we pay attention to OS security, ability to run in DLME, including
portability between different hardware (a single binary should be able to boot
on all x86 platforms with support for ACPI) and ability to chainload another,
target OS.

The table below summarises OS features which are a hard requirement, though they
could be implemented by us.

| Requirement             | Description                                                                    |
| ----------------------- | ------------------------------------------------------------------------------ |
| USB host driver         | Required for communication with Fobnail Token                                  |
| USB EEM driver          | Required for communication with Fobnail Token                                  |
| Network Stack           | Required for communication with Fobnail Token                                  |
| TPM driver              | Required to perform attestation                                                |
| Bootloader Capabilities | Required to boot target OS                                                     |
| C library               | [Fobnail Attester](https://github.com/fobnail/fobnail-attester) is writen in C |
| Bootable by SKL         | Whether OS can be loaded by TrenchBoot SKL without SKL or OS modification      |
| License                 | Whether given OS is open sourced can be fully used on embedded devices         |

These are another features which are taken into account (soft requirements).

| Feature                  | Description                                                               |
| ------------------------ | ------------------------------------------------------------------------- |
| Microkernel              | Microkernels are more secure and are preferred                            |
| OS portability           | Required to avoid rebuilding minimal OS for each device                   |
| CPU Architecture support | OS supported architectures, mostly we consider x86, ARM, RISC-V and POWER |

## Different OSs propositions

The research effect is presented below. 8 systems were considered

* [Zephyr](#zephyr)
* [Xous](#xous)
* [seL4](#sel4) (with and without CAmkES)
* [Genode](#genode)
* [Linux](#linux)
* [Little Kernel / Trusted Little Kernel](#little-kernel--trusted-little-kernel)
* [Fuchsia](#fuchsia-zircon)
* [Xen](#xen)

Each of them has a short description, an analysis of the launch in DLME, and the
possibilities and potential problems that will have to be addressed for the
Fobnail Token to be functional.

### Zephyr

Zephyr is a well-known and supported embedded RTOS with a monolithic kernel
written in C. It supports various devices ranging from Cortex-M to x86 CPUs.
It's main benefits are IPv4, CoAP, and USB (including USB EEM) support which are
required for communication with Fobnail Token. Other benefits are listed
[here](https://www.zephyrproject.org/benefits/).

#### Running in DLME

Secure launch uses a component called
[SKL](https://github.com/TrenchBoot/secure-kernel-loader) (Secure Kernel Loader)
which is responsible for booting OS in DLME (Zephyr in that case). Zephyr boots
using Multiboot1 but SKL does not support it (only Multiboot2 is supported). For
SKL to be able to boot Zephyr, Zephyr must gain Multiboot2 support.

Multiboot1 is hard to measure due to its tags being scattered around in memory.
To properly measure Multiboot1, SKL would have to know and parse every tag as
most of them contain pointers to other structures. This is not going to happen.

Also, Zephyr may have trouble with running on different hardware configurations.
Some important HW-related configuration is baked into Zephyr during build:

* LAPIC base is selected by `CONFIG_LOAPIC_BASE_ADDRESS`, if target platform has
  LAPIC remapped using `IA32_APIC_BASE` Zephyr will break.

* APIC mode (xAPIC vs x2APIC) is selected when building Zephyr and cannot be
  detected at runtime. Zephyr may not boot if it has different APIC mode set
  than firmware has.

As part of the PoC tests, the possibility of using Zephyr as a minimum OS
running in DLME was checked. The results are described
[here](./zephyr-in-dlme.md).

#### Fobnail integration

Zephyr provides most of the drivers needed for integration:

* USB hub drivers,
* USB EEM driver,
* Network stack with IPv4 and CoAP support.

TPM driver is missing, there is a fairly recent
[PoC implementation](https://github.com/drandreas/zephyr-tpm2-poc) of TPM2
stack, but it is intended for embedded devices and supports SPI only.

Zephyr provides good support for standard C library (newlib), which should
simplify [Fobnail Attester](https://github.com/fobnail/fobnail-attester)
porting.

#### License

Basically runs under GPLv2 License but more information can be found
[here](https://docs.zephyrproject.org/latest/LICENSING.html).

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
should not be a serious problem and may be solved by rewriting
[Fobnail Attester](https://github.com/fobnail/fobnail-attester) in Rust or by
adding required C bindings to Rust code. It means that getting
[Fobnail Attester](https://github.com/fobnail/fobnail-attester) running may
require some amount of work. Also, Xous needs to gain kexec-like abilities to
chainload target OS.

#### License

The Xous microkernel runs under
[Apache 2.0](https://github.com/betrusted-io/xous-core/blob/main/LICENSES/Apache-2.0.txt)
license.

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
gain kexec-like abilities. seL4 developers are
[willing](https://github.com/seL4/seL4/issues/832#issuecomment-1114385689) to
add kexec support.

#### Fobnail integration

seL4 provides virtually no drivers, except a few drivers listed
[here](https://docs.sel4.systems/projects/available-user-components.html#device-drivers).
According to the page linked above, `libusbdrivers` is inactive and lacks XHCI
support. seL4 has basic support for an old version of musl C library (v1.1.16).

Using seL4 would require a significant amount of work:

* Extending USB drivers,
* Implementing USB EEM driver,
* Implementing TPM driver,
* Porting [Fobnail Attester](https://github.com/fobnail/fobnail-attester),
* seL4 runs all processes in unprivileged mode, so the kernel itself would have
  to be modified to allow booting of another kernel.

[Genode](https://github.com/genodelabs/genode) can provide many of the required
drivers. See the section below for details.

[CAmkES](https://docs.sel4.systems/projects/camkes/) provides tools for writing
seL4-native components, it also has VMM support for running virtual machines.
Contrary to Genode CAmkES does not provide drivers, but VMs may provide them.
However this requires virtualization and seL4 currently supports it only on
Intel CPUs. See the section below for more details.

#### License

The [documentation](https://sel4.systems/Info/license.pml) shows that the seL4
kernel itself and most of its proof is licensed under GPL Version 2.

### Genode

[Genode](https://github.com/genodelabs/genode) is a framework for creating
custom, specialized OSes. It can work on various kernels/hypervisors, including
seL4 and Linux. It provides many drivers, including USB drivers (including
`usb-net` driver which should handle USB EEM) and a network stack. When running
on Linux all components are sandboxed with seccomp, however it should be noted
that Linux drivers are used. For other kernels Genode provides sandboxed
drivers.

#### Running in DLME

Whether Genode can run in DLME depends on the choosen kernel, however we are
interested mainly in seL4. Please see seL4 [section](#running-in-dlme-2) above
for details.

#### Fobnail integration

TPM driver would have to be ported to Genode. Underlying kernel must have kexec
abilities and Genode must have component capable of invoking kexec. USB and
EEM drivers should work out-of-the-box.

Genode's seL4 support is considered to be in healthly state (see
[this](https://github.com/genodelabs/genode/issues/4480#issuecomment-1108525737)
comment). But it should be noted Genode with seL4 has not been used outside the
lab so far.

#### License

Genode license (AGPLv3) is very restrictive and it could be a problem for the
Fobnail Project.

### seL4 + CAmkES

CAmkES provides tools and libraries for quickly writing seL4-native components.
Contrary to `seL4_libs` CAmkES is planned to be formally verified (verification
is in progress).

#### Running in DLME

Since CAmkES is based on seL4, the same restrictions apply. See the
[section](#running-in-dlme-2) above for details.

#### Fobnail integration

CAmkES also does not provide many drivers. However it does provide VMM for
virtual machines. Features:

* Support for booting Linux kernel.
* Support only for 32-bit VMs, according to
  [camkes-vm](https://docs.sel4.systems/projects/camkes-vm/) documentation.
  > The first step is to install Ubuntu natively on the cma34cr. It’s currently
  > required that guests of the camkes-vm run in 32-bit mode, so install 32-bit
  > Ubuntu.
* Support for hardware passthrough, however it is severely limited and it
  requires manual configuration, including IO port assignment, memory mapping,
  interrupts setup. This is completely not portable and cannot be used in this
  form. Normally, VM config is written in CAmkES specific language, which is
  processed during build. VM config cannot be generated at runtime making robust
  HW passthrough impossible.
* Lack of support for AMD-V. Until this is implemented there is no
  virtualization on AMD CPUs.

We expect some of these limitations to be fixed by the
[Makatea](https://trustworthy.systems/projects/TS/makatea) project, which aims
at creating a [Qubes](https://www.qubes-os.org/)-like OS on top of seL4.

If CAmkES had good enough virtualization we could take the following approach to
build secure OS for running
[Fobnail Attester](https://github.com/fobnail/fobnail-attester):

* Fobnail Attester would have to be split into 2 parts: part that communicates
  with Fobnail Token through network and a separate part called secure
  component.
* Secure component would be a program running natively under microkernel and it
  would be responsible for:
  - Loading next stage OS into RAM (disk drivers would be located in Linux VM)
    and extending PCRs early during minimal OS startup.
  - Starting Fobnail Attester in Linux.
  - Acting as a proxy between Fobnail Attester and TPM - only operations
    required to perform platform provisioning/attestation like TPM quote,
    reading EK certificate, etc. (this must be a minimal component providing
    only the most basic set of features, it must be verified that it has no
    exploitable bugs)
* Secure component would be responsible for booting target OS after attestation
  is successful.

With this approach, even if VM got compromised it wouldn't break attestation.
Linux VM can't boot another OS, it can only signal secure component to boot the
exact OS that has been loaded earlier and nothing more. We avoid a situation
when compromised VM performs successful attestation but boots something else.

Also, it could be possible to use Rump kernels instead of Linux VM. Rump is a
library kernel capable of running unmodified Netbsd drivers. Usually it is used
in Netbsd to test drivers in userspace, however it has been used before to
provide drivers for other kernels through the
[Rumprun](https://github.com/rumpkernel/rumprun) project. Please note that it is
unikernel which is intended to run in VM, but Rump itself (theoretically) could
be adapted to run as CAmkES/seL4 component, eliminating the need for VM.

#### License

In case of `CAmkES` we can verify the licenses by checking
[LICENSE](https://github.com/seL4/camkes/blob/master/LICENSE.md#license)
document on the project repository. It uses standard open source licenses.

### Linux

In the case of Linux, a minimal distribution will be prepared that meets the
requirements of the Fobnail Project.
[Yocto Project](https://www.yoctoproject.org/) will be used for this, because it
gives a lot of freedom in manipulating the elements that make up the target
system.

#### Running in DLME

Ensuring that the operating system works in DLME can be achieved by using the
[TrenchBoot](https://trenchboot.org/) project. For this purpose, a Yocto
[meta-fobnail](https://github.com/fobnail/meta-fobnail) layer has been created
that integrates the necessary elements. The effects of the tests are presented
in a [separate document](./meta-fobnail-in-dlme.md).

#### Fobnail integration

At this point, the [Fobnail Attester](https://github.com/fobnail/fobnail-attester)
application is strongly dependent on Linux. Mainly due to the fact that its
development took place on this operating system. Therefore, it is important that
the created minimal OS also has an integrated attester application. The main
dependencies of its operation are drivers for USB, USB EEM and TPM, but running
them under Linux will not cause major problems.

#### License

The Linux Kernel runs under GPL-2.0 License which can be checked in
[documentation](https://www.kernel.org/doc/html/latest/process/license-rules.html#linux-kernel-licensing-rules).

### Linux with Busybox userspace

The main advantage of Busybox based userspace is its small size. Also booting
time would be significantly faster than userspace with fully-featured init
system. With Busybox, init system can be replaced by a simple shell script.

#### Running in DLME

Since we are still running Linux, there are no problems here.

#### Fobnail integration

Since busybox based userspace doesn't have big expectation of Linux feature set,
kernel can be reduced. Busybox can be compiled with only those features we need.
Currently, Fobnail Attester depends on another program to configure network
interface created by Fobnail Token. This is `systemd-networkd`, but in Busybox
userspace it would have to be something else. It should be noted that
configuration should be done on hotplug event, simply using `ifconfig` may be
unreliable because:

* It won't work if Fobnail Token isn't already plugged in.
* It won't work if Fobnail Token is plugged out and back in.

Hotplug detection and interface setup could be implemented in
[Fobnail Attester](https://github.com/fobnail/fobnail-attester) through Netlink.

### Linux with Go userspace

[u-root](https://github.com/u-root/u-root) provides an easy way to deploy Go
programs as standalone initramfs images. It is used by the
[LinuxBoot](https://www.linuxboot.org/) project for creating small Linux distro
that acts as bootloader for other OSes (through kexec).

#### Running in DLME

Since we are still running Linux, there are no problems here.

#### Fobnail integration

[Fobnail Attester](https://github.com/fobnail/fobnail-attester) is written in C,
and it rather doesn't make sense to integrate it with Go userland. Also, it
should be noted that `u-root` built initramfs contains only executable - all Go
modules are linked into a single binary. Unless we want to rewrite Fobnail
Attester there is no point in using userspace that was designed be Go-only.
Rewriting Fobnail Attester in Go may give benefits of producing less vurnelable
code.

### Little Kernel / Trusted Little Kernel

Little Kernel is a small, embedded monolithic kernel that runs on x86, ARM and
RISC-V among others, provides a custom very basic libc. LK focuses on ARM
platforms and lacks drivers for devices typically found in x86 platforms. LK has
been used as a base for aboot (Android bootloader) and
[Fuchsia OS](https://fuchsia.dev/).

Trusted Little Kernel (available at
`https://nv-tegra.nvidia.com/r/3rdparty/ote_partner/tlk.git`, not accessible
through web browser - must be cloned with `git`) is a fork of LK intended to be
run as TEE OS. It is developed by NVIDIA for Tegra platforms, support for all
other platforms has been removed.

#### Running in DLME

LK boots over Multiboot 1 which is not supported by SKL. The problem is similar
to that with Zephyr (see [Zephyr](#running-in-dlme) section for detail). However
Multiboot2 support could be added.

#### Fobnail integration

Running [Fobnail Attester](https://github.com/fobnail/fobnail-attester) on LK
would be problematic to say the last and the reasons for that are:

* LK lacks most drivers including USB host drivers (UHCI, OHCI, EHCI, XHCI),
  USB EEM driver and TPM drivers.
* Lack of cryptographic library in LK. TLK does provide one (potentially could
  be).
* Default C library is too much constrained - provides only most basic API like
  string operations, printing, etc. lacks anything more complex, including
  networking support, there is a LK's
  [fork](https://github.com/littlekernel/newlib) of Newlib, however of unknown
  quality.
* TCP/IP is provided by `minip` library (which is part of LK), attester depends
  on `libcoap3` which either would have to be ported to `minip` or `minip` would
  have to be integrated with `newlib`.
* LK has no suitable bootloader capabilities
  [lkboot](https://github.com/littlekernel/lk/tree/master/app/lkboot) could
  serve as reference to develop custom bootloader.

#### License

Little Kernel uses
[MIT License](https://github.com/littlekernel/lk/blob/master/LICENSE).

### Fuchsia (Zircon)

[Fuchsia](https://fuchsia.dev) is a general-purpose OS developed by Google. Its
purpose is to replace GNU/Linux in Google products (like Android, Chrome OS)
with Zircon microkernel (used to be known as Magenta) and custom userland.
Fuchsia runs on x86-64 and ARM64. There used to be an
[unofficial](https://github.com/slavaim/riscv-magenta) RISC-V port, however it
has been unmaintained for years. Support for 32-bit architectures was dropped
some time ago.

Zircon is based on LK, however similarities between these two are small and
constantly decreasing. LK is a kernel intended for embedded usage, while Zircon
is a general-purpose microkernel that could compete with Linux.

#### Running in DLME

Historically Zircon used Multiboot 1 for booting. Now it uses ZBI with a UEFI
bootloader called
[Gigaboot](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/firmware/gigaboot/).
Multiboot 1 support is still present, but SKL doesn't support it anyway. See
[Zephyr](#running-in-dlme) section above for details.

#### Fobnail integration

Fuchsia provides a Unix-like environment with musl C library. Network is
available but there is no EEM driver, and only XHCI host driver is supported.
Since EHCI is still widely used, this is a major limitation. TPM driver is
available but without `libtss2`.

Zircon has a kexec-like capability which allows to boot another Zircon instance.
This is known as `mexec`.

Following things would have to be done:

* Bring `libtss2` stack to Zircon.
* Bring EEM driver. There is ECM driver available, which could serve as
  reference.
* Bring at least USB EHCI driver, ideally should support UHCI and OHCI too.
* Figure out how to use `mexec` to boot Linux.

#### License

Information can be checked
[here](https://fuchsia.dev/fuchsia-src/contribute/governance/policy/open-source-licensing-policies).
Basically, we will be using here components under MIT, BSD and Apache 2.0
licenses.

### Xen

Xen is a bare-metal hypervisor available on x86 and ARM. It is able to run Linux
in a paravirtualized environment. Xen contains only drivers absolutely necessary
for hypervisor and VM to boot. Control over most of the hardware is handed to
the first VM known as [Dom0](https://wiki.xenproject.org/wiki/Dom0). Usuallly
Linux runs in Dom0, there are other OSes which also support running in Dom0
(BSD, Solaris), however there is no advantage of using them over Linux. Xen also
has ability to run in
[dom0-less](https://xenbits.xen.org/docs/unstable/features/dom0less.html) mode -
bootloader provides VM images and Xen starts VMs on their own (see section below
for details).

#### Running in DLME

Xen is supported by Trenchboot's GRUB and can run without any modifications.
See [xen-in-dlme.md](./xen-in-dlme.md) for demo.

#### Integration with Fobnail

Domain 0 less mode eliminates need for trusted VM. Also, since Xen itself is
responsible for booting multiple VMs there is no need for tools that manage Xen
(which don't work on any OS besides Linux). Fobnail Attester could be integrated
in a similar way that with CAmkES based VM (see CAmkES section above for
[details](#fobnail-integration-4)). The main difference would be that the secure
component would be a separate VM.

So it look like we could use Xen while we talk about early launch but in late
launch scenarios it makes no sense. And this is one of key reasons why Xen does
not meet the minimal OS requirements, because implementing late launch scenarios
is definitelly something that we would like to achive with the OS.

#### License

Xen Project is developed under [GPL-2.0
License](https://xenproject.org/about-us/).

## Summary

* The above report outlines 8 operating systems that should be considered
  candidates for Fobnail Token interoperability.
* The basic choice is Linux, the operating system based on it is created with
  the use of [Yocto Project](https://www.yoctoproject.org/). All achievements
  can be reproduced at any time using the
  [meta-fobnail](https://github.com/fobnail/meta-fobnail) layer.
* As part of the report, other systems were also checked. The possibility of
  running them in DLME and the integration of the
  [Fobnail Attester](https://github.com/fobnail/fobnail-attester) application
  was checked for each of them.
* The description of an attempt to run Zephyr on PC Engines apu2 in order to
  verify the current state of the system to work with Fobnail Token has been
  included. The effects are described in the [PoC test](#poc-test) section.
* The table below contains summary of features of all OSes together with theirs
  scores. Please note that when comparing driver support we take into account
  not only drivers which are part of the kernel, but also available userspace
  drivers. Each score is computed using the following rules:
  - if feature is present then +1,
  - if feature is not present then 0, or if feature would be hard to implement
    then -1,
  - if this is a hard requirement then multiple result by 2.

Definition of requirements can be found [here](#os-requirements).

| OS      | USB host driver  | USB EEM driver   | Network stack     | TPM driver        | OS portability | Bootloader capabilities | C library    | Microkernel | CPU Architecture support | Bootable by SKL | License | Score |
| ------- | ---------------- | ---------------- | ----------------- | ----------------- | -------------- | ----------------------- | -------------| ----------- | ------------------------ | --------------- | ------- | ----- |
| Zephyr  | Yes (+2)         | Yes (+2)         | Yes (+2)          | PoC available (0) | Limited (-1)   | No (0)                  | Yes (+2)     | No (0)      | Good (+1) [^4]           | No (0) [^6]     |  OK  | 8     |
| Xous    | No (0)           | No (0)           | Yes (+2)          | No (0)            | Limited (-1)   | No (0)                  | No (0)       | Yes (+1)    | RISC-V only (-1)         | No (0)          |  OK  | 1     |
| seL4    | No (0) [^1]      | No (0) [^2]      | Yes (+2)          | No (0)            | Limited (-1)   | No (0)                  | Yes (+2)     | Yes (+1)    | Good (+1) [^3]           | Yes (+2)        | OK (but problematic with Genode) | 7     |
| Linux   | Yes (+2)         | Yes (+2)         | Yes (+2)          | Yes (+2)          | Yes (+1)       | Yes (kexec) (+2)        | Yes (+2)     | No (0)      | Good (+1) [^5]           | Yes (+2)        |   OK  | 16    |
| LK      | No (0)           | No (0)           | Limited (-2) [^7] | No (0)            | Yes (+1)       | No (0)                  | Limited (-2) | No (0)      | Good (+1) [^8]           | No (0)          |  OK  | -2    |
| Fuchsia | Limited (0) [^9] | No (0)           | Yes (+2)          | Limited (0) [^10] | Yes (+1)       | Yes (mexec) (+2)        | Yes (+2)     | Yes (+1)    | Good (+1) [^11]          | No (0)          |  OK  | 7     |

[^1]: seL4 has an old unmaintained driver with no xHCI support. Better driver is
      available only from Genode.

[^2]: available from Genode only

[^3]: supports x86, ARM and RISC-V

[^4]: supports x86, ARM, RISC-V, SPARC and MIPS

[^5]: supports most of the architectures, much more than any other OS listed
      here

[^6]: not bootable due to of lack Multiboot1 support in SKL

[^7]: uses custom library, no integration with libc, which complicates using it
      with `libcoap3`

[^8]: supports x86, ARM, RISC-V and MIPS

[^9]: supports XHCI only

[^10]: TPM driver is present but there is no `libtss2` support

[^11]: supports 64-bit x86 and ARM. Support for 32-bit architectures has been
       dropped a while ago and is not coming back

## KUDOS

We want to thank the people listed below for their help in this research (listed
alphabetically).

- Andrew Cooper ([@andyhhp](https://github.com/andyhhp))
- Sid Hussmann ([@sidhussmann](https://github.com/sidhussmann))
- Marek Marczykowski-Górecki ([@marmarek](https://github.com/marmarek))
- Demi Marie Obenour ([@DemiMarie](https://github.com/DemiMarie))
- Daniel P. Smith ([@dpsmith](https://github.com/dpsmith))

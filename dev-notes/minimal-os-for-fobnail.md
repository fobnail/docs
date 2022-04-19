# Minimal OS for Fobnail project

The Fobnail project aims to provide a reference architecture for building
offline integrity measurement servers on the USB device and clients running in
Dynamically Launched Measured Environments (DLME). It should allow the Fobnail
owner to verify the trustworthiness of the running system before performing any
sensitive operation.

According to the information provided at TrenchBoot Summit 2021 (the fragment
discussed here - https://youtu.be/xZoCtNV8Qs0?t=5062), the minimum OS, together
with the connected Fobnail Token, should allow determining the state of the
platform before the target OS is launched. As we can see in the presentation,
`heads` would be responsible for the platform's trust control, while in this
document, we want to present the results of research on the possibilities of
using various operating systems.

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

Zephyr is an well-known and supported embedded RTOS with monolithic kernel
written in C. It supports variety of devices ranging from Cortex-M to x86 CPUs
and provides support for IPv4, CoAP and USB needed for communication with
Fobnail Token.

#### Running in DLME

<!--
TBD
-->

#### Fobnail integration

Zephyr provides most of the drivers needed for integration:

- USB hub drivers
- USB EEM driver
- Network stack with IPv4 and CoAP support

TPM driver is missing, however there is a fairly recent
[PoC implementation](https://github.com/drandreas/zephyr-tpm2-poc) of TPM2
stack.

Zephyr provides good support for standard C library (newlib) which should
simplify [fobnail-attester](https://github.com/fobnail/fobnail-attester)
porting.

### Xous

<!--
TBD - short overview
-->

#### Running in DLME

<!--
TBD
-->

#### Fobnail integration

<!--
TBD
-->

### seL4

seL4 is a secure L4 family microkernel written in C. It has strong security
guarantees assured by [Format proofs](https://sel4.systems/Info/FAQ/proof.pml),
however these proofs are still incomplete for x86, see
[Supported Platforms](https://docs.sel4.systems/Hardware) for an up-to-date
verification status. seL4 due to its microkernel nature provides higher
isolation, breach in one of the components (like USB driver, network stack)
wouldn't compromise entire OS contrary to monolithic kernels.

#### Running in DLME

It should be runnable, however due to microkernel nature all programs run in
unprivileged mode which may complicate booting of the target OS.

#### Fobnail integration

seL4 provides virtually no drivers, except few drivers listed
[here](https://docs.sel4.systems/projects/available-user-components.html).
According to the page linked above `libusbdrivers` is inactive, it also lacks
XHCI support. seL4 has a basic support for an old version of musl C library
(v1.1.16).

Using seL4 would require significant amount of work:
- Extending USB drivers
- Implementing USB EEM driver
- Implementing TPM driver
- Porting [fobnail-attester](https://github.com/fobnail/fobnail-attester)

[Genode](https://github.com/genodelabs/genode) can run on seL4 and provides
drivers at least for some devices and a network stack. Its support for seL4 used
to be incomplete and many components were broken, however that might have
changed. It may be an easier way to get software running on seL4.

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

<!--
ideas:
* run one of the Xous/seL4/Zephyr on PC engines apu2
*
-->

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

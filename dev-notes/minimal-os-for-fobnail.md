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

### Linux

In the case of Linux, a minimum distribution will be prepared that meets the
requirements of the project. [Yocto Project](https://www.yoctoproject.org/) will
be used for this, because it gives a lot of freedom in manipulating the elements
that make up the target system.

#### Running in DLME

Ensuring that the operating system works in DLME can be achieved by using the
[TrenchBoot](https://trenchboot.org/) project. For this purpose, a Yocto
[meta-fobnail](https://github.com/fobnail/meta-fobnail) layer has been created
that integrates the necessary elements. The effects of the tests are presented
in a [separate document](./).

#### Fobnail integration

<!--
TBD
describe that fobnail is strongly linux depend right now
-->

## PoC test

<!--
ideas:
* run one of the Xous/seL4/Zephyr on PC engines apu2
*
-->

## Summarize

<!--
TBD
-->

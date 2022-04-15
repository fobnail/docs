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

## Building

As mentioned earlier, we prepared a project using Yocto. For layers management
and simplification of the build process, we use
[kas](https://kas.readthedocs.io/en/latest/) tool with
[kas-container](https://github.com/siemens/kas/blob/master/kas-container)
script. All build components are open and public, so everyone can try to
reproduce our effort if only the prerequisites are met.

### Test of latest Trenchboot

From Trenchboot sources

#### 3mdeb GRUB fork

#### W/A patch for SKL

Link to TB issue

### Working solution

linux v5.8
landing-zone
3mdeb fork for GRUB

## Summary

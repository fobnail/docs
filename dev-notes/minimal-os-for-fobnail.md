# Minimal OS for Fobnail project

This document contains a set of information regarding the minimal OS to be
delivered in the Fobnail project.

## Previous informations

> Note: May contain links to an internal repos

Initial documentation with notes about minimal OS.

* https://gitlab.com/3mdeb/drtmkey/presentations/-/blob/master/fobnail-nitrokey.md#fobnail-enabled-boot-flow
  - heads with kexec then run operating system

* https://gitlab.com/3mdeb/drtmkey/docs/-/blob/master/description.md#project-description
  - description of Fobnail project

For phase 8 and 9?

```
To demponstrate the capabilities of implemented solution,
we will create the example use case that uses the TrenchBoot - our previous
NLnet project [9], to verify the integrity of the system firmware, D-RTM
Configuration Environment (DCE), Linux Kernel, and initrd.
```

Mention about Zephyr RTOS

```
The main use case that will be presented as a demonstration is boot
time attestation. The Fobnail Token will be used to verify an operating system
during boot time. We will run inside DLME [10] (Dynamically Launched Measured
Environment) the Zephyr RTOS which will check if the measurements of the
operating system met the reference measurements in the Fobnail Token.
If OS is compromised the Fobnail Token warns the user and Zephyr RTOS
prevents a platform from booting.
```

* Roadmap for phase 8 and 9

Project
doc - https://gitlab.com/3mdeb/drtmkey/docs/-/blob/master/fobnail-project-offload.md#fobnail-roadmap-eighth-phase

## Proposed OSes

* Zephyr RTOS

* seL4

* Xous

* Linux

## Questions

* Whether the supplied minimal OS is to be the target operating system or the
  system that then boots the target system?

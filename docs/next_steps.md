## Integrate Fobnail with NitroKey firmware

https://github.com/Nitrokey/nitrokey-3-firmware/issues/276

## Implement HID communication for Attester and Platform Owner applications

This removes network stack from code base. On the other hand, it requires
significant changes on applications running on PC, since they can no longer
benefit from CoAP libraries and network stack built into kernel.

## Add ability to provision more than one platform to Fobnail Token

Currently only one platform at a time can be provisioned with one Fobnail Token.
For easier usage for people maintaining multiple platforms, as well as for
economic reasons, it makes sense to be able to use one Fobnail Token for
multiple platforms.

With multiple platforms supported, it should also be possible to remove a single
platform from the list of provisioned ones. Otherwise, when one machine is
decommissioned, all others would have to be provisioned again after clearing
Fobnail's memory.

## Removing MAC address from platform metadata

Currently, MAC address of the primary NIC is used as part of platform metadata
described [here](https://fobnail.3mdeb.com/fobnail-api/#adminprovisionidmeta).
Determining which NIC is the primary one has been problematic due to lack of
standardized way to do so - procedure depends on how the NIC is connected, this
should be PCI on virtually any x86 platform, but not on other architectures.

- TODO: we could discuss other changes to platform metadata if so desired.

## Recovery mechanism

We need some mechanism to recover secrets in case attestation fails. Attestation
may fail due to two causes: either the platform has been compromised and is no
longer trusted, or after firmware/OS update. Updating platform's firmware
inevitably changes PCR measurements, requiring re-provisioning of the platform.

Keeping an access would require user to re-provision the platform prior to
installing updates. If user failed to do so, the access would be lost. Going
into previous state may not always be feasible - anti-rollback protection in
device's firmware, or inability to obtain old packages from OS repository in
order to perform OS rollback.

The problem is even more serious as Ubuntu automatically installs security
updates which could change measurements. Other distros are likely to follow.

This is related to [#44](https://github.com/fobnail/fobnail/issues/44) but quite
different as is not about backing up the keys, but regaining access to whatever
is stored inside.

## Windows support

Currently Windows is not supported by Fobnail. First, CDC EEM (the protocol used
for IP-over-USB) is not working with Windows usbnet driver (this would no longer
be a problem after dropping IP).

The second problem is that `fobnail-attester` application has never been built
nor tested on Windows. Esys from
[tpm2-tss](https://github.com/tpm2-software/tpm2-tss) can be built on Windows
which should allow getting `fobnail-attester` up and running.

TODO: discuss Windows support

## Packaging of fobnail-attester

Packaging of `fobnail-attester` for various distributions (we would support the
same distributions as Nitrokey). Users of a supported distribution could easily
install software and start using Fobnail. Also, `fobnail-attester` could be
easily pulled-in as a dependency by Nitrokey software.

## Multiple enhancements to applications

- [Add option to use offline EK certificate chain](https://github.com/fobnail/fobnail-attester/issues/37)
- [Don't block TPM](https://github.com/fobnail/fobnail-attester/issues/35)
- [Add support for different use cases](https://github.com/fobnail/fobnail-platform-owner/issues/3)

## Update documentation

Top level description wasn't changed since first PoC. Project evolved
significantly since then, and information located there is no longer describing
the state of the project.

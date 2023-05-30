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

## Multiple enhancements to applications

- [Add option to use offline EK certificate chain](https://github.com/fobnail/fobnail-attester/issues/37)
- [Don't block TPM](https://github.com/fobnail/fobnail-attester/issues/35)
- [Add support for different use cases](https://github.com/fobnail/fobnail-platform-owner/issues/3)

## Update documentation

Top level description wasn't changed since first PoC. Project evolved
significantly since then, and information located there is no longer describing
the state of the project.

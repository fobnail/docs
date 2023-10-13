# What is Fobnail Project?

Fobnail Project provides resources to create axiomatically trustworthy
[device][mccune] and simple user interface to attest platform state. For a
detailed project description, please check [here](description).

* [Fobnail firmware][fobnail_fw] is an open-source implementation of the
  [iTurtle][mccune] security architecture concept presented at HotSec07; in
  addition, it will leverage industry standards like [TCG D-RTM][tcg_drtm]
  trusted execution environment and IEFT RATS. The Fobnail project aims to
  provide a reference architecture for building offline integrity measurement
  verifiers on the USB device and attesters running in Dynamically Launched
  Measured Environments (DLME). It allows the Fobnail owner to verify the
  trustworthiness of the running system before performing any sensitive
  operation. Fobnail does not need an Internet connection, which makes it immune
  to the network stack and remote infrastructure attacks. It brings the power of
  solid system integrity validation to the individual in a privacy-preserving
  solution.
* [The Fobnail Token](fobnail_token) is a tiny open-source hardware reference
  design of a USB device that provides a means for a
  user/administrator/enterprise to determine the integrity of a system. To make
  this determination, Fobnail Token leverages [Fobnail firmware][fobnail_fw],
  which acts as an attestor capable of validating attestation assertions made by
  the system. As an independent device, Fobnail Token provides a high degree of
  assurance that an infected system cannot influence Fobnail Token as it
  inspects the attestations made by the system.

## Where to go next?

* Get familiar with [Fobnail architecture](architecture.md)
* Read Fobnail Project [detailed description](description.md)
* [Building instructions](building.md)
* How to support the project?

## Other resources

* [TPM remote attestation over Bluetooth][gkerneis] blog post by Gabriel
  Kerneis, where he mentioned the Fobnail project.

[mccune]: https://www.usenix.org/legacy/event/hotsec07/tech/full_papers/mccune/mccune_html/index.html
[fobnail_fw]: https://github.com/fobnail/fobnail
[tcg_drtm]: https://trustedcomputinggroup.org/wp-content/uploads/DRTM-Specification-Overview_June2013.pdf
[gkerneis]: https://gabriel.kerneis.info/2021/11/local_attestation/

## Where to buy?
* The Fobnail Token Development Kit can be ordered at the [3mdeb online store](https://shop.3mdeb.com/shop/open-source-hardware/fobnail-token-development-kit/).

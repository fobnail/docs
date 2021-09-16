# What is Fobnail?

The Fobnail Token is a tiny open-source hardware USB device that provides
a means for a user/administrator/enterprise to determine the integrity of
a system. To make this determination, Fobnail functions as an attestor capable
of validating attestation assertions made by the system. As an independent
device, Fobnail provides a high degree of assurance that an infected system
cannot influence Fobnail as it inspects the attestations made by the system.

Fobnail software is an open-source implementation of the iTurtle security
architecture concept presented at HotSec07; in addition, it will leverage
industry standards like TCG D-RTM trusted execution environment and IEFT
RATS. The Fobnail project aims to provide a reference architecture for
building offline integrity measurement servers on the USB device and clients
running in Dynamically Launched Measured Environments (DLME). It allows the
Fobnail owner to verify the trustworthiness of the running system before
performing any sensitive operation. Fobnail does not need an Internet
connection what makes it immune to the network stack and remote infrastructure
attacks. It brings the power of solid system integrity validation to the
individual in a privacy-preserving solution.

## Table of content

* [Project description](description.md)
* [Fobnail architecture](architecture.md)
* [Environment setup](environment.md)
* [Flashing sample applications](flashing_samples.md)

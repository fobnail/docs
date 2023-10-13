# Fobnail Project detailed description

The Fobnail Project is an implementation of the [iTurtle][mccune].

The iTurtle concept presents a portable device that is a security USB stick,
which provides platform trustworthiness checks. The result of the integrity
check must be presented in an easy, visible way - for example, with the LED. We
([3mdeb](https://3mdeb.com)) want to provide the open-source reference
implementation of the iTurtle device.

Our goal is to create a scalable, flexible implementation that will be the
core building block for future security solutions (created by the community)
based on DRTM payloads. The Fobnail architecture concept is based on the [IEFT
specification - Remote ATtestation ProcedureS (RATS)][rats].

The architecture supports [Fobnail Token provisioning][arch_prov],
[remote][r_att_prov] and [local][l_att_prov] Platform (Attester) provisioning,
and [attestation][attestation]. Currently, there is no open source
implementation of the USB attestation device.

During the Fobnail Token provisioning, the Platform Owner creates the identity
and encryption certificates that are used by the Attester to the Fobnail Token
verification. The Fobnail Token verifies the Platform Owner certificate chain
and generates the identity and encryption key pair. The Fobnail Token obtains
the metadata (CPU serial) and conveys the encrypted public key parts and
metadata to the Platform Owner. The received data is decrypted, and the Platform
Owner creates identity and encryption certificates that are provided to the
Fobnail Token.

The Platform provisioning is split into remote provisioning (which is out of
[NLNet grant application][nlnet] scope) and local provisioning. In the case of
local provisioning, the Fobnail Token takes the Platform Owner role. The
Fobnail Token receives the Attester metadata (CPU serial, MAC, and EK
certificate checksum) and generates the checksum. The metadata checksum is used
for the Platform verification during the attestation. The Fobnail Token uses
default policies during the local attestation.

The [attestation architecture][attestation] is based on the Reference
Interaction Model for Challenge-Response-based Remote Attestation (CHARRA).
This is the IEFT attestation standard, which specifies the communication
between the Attester and the Verifier (Fobnail Token). The Fobnail Token
provides the ability to perform attestation in the absence of any connectivity.
During the attestation development, we will use the [proof-of-concept CHARRA
implementation][charra] of the proposed architecture, which is developed by the
Fraunhofer Institute.

To demonstrate the capabilities of the implemented solution, we will create the
example use case that uses the [TrenchBoot][tb], of which a significant part of
AMD support was sponsored by [NLNet as OpenDRTM][nlnet_tb], to verify the
integrity of the system firmware, D-RTM Configuration Environment (DCE), Linux
Kernel, and initrd.

The main use case that will be presented as a demonstration is boot time
attestation. The Fobnail Token will be used to verify an operating system during
boot time. We will run inside [DLME][tcg_drtm] (Dynamically Launched Measured
Environment) the Zephyr RTOS, which will check if the measurements of the
operating system meet the reference measurements in the Fobnail Token. If the OS
is compromised, the Fobnail Token warns the user, and Zephyr RTOS prevents a
platform from booting.

The Fobnail Project has the potential to scale up in the future. The Fobnail
architecture could be adapted for devices using different buses than USB. We
would like to implement another DRTM-based use case (that is out of the scope
of this grant application), for example, runtime OS attestation. The Fobnail
could be used to check if the runtime state of the platform is trusted. It
could be based on one of the existing projects, such as [Linux Kernel Runtime
Guard][lkrg]. It could be useful to check the integrity of the Linux Kernel
before performing sensitive operations.

[mccune]: https://www.usenix.org/legacy/event/hotsec07/tech/full_papers/mccune/mccune_html/index.html
[rats]: https://datatracker.ietf.org/doc/draft-ietf-rats-architecture/
[arch_prov]: architecture.md#Fobnail-provisioning-diagram
[r_att_prov]: architecture.md#Remote-platform-provisioning-diagram
[l_att_prov]: architecture.md#Local-platform-provisioning-diagram
[attestation]: architecture#Attestation
[nlnet]: https://nlnet.nl/project/Fobnail/
[charra]: https://github.com/Fraunhofer-SIT/charra
[tb]: https://trenchboot.org/
[nlnet_tb]: https://nlnet.nl/project/OpenDRTM/
[tcg_drtm]: https://trustedcomputinggroup.org/wp-content/uploads/DRTM-Specification-Overview_June2013.pdf
[lkrg]: https://www.openwall.com/lkrg

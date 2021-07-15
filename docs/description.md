# Project description

The Fobnail project is an implementation of the iTurtle idea shown in the
research paper [[1]](https://www.usenix.org/legacy/event/hotsec07/tech/full_papers/mccune/mccune_html/index.html).
 The iTurtle concept presents the portable device that is
a security USB stick, which provides platform trustworthiness checks.
The result of the integrity check must be presented in an easy, visible way -
for example with the LED. We (3mdeb team) want to provide the open source
implementation of the iTurtle device - the Fobnail Token.
Our goal is to create a scalable, flexible implementation which will be
the a core building-block for the future security solutions (created by
community) based on DRTM payloads. The Fobnail Token architecture concept
is based on the IEFT specification - Remote ATtestation ProcedureS (RATS)
[[2]](https://datatracker.ietf.org/doc/draft-ietf-rats-architecture/).
The architecture supports Fobnail Token provisioning
[[3]](architecture.md#Fobnail-provisioning-diagram), a Platform
(Attester) provisioning [[4]](architecture.md#Remote-platform-provisioning-diagram)
[[5]](architecture.md#Local-platform-provisioning-diagram), and the attestation
[[6]](architecture#Attestation). Currently,
there is no open source implementation of the USB attestation device.

During the Fobnail Token provisioning [[3]](architecture.md#Fobnail-provisioning-diagram),
the Platform Owner creates the identity and encryption certificates that are
used by the Attester to the Fobnail Token verification. The Fobnail Token
verifies the Platform Owner certificate chain and generates the identity and
encryption key pair. The Fobnail Token obtains the metadata (CPU serial) and
conveys the encrypted public key parts and metadata to the Platform Owner. The
received data is decrypted and the Platform Owner creates identity and
encryption certificates, that are provided to the Fobnail Token.

The Platform provisioning is split into external provisioning
(which is out of grant application scope)
[[4]](architecture.md#Remote-platform-provisioning-diagram) and local
provisioning [5]. In the case of local provisioning
[[5]](architecture.md#Local-platform-provisioning-diagram), the Fobnail Token
takes the Platform Owner role. The Fobnail Token receives the Attester metadata
(CPU serial, MAC, and EK certificate checksum) and generates the checksum.
The metadata checksum is used for the Platform verification during the
attestation. The Fobnail Token uses default policies during the local
attestation.

The attestation architecture [[6]](architecture#Attestation) is based on the
Reference Interaction Model for Challenge-Response-based Remote Attestation
(CHARRA) [[7]](https://tools.ietf.org/id/draft-birkholz-rats-reference-interaction-model-00.html).
This is the IEFT attestation standard, which specifies the communication
between the Attester and the Verifier (Fobnail). The Fobnail provides the
ability to perform attestation in absence of any connectivity. During the
attestation development, we will use the proof-of-concept implementation
[[8]](https://github.com/Fraunhofer-SIT/charra) of the proposed architecture,
which is developed by the Fraunhofer Institute.

To demponstrate the capabilities of implemented solution,
we will create the example use case that uses the TrenchBoot - our previous
NLnet project [[9]](https://nlnet.nl/project/OpenDRTM/), to verify the integrity
of the system firmware, D-RTM Configuration Environment (DCE), Linux Kernel,
and initrd.

The main use case that will be presented as a demonstration is boot
time attestation. The Fobnail Token will be used to verify an operating system
during boot time. We will run inside DLME
[[10]](https://trustedcomputinggroup.org/wp-content/uploads/DRTM-Specification-Overview_June2013.pdf)
(Dynamically Launched Measured Environment) the Zephyr RTOS which will check
if the measurements of the operating system met the reference measurements
in the Fobnail Token. If OS is compromised the Fobnail Token warns the user
and Zephyr RTOS prevents a platform from booting.

The Fobnail has the potential to scale up in the future. The Fobnail architecture
could be adapted for devices using different buses than USB. We would like to
implement another DRTM based use case (that is out of the scope of this grant
application) for example runtime OS attestation. The Fobnail could be used to
check if the runtime state of the platform is trusted. It could be base on one
of the existing projects such as Linux Kernel Runtime Guard [[11]](https://www.openwall.com/lkrg/).
It could be useful to check the integrity of the Linux Kernel before performing
sensitive operations.

[1] "Turtles All The Way Down: Research Challenges in User-Based Attestation",
Jonathan M. McCune, Adrian Perrig, Arvind Seshadri, Leendert van Doorn,
https://www.usenix.org/legacy/event/hotsec07/tech/full_papers/mccune/mccune_html/index.html

[2] "Remote Attestation Procedures Architecture",
H. Birkholz, D. Thaler, M. Richardson, N. Smith, W. Pan,
https://datatracker.ietf.org/doc/draft-ietf-rats-architecture/

[3] "Fobnail Token provisioning - Fobnail architecture", M. Zmuda, N. Kamiński,
https://drive.google.com/file/d/143wW4zWcLbedbuaWMlQpC9f_CMCcXdLH/view?usp=sharing

[4] "External platform provisioning - Fobnail architecture", M. Zmuda, N. Kamiński,
https://drive.google.com/file/d/1WGBbO9bCFlnarPCZzrKXUjpH4hzbf5l9/view?usp=sharing

[5] "Local platform provisioning - Fobnail architecture", M. Zmuda, N. Kamiński,
https://drive.google.com/file/d/12gqhdKBlFAMOOkjZu99RPVI_iy_nWfxE/view?usp=sharing

[6] "Local platform provisioning - Fobnail architecture", M. Zmuda, N. Kamiński,
https://drive.google.com/file/d/1L3CBQsh-qQ4I-7mDj0OYdpK73p_bgKS0/view?usp=sharing

[7] "Attestation - Fobnail architecture", M. Zmuda, N. Kamiński,
https://tools.ietf.org/id/draft-birkholz-rats-reference-interaction-model-00.html

[8] "CHARRA: CHAllenge-Response based Remote Attestation with TPM 2.0"
https://github.com/Fraunhofer-SIT/charra

[9] "Open Source DRTM implementation with TrenchBoot for AMD processors"
https://nlnet.nl/project/OpenDRTM/

[10] "The TCG Dynamic Root for Trusted Measurement", Lee Wilson
https://trustedcomputinggroup.org/wp-content/uploads/DRTM-Specification-Overview_June2013.pdf

[11] "LKRG - Linux Kernel Runtime Guard"
https://www.openwall.com/lkrg/

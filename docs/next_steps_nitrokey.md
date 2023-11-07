# Propose and document new API design based on CTAPHID

With CTAPHID used instead of CoAP, human-readable names of endpoints should be
changed to numeric IDs. Most of existing Fobnail communication already is
encoded as CBOR, so modifications should be straightforward. Some additional
information is passed as metadata (CoAP URI and response codes), and it shall be
included in CBOR body to keep implementation consistent and compliant with
CTAPHID.

# Implement functions specific to Fobnail in appropriate Nitrokey library

Newly designed CTAPHID commands (and potentially subcommands and parameters)
must be supported by library responsible for communication with Nitrokey dongle.
The exact library will depend on choices made during design and implementation
based on constraints and limitations of hardware and chosen software stack.
The library will be covered with unit test to ensure the proper implementation
of the documented API.

# Rewrite Attester application to be compatible with Fobnail Token changes

This task consists of supporting new communication interface. All uses of CoAP
will be removed, and API provided by library developed in previous phase will be
used instead.

# Change the way metadata is obtained

Currently, metadata is created from SMBIOS tables and MAC address of primary
network controller, as described [here](https://fobnail.3mdeb.com/fobnail-api/#adminprovisionidmeta).
Accessing SMBIOS requires root privileges, and reliably determining proper
network interface proved to be difficult, especially with USB network adapters
or systems with no network drivers like [Heads](https://osresearch.net). A new
way of obtaining metadata is required, preferably one that can easily be ported
between various operating systems.

# Port Attester application to Windows

Apart from metadata mentioned earlier, accesses to TPM and HID interface also
are done in a way specific to the operating system. On top of that, building
applications for Windows requires different tools and procedures, which also
will be done in this task.

## Update documentation

Top level description wasn't changed since first PoC. Project evolved
significantly since then, and information located there no longer reflects
current state of the project.

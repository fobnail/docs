# Propose and document new API design based on CTAPHID

With CTAPHID used instead of CoAP, human-readable names of endpoints should be
changed to numeric IDs. CoAP status reporting also has to be modified to account
for different interface.

# Implement functions specific to Fobnail in libnitrokey

This is temporary solution to allow testing until full Fobnail support is added
to libnitrokey.

# Rewrite Attester application to be compatible with Fobnail Token changes

This task consists of supporting new communication interface. All uses of CoAP
will be removed, and API provided by libnitrokey will be used instead.

# Change the way metadata is obtained

Currently, metadata is created from SMBIOS tables and MAC address of primary
network controller, as described [here](https://fobnail.3mdeb.com/fobnail-api/#adminprovisionidmeta).
Accessing SMBIOS requires root privileges, and reliably determining proper
network interface proved to be difficult, especially with USB network adapters
or systems with no network drivers like Heads. A new way of obtaining metadata
is required, preferably one that can easily be ported between various operating
systems.

# Port Attester application to Windows

Apart from metadata mentioned earlier, accesses to TPM and HID interface also
are done in a way specific to operating system. On top of that, building
applications for Windows requires different different tools and procedures,
which also will be done in this task.

## Update documentation

Top level description wasn't changed since first PoC. Project evolved
significantly since then, and information located there no longer reflects
current state of the project.

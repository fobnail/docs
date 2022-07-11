# Summary

Change Fobnail architecture - instead of having server on the attester and
client on Fobnail Token, server would be located on Fobnail Token and client on
Attester.

# Motivation

The current architecture severely limits Fobnail Token configuration abilities
due to Fobnail Token being a client. Fobnail Token has hardcoded paths for
performing provisioning and attestation. We are not capable of sending requests
to Fobnail Token which severely limits the amount configuration possible, e.g.
an once provisioned platform can not be unprovisioned without resetting Fobnail
Token into factory state. Fobnail Token configuration cannot be changed without
resetting and going once again through the provisioning process.

It is also not possible to expose any services from Fobnail Token to Attester,
such as access to symmetric cryptographic keys or performing cryptographic
operations on behalf of attester in case of asymmetric cryptography. This is a
must for features like disk encryption or Fobnail-based authentication to
services, like VPN.

Switching to a server based Fobnail Token allows to expose services for
accessing cryptographic keys and performing administrative tasks.

# Architecture change proposal

## Fobnail Token provisioning

Currently, if in unprovisioned state, Fobnail Token initiates provisioning by
requesting PO certificate chain from Platform Owner, then it verifies PO
certificate chain against a trusted certificate embedded into Fobnail firmware.
If verification passes, Fobnail generates Identity/Encryption keypair and
requests PO to generate certificate from this key. Retrieval of valid
Identity/Encryption certificate finalizes provisioning process.

Instead we propose the following protocol:
1. Unprovisioned Fobnail token awaits for server to commence provisioning by
   sending it's certificate chain.
2. Fobnail verifies certificate chain against trusted certificate embedded into
   its firmware. If verification is succesful, Fobnail generates
   Identity/Encryption keypair and responds with Certificate Signing Request.
   Otherwise an error is returned.
3. At this point Platform Owner configures Fobnail, like default appraisal
   policy. This step is optional, if skipped, defaults will be used.
4. Platform finalizes configuration by sending certificate generated from
   previously provided CSR.
5. Fobnail Token verifies and saves certificate and provided configuration to
   its internal memory, then locks configuration and sends status to Platform
   Owner, finalizing provisioning process.
6. Token can be unlocked only by reset, which wipes out all configuration,
   provisioned platforms and all keys.

## Local platform provisioning

Platform provisioning doesn't change much, except that the process is initiated
by platform instead of Fobnail Token. In order to protect against platform
re-provisioning itself, Fobnail Token must require user consent to provision
platform, presumably by pressing button for a few seconds. As an additional
security measure, platform provisioning could be lockable through Fobnail Token
APIs.

Previously all requests were sent in a specific order, but since Fobnail Token
will be a server now, and client is free to send requests in any order, the
server must handle this case. We want to reduce the amount of global state we
keep, all API should take as parameter everything that is required to complete
operation. To achieve this we introduce objects IDs, Fobnail Token may cache
some objects in memory, like EK or AIK. Client may use object IDs to reference
these objects. Each object is temporary and is kept at most as long as client is
connected to the server. Server regularly issues CoAP ping requests to check
client connectivity.

1. Platform starts provisioning by sending it's EK certificate chain.
2. Fobnail Token verifies certificate chain and sends EK object ID in response.
3. Platform generates and sends AIK together with EK object ID.
4. Fobnail Token responds with AIK challenge and AIK object ID.
5. Platform performs Credential Activation using the provided challenge and
   sends results to Fobnail Token together with EK object ID and AIK object ID.
6. Fobnail verifies results of Credential Activation and sends status to
   the Platform. AIK is marked as trusted at this point. Fobnail Token creates
   Provisioning Context and sends its ID (PC ID) to Platform.
7. Platform signs and sends its metadata to Fobnail, together with PC ID,
   Fobnail Token verifies metadata signature against AIK bound to specified
   Provisioning Context.
8. Fobnail Token verifies metadata and returns status. Metadata is bound to the
   specified Provisioning Context.
9. Platform sends RIMs.
10. Fobnail verifies RIMs and returns status. RIMs are bound to the Provisioning
    Context.
11. At this point additional calls may be issued to send more data. This feature
    is left for future provisioning extensions.
12. Platform finalizes provisioning by sending special request.
13. Fobnail verifies all data bound to the Provisioning Context (AIK and RIMs
    currently) saves it to flash and sends final status.
14. Platform is provisioned now.

## Remote platform provisioning

TBD

## Platform attestation

Since Fobnail Token is a server, attestation must be requested by the platform.
After plugging Fobnail Token into USB, attestation must start and complete
within a defined amount of time, otherwise platform will be considered
untrustworthy.

1. Platform starts attestation by sending signed metadata to Fobnail Token.
2. Fobnail token verifies metadata and returns Attestation Context ID together
   with PCR selection and nonce.
3. Platform does TPM quote with its AIK, Fobnail Token provided PCR selection
   and nonce. Result sent to Fobnail Token, to appropriate Attestation Context.
4. Fobnail Token performs Evidence Appraisal and returns trust decision to the
   Platform.
5. Fobnail Token unlocks access to keys stored in flash.

# Fobnail Token services

Fobnail services include among others access to protected files and keys. Access
is granted only to a platform which has passed attestation. Additional, security
measures could be imposed for finer grained control where each key access would
have to be granted to each platform.

Secure files can be read, written or deleted. Primary usage for this feature is
to keep keys which should be possible to read back, e.g. disk encryption keys
(having Fobnail Token do all crypto on behalf of Platform would result in too
high performance hit).

Keys, once written can not be retrieved (neither symmetric nor asymmetric). In
case of symmetric keys Fobnail Token exposes API to encrypt and decrypt data
using these keys. In case of asymmetric keys Fobnail Token exposes API to
decrypt or sign data using a private key, and to read public key. Platform is
responsible for performing cryptographic operations on public key.

Due to Fobnail Token being exposed as a server over (local) network it could be
accessed by any unprivileged application (just with access to network stack).
To prevent any application from accessing Fobnail Token services, all requests
must be signed with AIK which limits access only to application which have
access to TPM.

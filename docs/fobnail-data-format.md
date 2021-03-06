# Fobnail data format

Fobnail uses CBOR (with a few exceptions) for transmitting structured data both
from Fobnail to Attester and from Attester to Fobnail. We use subset of
[RFC7049](https://datatracker.ietf.org/doc/html/rfc7049) (no tagging and no
extensions).

Direction of communication below is specified with regard to Fobnail Token, as
it is present both in communication with Platform Owner, as well as with
Attester.

# Input

## EK certificate chain

EK certificate chain is transferred as an ordered array of raw DER certificates.
EK certificate is sent as last, and root CA certificate isn't sent at all - it
has to be known to Fobnail in advance anyway and sending it would make the chain
bigger.

```json
{
    // Array of certificates in DER format (major 4)
    "certs": [
        // CA immediately under root (major 2)
        [ 0x30, 0x82, 0x03, 0x8b, ... ],
        // Intermediate CAs, if any
        ...
        // EK certificate (major 2)
        [ 0x30, 0x82, 0x04, 0x8f, ... ]
    ],
}
```

## AIK

AIK is transferred in TPM format (TPM2B_PUBLIC structure). Re-encoding this into
CBOR would strip essential data. Fobnail needs unmodified TPM2B_PUBLIC to
successfully perform the "make credential" operation which is needed for
credential activation.

## Metadata

Metadata is sent in CBOR-encoded format (also described
[here](https://github.com/fobnail/docs/blob/main/dev-notes/metadata.md) in more
detail). Example metadata looks like this (for readability, presented as JSON)

```json
{
    // metadata version, used to detect backward-incompatible changes
    // (major 0, unsigned integer)
    "version": 1,
    // device manufacturer extracted from SMBIOS tables (major 3, UTF-8 string)
    "manufacturer": "Gigabyte",
    // device model extracted from SMBIOS tables (major 3, UTF-8 string)
    "model": "Gigabyte A520 AORUS ELITE",
    // MAC address of primary network card (major 2, byte string)
    "mac": [0xea, 0x96, 0x91, 0x87, 0x26, 0x8d],
    // device serial number extracted from SMBIOS tables (major 3, UTF-8 string)
    "sn": "S21N559B431",
}
```

Metadata is signed by Attester using AIK before sending (see
[Data signing](#data-signing) section below).

## RIM

RIM holds PCR registers (from TPM) which are used for attestation. It doesn't
contain information about the Attester since these are part of metadata. RIM
may contain multiple PCR banks (depending of hash algorithm: SHA-1, SHA-256,
others). Each PCR bank contains bitmap of present PCRs (`pcrs` field) and array
of PCR itself. The LSB of bitmap corresponds to PCR0. RIMs are sent in CBOR:

```json
{
    // PCR update counter, incremented every time when some PCR gets updated.
    "update_ctr": 0,
    // Array of PCR banks (major 4)
    "banks": [
        // First PCR bank
        {
            // Algorithm ID as used by TPMs (TPM2_ALG_* constants from libtss)
            // (major 0, unsigned integer)
            "algo_id": 0xb,
            // Bitmap of present PCRs (major 0, unsigned integer)
            "pcrs": 0xffffffff,
            // Array of PCR registers, each register holds hash corresponding to the
            // type of bank.
            // There must be as many hashes as set bits in "pcrs" field. If
            // these don't match then RIM is considered invalid.
            "pcr": [
                // Size of hash is verified, whether all hashes in a bank have
                // the same size, if these don't match then entire RIM is
                // considered invalid
                [ 0x00, 0x00, 0x00, 0x00, ... ],
                ...
            ]
        },
        // Another PCR bank (major 5, map)
        {
            "algo_id": 0xb,
            "pcrs": 0x000000ff,
            // major 4 (array)
            "pcr": [
                // major 2 (byte string)
                [ 0x00, 0x00, 0x00, 0x00, ... ],
                ...
            ]
        }
    ],
}
```

RIM is signed by Attester using AIK before sending (see
[Data signing](#data-signing) section below).

## Platform Owner certificate chain

Platform Owner certificate chain is sent in exactly the same format as was used
for [EK certificate chain](#ek-certificate-chain).

## Fobnail identity/encryption certificate

Nothing except the certificate is sent, raw DER is used instead of CBOR.

## Data signing

Signature is generated by taking an already CBOR-encoded object, a nonce
(generated by Fobnail and sent as part of request) and signing hash of them
both. Nonce is appended to CBOR as plain bytes. Resulting signature and original
data (without nonce) is wrapped into another CBOR object:

```json
{
    // The data we signed (nested CBOR, without nonce)
    // major 2 (byte array)
    "data": [0x42, ...],
    // The signature we just computed (major 2, byte array)
    "signature": [0x20, ...]
}
```

### Exception

Result of `quote` command has nonce explicitly included in returned structure.
No additional data is appended before signing. Returned CBOR object has the same
format as above. This (along with other mechanisms) ensures freshness of Claims,
in addition to freshness of Evidence (see RATS architecture for description of
those artifacts).

# Output

## PCR selection

PCR selection is sent to Attester during attestation. Its format is similar to
that of [RIM](#rim), but stripped out of unnecessary fields. Due to the nature
of PCR selection parsing done by `TPM2_Quote()`, order of PCR banks matters.

```json
{
    // Array of PCR banks (major 4)
    "banks": [
        // First PCR bank
        {
            // ID of algorithm used by PCRs (major 0)
            "algo_id": 0x0004,
            // Bitmap of present PCRs (major 0, unsigned integer)
            "pcrs": 0xffffffff,
        },
        // Another PCR bank (major 5, map)
        {
            "algo_id": 0x000b,
            "pcrs": 0x000000ff,
        }
    ],
}
```

## Certificate signing request (CSR)

CSR is sent in raw DER format. No other data is sent along with it so it isn't
packed into CBOR.

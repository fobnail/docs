# Fobnail API

Fobnail Token exposes API over
[CoAP](https://datatracker.ietf.org/doc/html/rfc7252) server exposed over UDP.

## General considerations

This document describes API (major) version 1 and all API endpoint addresses are
relative to (starting with) `/api/v1`. Version will be increased only when
backwards-incompatible changes are done.

### General request handling rules

Some APIs create temporary objects that live at most as long as client is
connected. To check connection status server issues CoAP Ping requests which
clients **must** respond to, to avoid losing theirs objects.

When an object has been created, server responds with **2.01**, `Location-Path`
specifies path to the resource (a numeric object ID). Object IDs are specific to
a client - client cannot access object IDs of other clients. `Location-Path` is
a string option whereas object ID is an integer so it must be encoded as a
string. Object IDs may be passed as part of CBOR payload (as integer) or in URI
(as string).

If `Content-Format` option is not present Fobnail Token assumes that
`Content-Format` is `application/octet-stream`. All requests which pass
CBOR-encoded data **must** set `Content-Format` option to `application/cbor` for
Fobnail Token to interpret the payload as CBOR. If unknown or invalid
`Content-Format` is given, server responds with **4.00** (Bad Request). If
passing CBOR-encoded data which is improperly encoded or not valid (e.g.
required fields missing) server responds with **4.00**. Passing
`application/cbor` to an endpoint that expects `application/octet-stream` is
invalid and will result in **4.00** response.

Client may sent one or more `Accept` options which **must** be handled by the
server. If the server can not respond in the format requested by `Accept`
options it **must** respond with **4.06** (Not Acceptable).

[Conditional Request Options](https://datatracker.ietf.org/doc/html/rfc7252#section-5.10.8)
are **not** supported and will result in **4.02** (Bad Option). Payload should
contain human-readable message describing what options are not supported. See
[RFC7252 section 5.4.1](https://datatracker.ietf.org/doc/html/rfc7252#section-5.4.1)
for details.

Each response from the Fobnail Token **must** contain `Content-Format` option
unless it is an error response with a diagnostic payload. See
[RFC7252](https://datatracker.ietf.org/doc/html/rfc7252#section-5.5.1) for
details.

### Error responses

Fobnail Token may return error codes as defined by CoAP specification. An error
response **may** contain additional context sent as payload. All cacheable error
responses **must** have `Max-Age` set to 0.

Error response must not contain `Content-Format` option. Message (if present)
must be passed as a raw UTF-8 string.

### Fobnail Data format

Fobnail uses CBOR (with a few exceptions) for transmitting structured data both
from Fobnail to Attester and from Attester to Fobnail. We use subset of
[RFC7049](https://datatracker.ietf.org/doc/html/rfc7049) (no tagging and no
extensions).

Clients are **required** to set `Content-Format` to `Application/CBOR` when
transmitting CBOR-encoded data.

### Data signing

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

#### Exception

Result of `quote` command has nonce explicitly included in returned structure.
No additional data is appended before signing. Returned CBOR object has the same
format as above. This (along with other mechanisms) ensures freshness of Claims,
in addition to freshness of Evidence (see RATS architecture for description of
those artifacts).

## API endpoints

### Fobnail Token provisioning

| Endpoint Name             | Method | Arguments                               |
| ------------------------- | ------ | --------------------------------------- |
| /admin/token_provision    | POST   | PO certificate chain                    |
| /admin/provision_complete | POST   | Fobnail Identity/Encryption certificate |

These APIs may be executed only when Fobnail Token is in a unprovisioned state.
Fobnail Token **must** respond with **4.03** if these APIs are called after
provisioning is complete.

#### /admin/token_provision

PO certificate chain is transferred as an ordered array of raw DER certificates,
starting with the certificate directly after the root certificate (root is
already known by the Fobnail Token).

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

Fobnail Token verifies certificate chain against the trust anchor embedded into
its firmware. If verification fails, **4.03** is returned.

On success, Fobnail generates Identity/Encryption key, prepares Certificate
Signing Request and responds with **2.01** status code, response **must**
contain CSR as a payload. CSR is sent in DER format and `Content-Format`
**must** be set to `application/octet-stream`.

#### /admin/provision_complete

Client sends a certificate generated from CSR previously provided by
`/admin/token_provision`. Fobnail Token verifies correctess of the provided
certificate, such as whether it is properly signed, whether it has correct
extensions defined, etc. On error **4.03** is returned.

On success **2.01** is returned  provisioning is complete.

Certificate is sent as a raw DER and `Content-Format` must be set to
`application/octet-stream` or the request will be rejected. Future API versions
may accept `application/cbor`.

### Platform provisioning

| Endpoint Name              | Method | Arguments                        |
| -------------------------- | ------ | -------------------------------- |
| /admin/provision/ek        | POST   | EK certificate chain             |
| /admin/provision/aik       | POST   | AIK public part                  |
| /admin/provision           | POST   | Credential Activation result     |
| /admin/provision/{id}/meta | POST   | Platform metadata                |
| /admin/provision/{id}/rim  | POST   | Reference Integrity Measurements |
| /admin/provision/{id}      | POST   | None                             |

#### /admin/provision/ek

EK certificate chain is transferred in the same format as PO certificate chain
described above. The certificate directly after the root certificate is the
first certificate in array while EK certificate is the last.

Client issues POST request with `Content-Format` set to `application/cbor` and
payload containing CBOR-encoded certificate array as described above. Fobnail
Token verifies provided certificate chain against one of roots embedded into
firmware. On success **2.01** response is returned, `Location-Path` contains EK
object ID. If EK certificate chain verification fails **4.03** is returned.

#### /admin/provision/aik

AIK is transferred in TPM format (TPM2B_PUBLIC structure). Re-encoding this into
CBOR would strip essential data. Fobnail needs unmodified TPM2B_PUBLIC to
successfully perform the "make credential" operation which is needed for
credential activation. Unmodified AIK and EK object ID are wrapped into CBOR:

```json
{
    // AIK (TPM2B_PUBLIC, major 2)
    "aik": [0x20, 0x01, 0x00, ...],
    // EK object ID (major 0 - unsigned integer)
    "ek": 1
}
```

Fobnail Token verifies whether AIK is a valid TPM2B_PUBLIC and whether key
algorithm is supported. On success **2.01** response is returned and
`Location-Path` contains AIK object ID. Response payload contains a challenge
(Make Credential) prepared using the provided AIK and EK.

```json
{
    "idObject": [0x20, 0x02, 0x10, ...],
    "encSecret": [0x20, 0x02, 0x10, ...],
}
```

If AIK verification fails **4.03** is returned, payload may contain additional
context. If provided EK object ID is invalid **4.04** is returned.

#### /admin/provision

Client calls this endpoint to create a Provisioning Context. Client provides
result of Credential Activation as payload:

```json
{
    // EK object ID of the EK used to make the challenge (major 0)
    "ek": 1,
    // AIK object ID of the AIK used to make the challenge (major 0)
    "aik": 2,
    // Decrypted secret - output of TPM Credential Activation (major 2)
    "secret": [0x00, 0x01, 0x02, 0x03, ...]
}
```

Fobnail Token verifies whether the decrypted secret matches with the secret kept
in Fobnail Token's memory. If verification is successful Fobnail Token creates a
Provisioning Context and responds with **2.01** response (`Location-Path`
contains Provisioning Context ID).

If secret does not match **4.03** is returned. If either EK object ID or AIK
object ID is invalid **4.04** is returned.

#### /admin/provision/{id}/meta

This method is used to upload platform metadata. After upload, the platform
metadata is bound to the provisioning context specified by `{id}`.

Metadata is sent in CBOR-encoded format (also described
[here](https://github.com/fobnail/docs/blob/main/dev-notes/metadata.md) in more
detail). Example metadata looks like this

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

Client **must** sign metadata with AIK (see [Data signing](#data-signing)
section below).

Fobnail Token verifies the provided metadata against AIK bound to the
Provisioning Context, if signature verification fails **4.03** is returned, if
metadata format is invalid **4.00** is returned. On success either **2.01** or
**2.04**. On the first successful call **2.01** is returned (`Location-Path`
must not be present), on subsequent calls **2.04** is returned to signify that
metadata has been replaced.

#### /admin/provision/{id}/rim

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

Client **must** sign RIM with AIK (see [Data signing](#data-signing) section
below).

Fobnail Token verifies RIM signature and format, if signature verification fails
**4.03** is returned, if RIM has invalid format **4.00** is returned. On success
**2.01** or **2.04** is returned. On the first successful call **2.01** is
returned (`Location-Path` must not be present), on subsequent calls **2.04** is
returned to signify that the previous RIM has been replaced with the new RIM.

#### /admin/provision/{id}

POST request to this endpoint results in writing data attached to the
Provisioning Context into persistent storage. Currently, this request does not
contain any data and payload **must** be empty or **4.00** response will be
returned.

Fobnail Token verifies whether all required data has been provided (metadata and
RIM is mandatory). On error, **4.03** response is returned. If verification is
successful Fobnail writes all required data into persistent storage, completing
the provisioning process. If an error occurs while writing into persistent
storage **5.01** is returned, payload may contain additional context. On success
**2.04** response is returned to signify provisioning is complete. At this point
the Provisioning Context becomes invalid and all subsequent requests to the
Provisioning Context will return **4.04**.

### Attestation

| Endpoint Name             | Method | Arguments            |
| ------------------------- | ------ | -------------------- |
| /attest                   | POST   | Platform metadata    |
| /attest/{id}              | POST   | Output of TPM quote  |

#### /attest

Client sends platform metadata in the same format as when calling
`/admin/provision/{id}/meta`. Metadata **must** be signed by signed by the same
AIK that was used during platform provisioning. If verification is successful
**2.01** is returned (otherwise **4.04**) and `Location-Path` contains
Attestation Context ID. Payload contains PCR selection and nonce:

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
    // Nonce to be passed to TPM_Quote() (major 2)
    "nonce": [0x00, 0x01, 0x02, 0x03, ...]
}
```

PCR selection format is similar to that of [RIM](#rim), but stripped out of
unnecessary fields. Due to the nature of PCR selection parsing done by
`TPM2_Quote()`, order of PCR banks matters.

### /attest/{id}

Client sends the evidence to this endpoint (the result of TPM Quote). Based on
the evidence, Fobnail Token decides whether platform is trustworthy or not. If
Fobnail Token decides that the platform is trustworthy **2.04** response is
returned, **4.03** otherwise.

If attestation is successful Fobnail Token unlocks access to the Fobnail Token
Services.

### Fobnail Token Services

Fobnail Token Services are available after succesful platform attestation.
Fobnail can store cryptographic keys (or other stuff) in its internal flash,
depending on key type (symmetric or asymmetric) and usage permissions various
operations are available. For symmetric keys it is encryption and decryption.
For asymmetric keys it is signing, decryption, KDF and public key read (Host
extracts public key and does encryption on its own).

Key, once is created, may not be read (except for public key). Fobnail Token
performs cryptographic operations on behalf of Platform, which may be slow. If
this behaviour is undesired, Secure Storage may be used instead, to store key as
a file inside Fobnail Token.

| Endpoint Name        | Method | Arguments                        |
| -------------------- | ------ | -------------------------------- |
| /crypto              | POST   | Key name, type, etc.             |
| /crypto/{id}/type    | GET    | None                             |
| /crypto/{id}/encrypt | POST   | Data to encrypt                  |
| /crypto/{id}/decrypt | POST   | Data to decrypt                  |
| /crypto/{id}/sign    | PUT    | Data to sign                     |
| /crypto/{id}/pub     | GET    | None                             |
| /storage/fs/{name}   | GET    | File name                        |
| /storage/fs/{name}   | PUT    | File name and file contents      |
| /storage/fs/{name}   | DELETE | File name                        |

If platform has not been attested (or failed attestation) these APIs will return
**4.03** error, otherwise Fobnail Token checks whether a specified key exists
and whether Platform has access to that key. If key or file does not exists or
there is no access **4.04** error is returned.

> Note: currently, crypto API is not fully defined. Future versions of this
> documents will address this problem.

### GET /storage/fs/{name}

Client sends a request to this endpoint to read a file. File path is encoded as
part of URI. If a file exists (and if it's accessible) **2.05** response is sent
with file contans as the payload. `Max-Age` option **must** be set to 0 to
prevent caching. ETag option **must not** be present in response and if present
in request it **must** be ignored.

> GET method always returns full contents of the file, further versions of this
> document may define API using FETCH method from RFC8132.

Fobnail Token attempts to open (or create) the requested file and returns
**2.01** response with file ID which is used as a handle to access the file.
**4.04** is returned if the file does not exists or there is no access.

### PUT /storage/fs/{name}

This endpoint is used to write file contents. Fobnail Token attempts to write
the specified file (URI) with the provided payload. If successful, Fobnail Token
returns either **2.01** (when the file has been created) or **2.04** (when the
file has been updated). Request may fail in the following situations:

- If name contains invalid characters (name may not contain NULLs and slash) -
  **4.03** error.
- If there is no access to file - **4.03** error.
- If writing the file failed for any reason - **5.00** error (payload may
  contain additional context).

### DELETE /storage/fs/{name}

Used to delete a file. File name is provided in URI and payload should be empty.
On success, or if the file didn't exist before **2.02** is returned. On error,
such as when the file does exist, but there is no permission to remove it
**4.03** is returned.
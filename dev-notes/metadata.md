# Platform metadata

Platform metadata contains information that uniquely identify the platform being
attested. Metadata is composed of

- Device manufacturer and model name
- MAC address
- Serial number

## Device manufacturer and model name

Human-readable device name as provided by firmware. On x86 this information
should be fetched from SMBIOS `System Information/Manufacturer` and
`System Information/Product Name`.

## MAC address

MAC address of the primary network adapter. Preferably onboard ethernet adapter
or Wi-Fi adapter in case there is no ethernet adapter. If there are multiple
network adapters of the same type, MAC of the adapter with the lowest PCI BDF
address should be taken. Let's consider the following case:

```shell
$ lspci
01:04.00 Network adapter 1
02:00.01 Network adapter 2
01:03.02 Network adapter 3
01:03.00 Network adapter 4
```

In that case we would select `01:03.00`.

## EK certificate hash

SHA-256 hash of TPM EK certificate, this is not sent as part of metadata
structure - EK certificate is already known to Fobnail token, and Fobnail token
can compute hash on its own. Sole purpose of EK hash is to serve as an
additional unique value when other data is not unique enough.

## Metadata binary format

Metadata is transferred in a CBOR-encoded format. Example metadata looks like
this (for readability, presented as JSON).

```jsonc
{
    // metadata version, used to detect backward-incompatible changes
    "version": 1,
    "manufacturer": "Gigabyte",
    "model": "Gigabyte A520 AORUS ELITE",
    "mac": [0xea, 0x96, 0x91, 0x87, 0x26, 0x8d],
    "sn": "S21N559B431",
}
```

Signature is generated from CBOR blob and added to metadata:

```jsonc
{
    // Metadata encoded in CBOR
    "encoded_metadata": [0x42, ...],
    // Signature created from "encoded_metadata"
    "signature": [0x20, ...],
}
```

This structure is again encoded with CBOR and sent to Fobnail token.

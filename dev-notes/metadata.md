# Platform metadata

Platform metadata contains information that uniquely identify the platform being
attested. Metadata is composed of

- Device model name
- MAC address
- CPU Serial number
- EK certificate hash

## Device model name

Human-readable device name as provided by firmware, usually manufacturer and
device name.

## MAC address

MAC address of the primary network adapter. Preferably onboard ethernet adapter
or Wi-Fi adapter in case there is no ethernet adapter.

## EK certificate hash

TBD

## Metadata binary format

Metadata is transferred in a CBOR-encoded format. Example metadata looks like
this

```json
{
    // metadata version, used to detect backward-incompatible changes
    "version": 1,
    "mac": [0xea, 0x96, 0x91, 0x87, 0x26, 0x8d],
    "sn": [],
}
```

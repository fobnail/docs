# Research of EK certificate chain sizes

This document combines results gathered from [this issue](https://github.com/fobnail/docs/issues/50).
It was created to set expectations about size of data that has to be sent to and
parsed by Fobnail Token.

## SLB 9665TT2.0 TPM

Used on PC Engines apu platforms:

```text
Certificate 0 size: 1177 bytes
Certificate 1 size: 1463 bytes
Certificate 2 size: 1455 bytes
Certificate is self-signed, assuming it is root

Chain length: 3
Total chain size: 4095 bytes
Size of the biggest certificate: 1463 bytes
```

## SLB 9670

Used on Raspberry Pi. Two different specimens had slightly different chains:

```text
Certificate 0 size: 1422 bytes
Certificate 1 size: 1705 bytes
Certificate 2 size: 1455 bytes
Certificate is self-signed, assuming it is root

Chain length: 3
Total chain size: 4582 bytes
Size of the biggest certificate: 1705 bytes
```

```text
Certificate 0 size: 1171 bytes
Certificate 1 size: 1449 bytes
Certificate 2 size: 1455 bytes
Certificate is self-signed, assuming it is root

Chain length: 3
Total chain size: 4075 bytes
Size of the biggest certificate: 1455 bytes
```

Same TPM on NovaCustom NS5x/7x and Protectli VP4630/50/70 platforms:

```text
Certificate 0 size: 1171 bytes
Certificate 1 size: 1449 bytes
Certificate 2 size: 1455 bytes
Certificate is self-signed, assuming it is root

Chain length: 3
Total chain size: 4075 bytes
Size of the biggest certificate: 1455 bytes
```

## Unknown TPM (probably also SLB 9670)

Found in NovaCustom NV4X laptops:

```text
Certificate 0 size: 1171 bytes
Certificate 1 size: 1449 bytes
Certificate 2 size: 1455 bytes
Certificate is self-signed, assuming it is root

Chain length: 3
Total chain size: 4075 bytes
Size of the biggest certificate: 1455 bytes
```

## Unknown TPM

[Reported by 0xDen](https://github.com/fobnail/docs/issues/50#issuecomment-1484607790)

```text
Certificate 0 size: 1184 bytes
Certificate 1 size: 1463 bytes
Certificate 2 size: 1455 bytes
Certificate is self-signed, assuming it is root

Chain length: 3
Total chain size: 4102 bytes
Size of the biggest certificate: 1463 bytes
```

## fTPM found in AMD processors

These TPMs don't have EK certificates.

## Summary

All encountered EK certificate chains have the same length - 3 certificates
including EK and root. Total size of chain oscillates around 4KB. Certificate
sizes are similar across all collected data, with EK being slightly smaller than
root and intermediate certificates. Biggest certificate in chain usually has
around 1460 bytes, but there was one TPM with significantly larger maximal size
of certificate - 1705 bytes.

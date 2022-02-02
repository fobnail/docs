# Trussed problems

This document describes problems encountered when integrating Trussed into
Fobnail firmware.

## Massive amount of unnecessary dependencies

Trussed pulls a massive amount of dependencies (mostly related software
cryptography). These dependencies are redundant as we are going to use hardware
crypto module, also they greatly increase firmware size and complexity: after
adding Trussed firmware size increased from ~400 KiB to ~1090 KiB exceeding
internal NOR flash capacity, we had to enable code optimization and LTO (after
doing so, size dropped to ~700 KiB).

## Software cryptography

Trussed provides something that is called a mechanism. Mechanism is like an
interface - it provides a set of methods that we can call using IPC. Trussed
provides default mechanism implementations which are a purely software based
cryptography.

## Lack of fine grained feature control

It is not possible to control which Trussed features should be enabled. Trussed
does have feature flags, but these flags are useless as disabling any feature
causes Trussed build to fail in complex and hard to fix ways.

## Lack of isolation implementation

Trussed by itself does not provide any isolation between the secure and
non-secure world. It only provides services and IPC interface to access these
services. Isolation has to be somehow else, but there are neither examples on
how do that nor documentation. Nitrokey firmware uses Trussed but does not
isolate it, such a setup Trussed doesn't provide any real benefits.

## Inconsistent APIs, lack of documentation

TBD

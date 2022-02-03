# Trussed problems

This document describes problems encountered when integrating Trussed into
Fobnail firmware.

## Massive amount of unnecessary dependencies

Trussed pulls a massive amount of dependencies (mostly related software
cryptography). These dependencies are redundant as we are going to use a
hardware crypto module. Also, they greatly increase firmware size and
complexity: after adding Trussed firmware size increased from ~400 KiB to
~1090 KiB exceeding internal NOR flash capacity, we had to enable code
optimization and LTO (after doing so, size dropped to ~700 KiB).

## Software cryptography

Trussed provides something that is called a mechanism. Mechanism is like an
interface - it provides a set of methods that we can call using IPC. Trussed
provides default mechanism implementations that are purely software-based
cryptography.

## Lack of fine-grained feature control

It is not possible to control which Trussed features should be enabled. Trussed
does have feature flags, but these flags are useless as disabling any feature
causes Trussed build to fail in complex and hard-to-fix ways.

## Lack of isolation implementation

Trussed by itself does not provide any isolation between the secure and
non-secure world. It only provides services and an IPC interface to access these
services. Isolation has to be using something else, but there are neither
examples of that nor documentation. Nitrokey firmware uses Trussed but does not
isolate it. In such a setup, Trussed doesn't provide any real benefits.

## Low quality and inconsistent APIs

There are many low-quality APIs, e.g., some APIs will panic if fed invalid
data instead of returning an error. APIs are inconsistent, e.g., some APIs
accept a reference to a byte array (`&[u8]`), while others, like file system
APIs, consume `Bytes<N>` object, which is a constant-size stack-allocated
vector. Trussed APIs need to know the size of that vector, so APIs require
objects like `Bytes<MAX_MESSAGE_LENGTH>`. This leads to 2 types of problems:

- data size may not cross `MAX_MESSAGE_LENGTH` - since file system APIs accept
  `Bytes<MAX_MESSAGE_LENGTH>` we cannot create files larger than
  `MAX_MESSAGE_LENGTH` (1024) bytes.

- We always allocate `MAX_MESSAGE_LENGTH` bytes - even if we want to create an
  empty file.

## Lack of RSA support

Trussed does not provide any mechanisms for performing RSA operations. Only
ED25519 is supported, but we cannot use it because it's not supported by TPMs.
Due to lack of documentation, it is unclear how a custom RSA mechanism could
be implemented.

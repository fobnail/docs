# Ethernet over USB research

## Protocol research

Therer are 4 protocols implementing Ethernet-over-USB:
- RNDIS - Microsoft's proprietary protocol, not covered here as there are open
  alternatives

- ECM - Ethernet Control Module
- EEM - Ethernet Emulation Model
- NCM - Network Control Model

ECM, EEM and NCM protocols are USB-IF managed open standards (part of USB CDC).

ECM is the oldest and according to
[Wikipedia article](https://en.wikipedia.org/wiki/Ethernet_over_USB) the
simplest protocol of these three, its main advantage is simplicity
- it has 2 communication channels: first channel is used for transferring
  control messages like link status, etc.
- requires 3 endpoints: 1 for control transfers, 1 bulk in and 1 bulk out
- second channel is used for sending raw, unmangled Ethernet frames.

Main disadvantage of this protocol is its poor performance, maximum packet size
is 64 bytes, Ethernet packets larger than that require few USB transfers.
Every Ethernet frame must be send in single USB transfer (1 USB transfer may
consist of many pakcets), it is not possible to bundle many Ethernet frames into
one USB transfer.

EEM protocol improves performance while still being relatively. EEM brings
following improvements
- no more 64 byte limit, max packet size for USB 2.0 is usally 512 bytes
- small Ethernet frames may be packed into single USB packet, thus improving
  performance
- CDC EEM 1.0 Specification, section 2 - Management Overview
  > Unlike CDC ECM, EEM does not extend an interface across the USB bus but
  > instead considers the USB bus to be a vehicle for moving Ethernet packets. 
- designed for local-only connection over USB emulated Ethernet,
  where ECM is designed for connecting to real network over Ethernet-to-USB
  modem.
- Requires only 2 endpoints (bulk in and bulk out)


NCM is the latest protocol designed for high throughput, like ECM it is designed
to control hardware with a real, physical Ethernet link, it is designed for
controlling modern, complex hardware, thus protocol itself is very complex.

I consider EEM protocol to be the best choice
- it is designed for what we need - emulated Ethernet with no physical layer
- better performance
- ability to pack multiple Ethernet frames into one USB transfer reduces amount
  of time the driver needs to spend handling interrupts, thus offloading CPU
- EMM requires only 2 endpoints so more endpoints are available for other
  potential protocols we may implement in the future

## Ethernet implementation in software

USB driver for NRF52 is already provided by [nrf-hal](https://github.com/nrf-rs/nrf-hal)
project.

Implementations of EEM protocol exist, but these are mostly written in C. I
haven't found a single Rust implentation of EEM protocol (nor ECM), so a custom
implementation is needed.

Besides EEM driver we need an actual Ethernet driver, additionally we may need
IP + UDP support for CoAP protocol, this may not necessarily be a requirement as
CoAP protocol (used by CHARRA) doesn't seem to depend on UDP or any other
transport layer, yet implementations do depend and removing that dependence
could require more work than getting UDP support.


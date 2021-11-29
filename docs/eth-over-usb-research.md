# Ethernet over USB research

## Protocol research

There are 4 protocols implementing Ethernet-over-USB:

- RNDIS - Microsoft's proprietary protocol, not covered here as there are open
  alternatives
- ECM - Ethernet Control Module
- EEM - Ethernet Emulation Model
- NCM - Network Control Model

ECM, EEM and NCM protocols are USB-IF managed open standards (part of USB CDC).

ECM is the oldest one, and according to
[Wikipedia article](https://en.wikipedia.org/wiki/Ethernet_over_USB) the
simplest protocol of these three, its main advantage is simplicity
- it has 2 communication channels: first channel is used for transferring
  control messages like link status, etc.
- requires 3 endpoints: 1 for control transfers, 1 bulk in, and 1 bulk out
- second channel is used for sending raw, unmangled Ethernet frames.

The main disadvantage of this protocol is its poor performance, maximum packet
size is 64 bytes, Ethernet packets larger than that require few USB transfers.
Every Ethernet frame must be sent in a single USB transfer (1 USB transfer may
consist of many packets), it is not possible to bundle many Ethernet frames into
one USB transfer.

EEM protocol improves performance while still being relatively simple. EEM
brings following improvements
- no more 64 byte limit, can have any max packet size (usually 512 bytes for
  USB 2.0)
- small Ethernet frames may be packed into a single USB packet, thus improving
  performance
- CDC EEM 1.0 Specification, section 2 - Management Overview
  > Unlike CDC ECM, EEM does not extend an interface across the USB bus but
  > instead considers the USB bus to be a vehicle for moving Ethernet packets. 
- designed for local-only connection over USB emulated Ethernet,
  where ECM is designed for connecting to a real network over Ethernet-to-USB
  modem.
- Requires only 2 endpoints (bulk in and bulk out)


NCM is the latest protocol designed for high throughput, like ECM it is designed
to control hardware with a real, physical Ethernet link, it is designed for
controlling modern, complex hardware, thus protocol itself is very complex.

I consider EEM protocol to be the best choice
- it is designed for what we need - emulated Ethernet with no physical layer
- better performance than ECM
- ability to pack multiple Ethernet frames into one USB transfer reduces the
  amount of time the driver needs to spend handling interrupts, thus offloading
  CPU
- EMM requires only 2 endpoints, so more endpoints are available for other
  potential protocols we may implement in the future

## Ethernet implementation in software

USB driver for NRF52 is already provided by [nrf-hal](https://github.com/nrf-rs/nrf-hal)
project.

Implementations of EEM protocol exist, but these are mostly written in C. I
haven't found a single Rust implementation of EEM protocol (nor ECM), so a
custom implementation is needed.

Besides the EEM driver, we need an actual Ethernet driver, additionally we may
need IP + UDP support for CoAP protocol, this may not necessarily be a
requirement as CoAP protocol (used by CHARRA) doesn't seem to depend on UDP or
any other transport layer, yet implementations do depend and removing that
dependence could require more work than getting UDP support.

There is one particularly interesting project called
[smoltcp](https://github.com/smoltcp-rs/smoltcp), it is a library providing
implementation of TCP/IP stack. These are the most important features:
- provides support for IP + TCP + UDP and others
- support for individual protocols may be disabled at compile time so we can
  remove unneeded features like TCP (we can rely on UDP since USB already
  provides reliable transmission)
- no unsafe code inside library

Looks like this is the only TCP/IP stack written in Rust and compatible with
`no_std` environments.

# Implementing Ethernet-over-USB

This document describes various stuff regarding implementation of
Ethernet-over-USB. Before commencing implementation we have conducted research
(see [eth-over-usb-research.md](eth-over-usb-research.md)) for more information.
To sum up, thats what we have decided:

- we run on bare-metal using `nrf-hal`
- we use EEM protocol for Ethernet emulation and `smoltcp` as TCP/IP stack

## Used crates

We are using following crates

- `nrf-hal` - provides things related directly to nRF52840 hardware, like
  register definitions, interrupt handling support, for some peripherals
  provides higher-level API (Timers, GPIO, ...)

- `nrf-usb` - provides implementation of `UsbBus` trait (from `usb-device`
  crate)

- `usb-device` - high level USB driver, on top of this we are building EMM
  driver

## Encountered problems

During implementation of EEM driver we have encountered following problems

- `nrf-usb` does not provide interrupt support, we have worked around this
  problem by continuously polling USB, for testing purposes it is good enough
  but ideally we should use interrupt-driven model.

- `nrf-usb` underpowers DMA - during DMA transfers CPU busy-waits in a loop
  till transfer is complete
  (see [here](https://github.com/nrf-rs/nrf-usbd/blob/main/src/usbd.rs#L437))

- `usb-device` API limitation - we cannot read single packet into two
  non-contiguous memory regions, because of that we cannot use ring buffer, as a
  temporary workaround we implemented a simple linear buffer where data is
  appended to the tail, removing data from the front requires moving of large
  amounts of memory which is very suboptimal. The same problem affects writes.

- seems like `nrf-hal` provides no high level API for setting periodic timers
  with automatically resetting counter (does the hardware support it?), counter
  must be manually reset on each interrupt handler invocation.

- integration of smoltcp may be quite hard to do efficiently - smoltcp uses
  producer consumer model, when reading data from Ethernet smoltcp calls
  `consume()` method providing a callback that is called when next Ethernet
  frame is ready, so we have to efficiently transfer Ethernet frame from USB
  thread to smoltcp thread, we want to avoid copying memory around as much as
  possible.

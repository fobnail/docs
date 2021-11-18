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

- smoltcp callback accepts a pointer to a continuous buffer, so we can't use
  ring buffer anyway

- seems like `nrf-hal` provides no high level API for setting periodic timers
  with automatic counter reset (does the hardware support it?), counter must be
  manually reset on each interrupt handler invocation, or interrupt will keep
  firing instantly one after another.

- integration of smoltcp may be quite hard to do efficiently - smoltcp uses
  producer consumer model, when reading data from Ethernet smoltcp calls
  `consume()` method providing a callback that is called when next Ethernet
  frame is ready, so we have to efficiently transfer Ethernet frame from USB
  thread to smoltcp thread, we want to avoid copying memory around as much as
  possible.

- USB driver tends to hang if attempting to write packets into full buffer. When
  we send USB packets to the host we call `usb_device::EndpointIn::write` method
  which transfers packet to USBD internal memory, packet is actually sent when
  host polls the endpoint. If USBD memory is full
  `usb_device::EndpointIn::write` returns `WouldBlock` error. Now calling
  `write()` a few times more causes the driver to deadlock forever (TX only, RX
  still works). To workaround this we must track state TX buffer ourselves: when
  we get `WouldBlock` we freeze any further transfers until USBD signals us that
  previous packet was sent.

- Device is enumerated as a full-speed device instead of high-speed, because of
  that we cannot use max packet size larger than 64 bytes.

## Status of current implementation

We have a working implementation with an echo service exposed over UDP port
`9400`. Fobnail has IP address `169.254.0.1/16`. Host must have a manually
assigned IP address as there is no DHCP. EEM implementation has few limitations
which aren't a big problem for a PoC, but should eventually be solved.

- EEM driver cannot handle EEM packets exceeding internal buffer size (driver
  will panic).
- EEM driver will panic when encounters an invalid EEM packet.
- because of problems described above we have to use a linear FIFO (instead of
  ring), maybe we could somehow optimize it?
- driver needs more testing.
- Linux's NetworkManager tries to configure EEM network interface over DHCP
  which fails. Then tries doing it again, showing every time an error message
  and breaking internet connection (about every half minute). This is a Network
  Managers's bug, but it affects Fobnail users. You can workaround this by
  appending following lines to `/etc/NetworkManager/NetworkManager.conf`
  ```ini
  [keyfile]
  unmanaged-devices=interface-name:usb0
  ```

## Testing

If you environment is properly set up, app can be flashed by simply running

```shell
$ cargo embed
```

`cargo embed` spawn RTT console used for debugging. On host you should see new
network interface.

```shell
$ ifconfig -a usb0
usb0: flags=4098<BROADCAST,MULTICAST>  mtu 1500
        ether 86:7e:55:2c:a7:b5  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

Assign IP address to `usb0`, Fobnail token is at `169.254.0.1`

```shell
$ ifconfig usb0 169.254.0.2
```

Fobnail should start printing debug information on RTT console. Note that these
errors are caused by host sending IPv6 packets. IPv6 support is disabled in
`smoltcp`.

```
16:51:15.107 TRACE smoltcp::iface::neighbor > filled 169.254.0.8 => ae-89-60-61-ca-4f (was empty)
16:51:15.133 DEBUG smoltcp::iface::ethernet > cannot process ingress packet: unrecognized packet
16:51:15.133 DEBUG smoltcp::iface::ethernet > packet dump follows:
16:51:15.133 EthernetII src=ae-89-60-61-ca-4f dst=33-33-00-00-00-16 type=IPv6
16:51:15.133 ERROR fobnail_poc > smoltcp error: unrecognized packet
16:51:15.224 DEBUG smoltcp::iface::ethernet > cannot process ingress packet: unrecognized packet
16:51:15.224 DEBUG smoltcp::iface::ethernet > packet dump follows:
16:51:15.224 EthernetII src=ae-89-60-61-ca-4f dst=33-33-00-00-00-16 type=IPv6
16:51:15.224 ERROR fobnail_poc > smoltcp error: unrecognized packet
16:51:15.691 DEBUG smoltcp::iface::ethernet > cannot process ingress packet: unrecognized packet
16:51:15.691 DEBUG smoltcp::iface::ethernet > packet dump follows:
16:51:15.691 EthernetII src=ae-89-60-61-ca-4f dst=33-33-ff-61-ca-4f type=IPv6
16:51:15.691 ERROR fobnail_poc > smoltcp error: unrecognized packet
16:51:16.669 DEBUG smoltcp::iface::ethernet > cannot process ingress packet: unrecognized packet
16:51:16.669 DEBUG smoltcp::iface::ethernet > packet dump follows:
16:51:16.669 EthernetII src=ae-89-60-61-ca-4f dst=33-33-00-00-00-16 type=IPv6
16:51:16.669 ERROR fobnail_poc > smoltcp error: unrecognized packet
16:51:16.697 DEBUG smoltcp::iface::ethernet > cannot process ingress packet: unrecognized packet
16:51:16.697 DEBUG smoltcp::iface::ethernet > packet dump follows:
16:51:16.697 EthernetII src=ae-89-60-61-ca-4f dst=33-33-00-00-00-02 type=IPv6
16:51:16.697 ERROR fobnail_poc > smoltcp error: unrecognized packet
```

Fobnail responds to ICMP echo requests:

```shell
$ ping 169.254.0.1
PING 169.254.0.1 (169.254.0.1) 56(84) bytes of data.
64 bytes from 169.254.0.1: icmp_seq=1 ttl=64 time=24.7 ms
64 bytes from 169.254.0.1: icmp_seq=2 ttl=64 time=23.1 ms
64 bytes from 169.254.0.1: icmp_seq=3 ttl=64 time=22.2 ms
64 bytes from 169.254.0.1: icmp_seq=4 ttl=64 time=21.4 ms
64 bytes from 169.254.0.1: icmp_seq=5 ttl=64 time=30.3 ms
64 bytes from 169.254.0.1: icmp_seq=6 ttl=64 time=29.4 ms
64 bytes from 169.254.0.1: icmp_seq=7 ttl=64 time=28.2 ms
```

Fobnail has an echo service exposed on UDP port `9400`, when you line of text it
will be echoed back.

```shell
$ nc -u 169.254.0.1 9400
test
test
1234567890
1234567890
```

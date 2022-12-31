# Developing firmware on PC

Fobnail firmware can be built to run either on nRF52840 or Linux. The latter
option is more suitable for development as it does not need any hardware except
a development machine.

## Obtaining firmware source code

You can obtain source code from our
[GitHub repository](https://github.com/fobnail/fobnail).

```shell
$ git clone https://github.com/fobnail/fobnail --recurse-submodules
```

## Building and running firmware locally

Make sure you have the latest Fobnail SDK installed and network setup properly.
See [Environment setup](environment.md) for instructions. For firmware to work
properly you need the `fobnail0` network interface configured.

To build and run firmware execute the following commands
(from fobnail directory).

```shell
$ env FOBNAIL_LOG=trace ./build.sh -t pc --run
    Finished dev [unoptimized + debuginfo] target(s) in 0.60s
     Running `target/x86_64-unknown-linux-gnu/debug/fobnail-poc`
 DEBUG fobnail > UDP socket initialized
 TRACE smoltcp::socket::set > [0]: adding
```

> FOBNAIL_LOG environment variable sets log level. The possible log levels are:
> error, warning, info, debug, trace.


When Fobnail firmware is running, `fobnail0` interface should be up.

```shell
$ ip addr show dev fobnail0
19: fobnail0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 0a:d5:59:6a:06:77 brd ff:ff:ff:ff:ff:ff
    inet 169.254.0.8/16 scope global fobnail0
       valid_lft forever preferred_lft forever
    inet6 fe80::8d5:59ff:fe6a:677/64 scope link
       valid_lft forever preferred_lft forever
```

Note that the `NO-CARRIER` is gone now, the state has changed from `DOWN` to
`UP`, and the interface has an IP address assigned. Now, you should be able to
ping Fobnail and use the echo service exposed on UDP port 9400.

```
$ ping 169.254.0.1
64 bytes from 169.254.0.1: icmp_seq=1 ttl=64 time=0.419 ms
64 bytes from 169.254.0.1: icmp_seq=2 ttl=64 time=0.107 ms
64 bytes from 169.254.0.1: icmp_seq=3 ttl=64 time=0.103 ms
64 bytes from 169.254.0.1: icmp_seq=4 ttl=64 time=0.105 ms

$ nc -u 169.254.0.1 9400
hello
hello
1234567890
1234567890
(...)
```

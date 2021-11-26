# Developing firmware on PC

Fobnail firmware can be built to run either on nRF52840 or Linux. The latter
option is more suitable for development as it does not need any hardware except
a development machine.

## Networking setup

Fobnail firmware needs a special network interface to communicate with the
external world. You can create such an interface by executing the following
commands:

```shell
$ sudo ip tuntap add fobnail0 mode tap user `whoami`
$ sudo ip addr add 169.254.0.8/16 dev fobnail0
$ sudo ip link set dev fobnail0 up
```

To make this persistent across reboots, you can use `systemd-networkd`, which
ships with most distributions. To make the interface persistent, follow these
steps:

* Create `/etc/systemd/network/fobnail0.netdev`.

  ```ini
  [NetDev]
  Name=fobnail0
  Kind=tap

  [Tap]
  User=put your user name here
  ```

* Create `/etc/systemd/network/fobnail0.network`.

  ```ini
  [Match]
  Name=fobnail0

  [Network]
  Address=169.254.0.8/16
  DHCPServer=false
  ```

* Enable `systemd-networkd`.

  ```shell
  $ sudo systemctl enable --now systemd-networkd
  ```

You should see a new network interface created.

```shell
$ ip addr show dev fobnail0
40: fobnail0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
    link/ether 8e:36:f4:4c:98:28 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::8c36:f4ff:fe4c:9828/64 scope link
       valid_lft forever preferred_lft forever
```

## Docker setup

Make sure you have the latest Fobnail SDK installed. See
[Environment setup](environment.md) for instructions. For the network interface
to be accessible from the SDK environment, Docker must be run with
`--privileged` and `--network=host` switches set. Make sure the `fobnail0`
interface is accessible.

```shell
$ docker run --rm -it --privileged --net=host 3mdeb/fobnail-sdk /bin/bash
(docker)$ ip link
40: fobnail0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN mode DEFAULT group default qlen 1000
    link/ether 8e:36:f4:4c:98:28 brd ff:ff:ff:ff:ff:ff
```

> Note: TAP interface passthrough may not work if your distro is using SELinux.
> If you can't see the network interface in Docker, try turning SELinux into
> permissive mode.
> ```shell
> $ sudo setenforce 0
> ```

> If you are using WSL (Windows Subsystem for Linux) with Docker Desktop,
> interface passthrough won't work because Docker is located on a separate
> Virtual Machine. In that case, you can still use Fobnail SDK to build the
> firmware, but the firmware must be run from outside the container.

## Building and running firmware locally

Clone Fobnail firmware repo and start Fobnail SDK.

```shell
$ git clone https://github.com/fobnail/fobnail
$ cd fobnail
$ ./run-container
```

By default, firmware is built for the `thumbv7em-none-eabihf` target, so you
have to set the target manually to build it for the host.

```shell
(docker)$ env FOBNAIL_LOG=trace cargo run --target=x86_64-unknown-linux-gnu
    Finished dev [unoptimized + debuginfo] target(s) in 0.60s
     Running `target/x86_64-unknown-linux-gnu/debug/fobnail-poc`
 DEBUG fobnail > UDP socket initialized
 TRACE smoltcp::socket::set > [0]: adding
 (...)
```

> FOBNAIL_LOG environment variable sets log level. The possible log levels are:
> error, warning, info, debug, trace.


When Fobnail firmware is running, network interface should be up.

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
$ nc -u 169.254.0.1 9400
hello
hello
1234567890
1234567890
(...)
```
# Environment setup

This document describes how to prepare development environment for fobnail.

## Building docker container

For the purpose of fobnail application development we have put necessary
software into a single docker container which we called [fobnail SDK](https://github.com/fobnail/fobnail-sdk).
In order to build the container follow the steps below:

1. [Install docker](https://docs.docker.com/engine/install/) using the guide
   for Linux distribution present on your computer.
2. Add your user to docker group: `sudo usermod -aG docker $USER`. Then log out
   of the desktop session then log in again. This step is one-time only.
2. Clone the fobnail SDK repository:
   ```
   git clone https://github.com/fobnail/fobnail-sdk.git
   ```
3. Go to fobnail SDK directory: `cd fobnail-sdk`
4. Execute `./build.sh`. It will build the container with following software
   available:
   - Rust 1.55.0
   - Cargo-embed: always the latest version available from Cargo registry

You will need fobnail-sdk to proceed with any work. The process will take a
while to build the container. If we have built the container, time to verify
it. We will build a sample application from the fobnail directory using the
freshly built docker container. Follow the steps below to test the container:

1. `git clone https://github.com/fobnail/nrf-hal`
2. `cd nrf-hal`
3. Switch to `blinky-demo-nrf52840` branch: `git checkout blinky-demo-nrf52840`
4. Start container: `./run-container.sh`
5. Build a blinky application:
   ```
   cd examples/blinky-demo-nrf52840
   cargo build --target thumbv7em-none-eabihf
   ```
6. At the end of the process you should see something like this:
   ```
   Compiling blinky-demo-nrf52840 v0.1.0 (/home/build/nrf-hal/examples/blinky-demo-nrf52840)
   Finished dev [unoptimized + debuginfo] target(s) in 1m 14s
   ```

## Networking setup

You can build Fobnail firmware to run directly on your PC (see
[Developing firmware on PC](local_development.md) for more information).
In that case, Fobnail firmware needs a special network interface to communicate
with the external world. You can create such an interface by executing the
following commands:

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

You should see the same interface inside Docker container.

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

# Networking setup

Note: these steps are required **only when firmware is run on PC**. They
shouldn't be done for hardware Fobnail Token.

To run Fobnail firmware directly on your PC a special network interface to
communicate with the external world is needed. You can create such an interface
by executing the following commands:

```shell
sudo ip tuntap add fobnail0 mode tap user `whoami`
```

```shell
sudo ip addr add 169.254.0.8/16 dev fobnail0
```

```shell
sudo ip link set dev fobnail0 up
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
  sudo systemctl enable --now systemd-networkd
  ```

You should see a new network interface created.

```shell
ip addr show dev fobnail0
```

```shell
40: fobnail0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
    link/ether 8e:36:f4:4c:98:28 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::8c36:f4ff:fe4c:9828/64 scope link
       valid_lft forever preferred_lft forever
```

You should see the same interface inside Docker container.

```shell
run-fobnail-sdk.sh /bin/bash
```

Inside docker container:

```shell
ip link show fobnail0
```

```shell
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

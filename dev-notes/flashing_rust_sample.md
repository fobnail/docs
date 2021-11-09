# Flashing rust hello-world app

## Environment prepare

* Download and build Fobnail SDK and

```shell
$ git clone https://github.com/fobnail/fobnail-sdk.git
$ cd fobnail-sdk
$ git checkout rust
$ ./build.sh
```

## Building app

This is a custom app based on `blinky-button-demo` sample from `nrf-hal`. Code
has been adapted run on nRF52840 dongle and is available from our
[nrf-hal](https://github.com/fobnail/nrf-hal/tree/master/examples/blinky-demo-nrf52840)
fork.

* Clone code and start Fobnail SDK

```shell
$ git clone https://github.com/fobnail/nrf-hal
$ cd nrf-hal
$ ./run-container.sh
```

* Build sample app

```
(docker)$ cd examples/blinky-demo-nrf52840
(docker)$ cargo build --target thumbv7em-none-eabihf
```

## Flashing app with J-Link compatible device

This is the simpler procedure, it requires J-Link (or compatible device) or
CMSIS-DAP device. I am using
[nRF52840-DK](https://www.nordicsemi.com/Products/Development-hardware/nrf52840-dk)
development board (J-Link compatible).

Start Fobnail SDK and check if your device is detected `probe-rs`.

```shell
(docker)$ probe-rs-cli list
The following devices were found:
[0]: J-Link (J-Link) (VID: 1366, PID: 1015, Serial: 000683081460, JLink)
```

Add correct Udev rules to grant yourself needed permissions.

```
SUBSYSTEM=="usb", ATTR{idVendor}=="1366", ATTR{idProduct}=="1015", OWNER="username", MODE="0660"
```

Set correct `OWNER`, if needed update `idVendor` and `idProduct` to match with
your device (check output of `lsusb`). Save these rules to
`/etc/udev/rules.d/99-usb.rules` (this must be done on host), then run

```shell
$ sudo systemctl reload systemd-udevd
```

Re-plug J-Link for rules to start working, then type following command to build
and run app on target device.

```shell
(docker)$ cargo embed --target thumbv7em-none-eabihf
```

Console with emulated UART should start, you should see `Blinky demo starting`
message, on-board leds should start flashing.

## Flashing app with OpenOCD

Flashing is done using Olimex ARM-USB-OCD-H and OpenOCD. Note that OpenOCD isn't
part of Fobnail SDK so you have to install it manually.

You need OpenOCD version 0.11.0 or later for flashing to work, if your distro
has older version you need to build it from source.

```shell
$ sudo apt install libusb-dev libusb-1.0-0-dev libusb-1.0-0
$ sudo apt install libhidapi-dev libftdi-dev libftdi1-dev
$ sudo apt install libtool # for openocd

$ git clone https://git.code.sf.net/p/openocd/code openocd
$ cd openocd
$ ./bootstrap
$ mkdir build; cd build
$ ../configure --enable-cmsis-dap --enable-openjtag --prefix=/opt/openocd
$ make
$ sudo make install
```

* Build app using Fobnail SDK, then convert it into binary format

```shell
$ cargo objdump --target thumbv7em-none-eabihf -- -O binary app.bin
```

* Flash it

```shell
$ /opt/openocd/bin/openocd \
    -f interface/ftdi/olimex-arm-usb-ocd-h.cfg \
    -f interface/ftdi/olimex-arm-jtag-swd.cfg \
    -f target/nrf52.cfg \
    -c "program app.bin"
```

* Device should start blinking both red and green LED with 1 second intervals

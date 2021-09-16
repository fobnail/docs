# Flashing sample applications

## Preparing environment

Prepare the development environment first as described in
[Environment setup document](environment.md).

## nRF52840 buttons and LEDs

![](images/nRF52840_dongle_buttons_leds.svg)

## Building and flashing mcuboot

[MCUboot](https://github.com/mcu-tools/mcuboot) is a secure bootloader for
32-bit microcontrollers. It also provides options for device firmware upgrade,
e.g. via USB which makes it an ideal target (along with secure boot features)
for fobnail.

If you prepared the environment properly, you should have cloned the
[fobnail main repository](https://github.com/fobnail/fobnail)
which contains zephyr and mcuboot. To compile mcuboot follow the steps below:

1. Plug the nRF52840 dongle into the USB port of your computer while holding
   the reset switch (as shown on the picture above). The red diode (LED2)
   should start blinking (in fading manner). This means the dongle is in
   bootloader mode. Yuo may watch the dmesg to see if the USB device appears
   with `sudo dmesg -wH`:
   ```
   usb 1-1.4: new full-speed USB device number 62 using ehci-pci
   usb 1-1.4: New USB device found, idVendor=1915, idProduct=521f, bcdDevice= 1.00
   usb 1-1.4: New USB device strings: Mfr=1, Product=2, SerialNumber=3
   usb 1-1.4: Product: Open DFU Bootloader
   usb 1-1.4: Manufacturer: Nordic Semiconductor
   usb 1-1.4: SerialNumber: EB443991D70D
   ```
2. `git clone https://github.com/fobnail/fobnail.git` if you haven't done it
   yet.
3. Go to fobnail main repository directory: `cd /path/to/fobnail` and initialize
   submodules:
   ```
   git submodule update --init --checkout
   ```
4. Run the fobnail-sdk container with `./run-container.sh`. Wait or a while
   while the container initializes the necessary Zephyr modules. When finished
   (the prompt appears), proceed with next step.
5. Execute `make mcuboot_demo dfu`. This command will build the mcuboot for
   nRF52840 dongle and flash it using USB DFU (device firmware upgrade). At the
   end of the process you should see:
   ```
   Zip created at build/mcuboot.zip
     [####################################]  100%          
   Device programmed.
   ```
6. Now unplug the nRF52840 dongle. In order to prove the process was
   successful, plug the dongle again into the USB port of your computer but
   this time, hold the SW1 switch while plugging. The SW1 switch is the white
   button near the reset button. This will put the dongle into mcuboot
   bootloader mode. DO nto press the reset button as it will put the dongle
   into Nordic bootloader mode (we do not want this). Note this is different
   than previous situation, because this time we are using mcuboot instead of
   stock Nordic's bootlaoder. Unlike Nordic's bootloader, mcuboot does not
   blink a LED while in bootloader mode so we have to watch dmesg for new USb
   device `sudo dmesg -wH`:
   ```
   usb 1-1.4: New USB device found, idVendor=2fe3, idProduct=0100, bcdDevice= 2.06
   usb 1-1.4: New USB device strings: Mfr=1, Product=2, SerialNumber=3
   usb 1-1.4: Product: MCUBOOT
   usb 1-1.4: Manufacturer: ZEPHYR
   usb 1-1.4: SerialNumber: 05AA335B46AC1D80
   cdc_acm 1-1.4:1.0: ttyACM0: USB ACM device
   ```
   Notice the different in the Product name: `Open DFU Bootloader` vs
   `MCUBOOT`. This means we have booted the mcuboot in bootloader mode.

## Building and flashing Zephyr blinyk sample

[Zephyr](https://github.com/zephyrproject-rtos/zephyr) is a very popular RTOS
which supports a wide variety of microcontrollers including nRF52840 dongle. We
will leverage it to show how to launch a sample application on nRF52840 dongle
using mcuboot. If you don't have the environment ready go to [Preparing environment](#preparing-environment).

In order to flash a sample application on the nRF52840 dongle we will use
blinky sample application from Zephyr repository. Follow the steps below to
build and flash blinky sample:

1. Put the dongle into mcuboot bootloader mode as instructed in step 6 of
   [Building and flashing mcuboot](#building-and-flashing-mcuboot)
2. Go to fobnail main repository directory: `cd /path/to/fobnail`
3. Run the fobnail-sdk container with `./run-container.sh`. This time the
   container will not initialize the environment if you have done it
   previously. You will see an error like this (which you may ignore):
   ```
   FATAL ERROR: already initialized in /home/build, aborting.
   Note:
       In your environment, ZEPHYR_BASE is set to:
       /home/build/zephyr
   
       This forces west to search for a workspace there.
       Try unsetting ZEPHYR_BASE and re-running this command.
   ```
4. Execute `make blinky_demo dfu`. This command will build the mcuboot for
   nRF52840 dongle and flash it using USB DFU (device firmware upgrade). At the
   end of the process you should see:
   ```
   === image configuration:
   partition offset: 65536 (0x10000)
   partition size: 385024 (0x5e000)
   rom start offset: 512 (0x200)
   === signing binaries
   unsigned bin: /home/build/build/blinky/zephyr/zephyr.bin
   signed bin:   build/blinky.signed.bin
    13.55 KiB / 13.55 KiB    [============================================================================================================================================================] 100.00%    3.09 KiB/s 4s
   Done
   Done
   ```
   Notice this time the output from DFU looks different as we are using mcuboot
   DFU application instead of Nordic's.
5. To verify the process was successful, unplug the dongle and plug it again to
   the USB port of your CPU. This time do not press any buttons. YOo should
   notice a green LED blinking (LED1). This means the Zephyr blinky sample has
   been flashed with USB DFU and launched successfully.

## Summary

MCUboot is a great bootloader with secure boot and USB DFU capabilities for
secure applications and great firmware update experience.
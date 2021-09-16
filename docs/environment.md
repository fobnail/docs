# Environment setup

This document describes hwo to prepare development environment for fobnail.

## Building docker container

For the purpose of fobnail application development we have put necessary
software int oa single docker container which we called [fobnail SDK](https://github.com/fobnail/fobnail-sdk).
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
4. Execute `./build.sh`. It will build the container from the most recent
   stable software components described in
   [tools-versions file](https://github.com/fobnail/fobnail-sdk/blob/main/tools-versions).
   Components that are used in the process:
   - NRF commandline tools: v10.14.0
   - Zephyr SDK: v0.13.0
   - Zephyr RTOS: v2.6.0
   - Segger JLink: V7.54a
   - cmake: 3.21.2

You will need fobnail-sdk to proceed with any work. The process will take a
while to build the container. If we have built the container, time to verify
it. We will build a sample application from the fobnail directory using the
freshly built docker container. Follow the steps below to test the container:

1. `git clone https://github.com/fobnail/fobnail.git`
2. `cd fobnail`
3. `git submodule update --init --checkout`
4. `./run-container.sh` This will launch the container. It will take a moment
   to initialize when invoke for the first time in given directory.
5. Build a blinky application:
   ```
   west build -b nrf52840dongle_nrf52840 -d build/blinky zephyr/samples/basic/blinky
   ```
6. At the end of the process you should see something like this:
   ```
   [138/138] Linking C executable zephyr/zephyr.elf
   Memory region         Used Size  Region Size  %age Used
              FLASH:       12872 B      1020 KB      1.23%
               SRAM:        4288 B       256 KB      1.64%
           IDT_LIST:          0 GB         2 KB      0.00%
   
   ```

## Preparing environment

1. Be sure to have the most recent container compiled as described in
   [Building docker container](#building-docker-container) section.
2. Clone the repository with submodules if you haven't done it yet:
   ```
   git clone https://github.com/fobnail/fobnail.git
   cd fobnail
   git submodule update --init --checkout
   ```
   If you have cloned the repository pull the recent changes with `git pull`.
3. Enter the docker container with `./run-container.sh`

> Note that sometimes the container will not catch the dongle device. In order
> for the container to see the dongle, plug it first into the USB port (in
> bootloader mode if needed) and the execute `./run-container.sh`.

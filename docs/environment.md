# Environment setup

This document describes hwo to prepare development environment for fobnail.

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
4. To use latest container checkout `rust` branch: `git checkout rust`
5. Execute `docker build . -f Dockerfile.rust -t 3mdeb/fobnail-rust-sdk`. It
   will build the container with following software available:
   - Rust 1.55.0
   - Cargo-embed: always the latest version available from Cargo registry

You will need fobnail-sdk to proceed with any work. The process will take a
while to build the container. If we have built the container, time to verify
it. We will build a sample application from the fobnail directory using the
freshly built docker container. Follow the steps below to test the container:

1. `git clone https://github.com/fobnail/nrf-hal`
2. `cd nrf-hal`
3. Start container: `./run-container.sh`
4. Build a blinky application:
   ```
   cd examples/blinky-demo-nrf52840
   cargo build --target thumbv7em-none-eabihf
   ```
5. At the end of the process you should see something like this:
   ```
   Compiling blinky-demo-nrf52840 v0.1.0 (/home/build/nrf-hal/examples/blinky-demo-nrf52840)
   Finished dev [unoptimized + debuginfo] target(s) in 1m 14s
   ```

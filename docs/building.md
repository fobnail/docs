# Cloning and building

Fobnail project consists of 3 main repositories with code:

- [Fobnail firmware](https://github.com/fobnail/fobnail) (Rust)
- [Platform Owner](https://github.com/fobnail/fobnail-platform-owner) (C)
- [Attester](https://github.com/fobnail/fobnail-attester) (C)

There is also [Fobnail SDK](https://github.com/fobnail/fobnail-sdk) that
simplifies building of firmware by using Docker container, but only
[one script](https://github.com/fobnail/fobnail-sdk/blob/main/run-fobnail-sdk.sh)
from this repository is actually needed.

Following steps were successfully performed on Ubuntu 22.04.1 LTS.

## Fobnail Token firmware

Start by installing prerequisites:

```shell
sudo apt -y install git
```

Log out of the desktop session then log in again - this is required after making
the change to user groups. You may confirm it by running `groups`.

Clone the repository along with its submodules:

```shell
git clone https://github.com/fobnail/fobnail --recurse-submodules
```

Build process for Token firmware is by far the most complicated one. For that
reason use of Fobnail SDK is strongly suggested. Follow [these
instructions](/fobnail-sdk) to install it.

With the SDK installed, one can prepare for the build process itself. Firmware
can be build for a physical Fobnail Token or simulated on PC, with different
preparatory steps required.

#### Environment variables common for both targets

Fobnail Token firmware is configured with environment variables passed to
`build.sh`.

Note that due to the way Docker mounts directories all files and directories
pointed to by following variables must be located somewhere in `fobnail`
directory.

- `FOBNAIL_PO_ROOT` - **required** option, must point to valid PEM or DER file
  with Platform Owner's root certificate. See [this document](/keys_and_certificates/#platform-owner-certificate-chain)
  for description of PO certificate chain and instructions for building such.
  For Fobnail Token firmware it is enough to create only [root CA certificate](/keys_and_certificates/#root-ca).
  The rest of chain will be required for provisioning.

- `FOBNAIL_EK_ROOT_DIR` - points to directory with TPM root certificates.
  Fobnail repository includes [such directory](https://github.com/fobnail/fobnail/tree/main/tpm_ek_roots)
  which normally should be used as `FOBNAIL_EK_ROOT_DIR`, but you may change it
  if certificates supplied in repository became outdated. Either this or
  `FOBNAIL_EXTRA_EK_ROOT` (or both) **must be specified**.

- `FOBNAIL_EXTRA_EK_ROOT` - points to one specific TPM certificate. Useful when
  [testing with TPM emulator](/tpm-simulators/), or to limit Fobnail usage to
  one specific TPM vendor. Either this or `FOBNAIL_EK_ROOT_DIR` (or both)
  **must be specified**.

#### Building and flashing firmware to Fobnail Token

For hardware setup instructions see [Flashing preparation](flashing_preparation.md).

Building and flashing is performed by executing (from `fobnail` directory):

```shell
env FOBNAIL_PO_ROOT=root_ca.crt FOBNAIL_EK_ROOT_DIR=tpm_ek_roots \
    ./build.sh -t nrf --run
```

A console with Fobnail Token output will be displayed. It is not required for
normal operation but can be useful for debugging. It can be closed with Ctrl-C
at any point. After that, Token can be used without nRF52840-DK - just plug it
wherever it's needed.

#### Building and running firmware locally

Make sure you have the network set up properly. See [Networking setup](networking_setup.md)
for instructions. For firmware to work properly you need the `fobnail0` network
interface configured.

To build and run firmware execute the following commands (from `fobnail`
directory).

```shell
env FOBNAIL_LOG=info FOBNAIL_PO_ROOT=root_ca.crt \
    FOBNAIL_EK_ROOT_DIR=tpm_ek_roots ./build.sh -t pc --run
```

`FOBNAIL_LOG` environment variable sets log level. The possible log levels are:
`error`, `warning`, `info`, `debug`, `trace`. This variable is valid only for PC
target.

Another variable used only on PC is `FOBNAIL_DEVICE_ID` - on hardware we used
FICR registers to create a device ID, this gives a way of configuring it for
emulation. This variable is optional, without it an ID of 0 is used.

`build.sh` automatically starts Token emulation. It runs until it's terminated
with Ctrl-C.

## PC applications

Platform Owner and Attester have similar set of prerequisites, listed below.
Depending on use case, they may or may not be run on the same PC, so these steps
may have to be repeated on different computers.

```shell
sudo apt -y install git make gcc autoconf automake pkg-config libtool libssl-dev
```

Both applications use [libcoap with v3 API](https://github.com/obgm/libcoap/tree/release-4.3.0)
which isn't provided by main Linux distributions yet, so it has to be built from
sources and installed:

```shell
git clone https://github.com/obgm/libcoap.git --recurse-submodules
cd libcoap
git checkout release-4.3.0
./autogen.sh && \
./configure --exec-prefix=/usr --disable-tests --disable-documentation \
    --disable-manpages --enable-dtls --with-tinydtls --enable-fast-install && \
make && sudo make install
```

#### Platform Owner

With prerequisites installed, building Platform Owner application is simple -
just clone and build:

```shell
git clone https://github.com/fobnail/fobnail-platform-owner --recurse-submodules
cd fobnail-platform-owner
make
```

Produced binary is located in `bin/fobnail-platform-owner`, from where it can be
moved to `PATH` or just started from there.

#### Attester

Attester requires additional packages to talk with TPM and download TPM's
certificate chain from Internet:

```shell
sudo apt -y install libtss2-dev libcurl4-openssl-dev
```

After that, building is simple:

```shell
git clone https://github.com/fobnail/fobnail-attester --recurse-submodules
cd fobnail-attester
make
```

Two executables are produced in `bin` folder: `fobnail-attester` and
`fobnail-attester-with-provisioning`. First one can be moved to `PATH` for
easier use. The latter is expected to be run only once per Token, by an
administrator in a controlled environment, and not for daily use, so it can be
not installed to avoid confusion.

## Summary and next steps

Congratulations, you have built all of the components of Fobnail! From here you
should proceed to [Token provisioning guide](/token_provisioning).

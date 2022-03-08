# Installing certificates

Currently certificates are installed by copying them directly to emulated flash
memory. Currently, this is supported only on PC target, for nRF we would need
tool to copy to and from nRF flash.

* Before starting, build `lfs` tool (located in Fobnail firmware repo in
  `tools/lfs`).

  ```shell
  $ cargo build --release
  ```

* For convenience there is `install-certificate` command that takes PEM/DER
  certificate, creates required directories and writes DER-encoded certificate
  to memory.

  Example usage:

  ```shell
  $ lfs -f target/flash.bin install-certificate /path/to/root_ca.pem --trusted
  $ lfs -f target/flash.bin install-certificate /path/to/intermediate_ca.pem
  ```

  * `-f` - path to flash storage, this file is automatically created when
    running Fobnail on PC.
  * `--trusted` flag controls whether certificate is marked as trusted, this
    flag should be set for root CAs. For intermediate certificates it shouldn't.
  * `--der` flag allows to install DER certificate (default is PEM).
  * `root_ca.pem` and `intermediate_ca.pem` are certs generated with
    [fobnail-attester](https://github.com/fobnail/fobnail-attester)

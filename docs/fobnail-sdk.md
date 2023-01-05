# Fobnail SDK

Fobnail SDK is a Docker container image that contains all the tools required to
build Fobnail Token firmware, flash it to nRF52840 dongle and debug it.

Full repository can be found [here](https://github.com/fobnail/fobnail-sdk), but
to use it you can just simply download script `run-fobnail-sdk.sh` and save it
to a directory in your `PATH` environment variable:

```shell
sudo wget -O /usr/local/bin/run-fobnail-sdk.sh \
  https://raw.githubusercontent.com/fobnail/fobnail-sdk/main/run-fobnail-sdk.sh
sudo chmod +x /usr/local/bin/run-fobnail-sdk.sh
```

For proper operation, you need to have Docker [installed](https://docs.docker.com/engine/install/ubuntu/)
and [properly configured](https://docs.docker.com/engine/install/linux-postinstall/).

First execution of this script downloads Docker container image, which can be
both time- and bandwidth-consuming. Running this script without any additional
parameters will open interactive shell.

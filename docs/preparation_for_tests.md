# Preparation for tests

## Requirements

* root.crt
* provision.sh

## Procedure

1. To prepare the Fobnail environment run the following commands:

```bash
git clone git@github.com:fobnail/fobnail.git
cd fobnail/
git submodule update --init --recursive
git checkout token_provisioning
```

1. https://github.com/fobnail/fobnail-sdk/blob/main/run-fobnail-sdk.sh - Put
    this file in PATH - to do this put this fail to `~/.local/bin/` and run the
    following command:

    ```bash
    source ~/.bashrc
    ```

1. Put the `root.crt` file in the `fobnail` dictionary.
1. [Configure Network](https://fobnail.3mdeb.com/environment/#networking-setup)
1. To prepare some variables, run the following commands:

```bash
export FOBNAIL_PO_ROOT=root.crt
export FOBNAIL_LOG=debug
```

1. To start Fobnail run the following command:

```bash
./build.sh --run
```

1. Wait for similar output:

```bash
.Finished dev [optimized + debuginfo] target(s) in 3m 21s
    Running `target/x86_64-unknown-linux-gnu/debug/fobnail`
INFO  fobnail > Hello from main
```

1. To provisioning token run the script with the following commands:

```bash
./provision.sh
```

TBD

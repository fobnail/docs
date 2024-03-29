# Preparation for tests

This document describes step by step how to prepare the environment to run
Fobnail and prepare it for provisioning.

## Running Fobnail Firmware

1. [Install docker](https://docs.docker.com/engine/install/) using the guide
   for Linux distribution present on your computer.
1. Add your user to the docker group: `sudo usermod -aG docker $USER`. Then log
    out of the desktop session then log in again. This step is one-time only.
1. Prepare the Fobnail environment by opening the terminal and running the
    following commands:

    ```bash
    git clone git@github.com:fobnail/fobnail.git
    cd fobnail/
    git checkout token_provisioning
    git submodule update --init --recursive
    ```

1. Put file `run-fobnail-sdk.sh` from `fobnail-sdk` repository in PATH by
    running the following commands in the new terminal:

    ```bash
    git clone git@github.com:fobnail/fobnail-sdk.git
    cd fobnail-sdk/
    mv run-fobnail-sdk.sh ~/.local/bin/
    source ~/.bashrc
    ```

1. [Configure Network](https://fobnail.3mdeb.com/environment/#networking-setup),
    for a fast but temporary solution, you can only use the first 3 commands.
1. Put the `fobnail_test_root.crt` file in the main `fobnail` dictionary.

    > You can take `fobnail_test_root.crt` from
    [fobnail-test-environment](https://github.com/fobnail/fobnail-test-environment)

1. Start Fobnail by running the following commands:

    ```bash
    export FOBNAIL_PO_ROOT=fobnail_test_root.crt
    export FOBNAIL_LOG=debug
    ./build.sh --run
    ```

1. Wait for similar output:

    ```bash
    .Finished dev [optimized + debuginfo] target(s) in 3m 21s
        Running `target/x86_64-unknown-linux-gnu/debug/fobnail`
    INFO  fobnail > Hello from main
    ```

    If the output as above is displayed, it means that the CoAP server was
    successfully set up and started.

## Installing libraries required for provisioning

1. Install the necessary libraries by running the following command:

    ```bash
    pip3 install cbor pem hexdump
    ```

1. Install the `libcoap` library by following the
    [procedure](https://github.com/fobnail/fobnail-attester#install-dependencies-for-building-the-project)

    If you run into `git checkout` problems, try copying all of the content
    below and running it in a terminal:

    ```bash
    git clone --recursive -b 'develop' \
    'https://github.com/obgm/libcoap.git' /tmp/libcoap && \
    cd /tmp/libcoap && \
    git checkout --recurse-submodules 2a329e1c763a47a910f075aad4478398aaaea400
    ```

    > libcoap3 is required, coap-client from libcoap2 will not work properly

1. Open the terminal and run the following command to test coap-client:

    ```bash
    coap-client -m get coap://169.254.0.1
    ```

    If you get `4.04 Not found` output, you have installed the dependencies
    correctly.

The CoAp server should be running to be able to receive commands from the
client. Note that provisioning token won't work when the token is already
provisioned. To unprovision the token remove the `flash.bin` file from the
`fobnail/target` directory.

## Testing environment

1. Download and prepare the testing repository by opening the terminal and
    running the following commands:

    ```bash
    git clone git@github.com:fobnail/fobnail-test-environment.git
    cd fobnail-test-environment
    virtualenv -p $(which python3) robot-venv
    source robot-venv/bin/activate
    pip install -U -r requirements.txt
    ```

1. Running test cases example:

    ```bash
    robot -L TRACE -o <output-file-prefix> -r <report_file_prefix> -l <output_file_name>.html -v absolute_fobnail_path:<path_to_fobnail> tests/<test_file_name>.robot
    ```

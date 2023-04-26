# Fobnail Token provisioning

The process of Token provisioning is initialized by [Platform Owner
application](/building/#platform-owner).

## Prerequisites

Platform Owner binary expects two files: PO certificate chain `cert_chain.pem`
and private key `po_priv_key.pem`, in that order. For quick start both of those
files can be produced using [these TL;DR
instructions](/keys_and_certificates/#tldr-version).

Certificate chain must begin with the root CA that was passed as
`FOBNAIL_PO_ROOT` when [Token firmware was
built](/building/#environment-variables-common-for-both-targets).

## Provisioning instructions

Instructions below assume that `fobnail-platform-owner` was installed to a
directory in `$PATH` variable and both required PEM files are in current
directory. The process differs slightly between physical Token and emulated one,
choose proper version before continuing.

=== "Physical Fobnail Token"

    1. Plug in the Token. It will present itself as network interface, check its
       name in `dmesg` - search for lines like these one, where `enp0s26u1u1` is
       the name of the interface:

        ```text
        [54.928687] cdc_eem 1-1.1:1.0 usb0: register 'cdc_eem' at usb-0000:00:1a.0-1.1, CDC EEM Device, f6:82:61:5e:71:2a
        [55.052471] cdc_eem 1-1.1:1.0 enp0s26u1u1: renamed from usb0
        [55.153313] cdc_eem 1-1.1:1.0 enp0s26u1u1: unregister 'cdc_eem' usb-0000:00:1a.0-1.1, CDC EEM Device
        ```

        Red LED should be lit on the Token to signal that it isn't provisioned
        yet.

    2. Assign IP to that interface. There is no need to make it permanent,
       provisioning is a one-time operation. Change `enp0s26u1u1` to the name of
       your interface:

        ```shell
        sudo nmcli con add save no con-name Fobnail ifname enp0s26u1u1 \
             type ethernet ip4 169.254.0.8/16 ipv6.method disabled
        ```

    3. Run Platform Owner application:

        ```shell
        fobnail-platform-owner cert_chain.pem po_priv_key.pem
        ```

        After a while, green LED will blink once to report that the Token
        provisioning was successful. After that Token enters idle state in which
        both LEDs blink shortly every 5 seconds.

=== "PC simulation"

    1. Network should be set up as a part of build instructions. If that isn't
       the case, follow [instructions](networking_setup.md).

    2. Start firmware by executing the following commands (from `fobnail`
       directory). Make sure that `FOBNAIL_PO_ROOT` and `FOBNAIL_EK_ROOT_DIR`
       are the same as during initial build, otherwise firmware will be rebuilt.
       If TPM is also emulated, `FOBNAIL_EXTRA_EK_ROOT` in addition or instead
       of `FOBNAIL_EK_ROOT_DIR` should be provided.

        ```shell
        env FOBNAIL_LOG=info FOBNAIL_PO_ROOT=root_ca.crt \
            FOBNAIL_EK_ROOT_DIR=tpm_ek_roots ./build.sh -t pc --run
        ```

        Contrary to physical Token, simulation doesn't have physical LEDs, so
        the state is reported in textual form:

        ```text
        INFO  pal_pc::led > LED controller state: TokenNotProvisioned
        ```

    3. Run Platform Owner application:

        ```shell
        fobnail-platform-owner cert_chain.pem po_priv_key.pem
        ```

        This will start the provisioning, with progress and result printed in
        the console:

        ```text
        INFO  fobnail     > new client: 169.254.0.8:40625
        INFO  fobnail::server::token_provisioning > Commencing token provisioning
        INFO  fobnail::server::token_provisioning > Certificate chain loaded
        INFO  fobnail::server::token_provisioning > Generating new Ed25519 keypair
        INFO  fobnail::server::token_provisioning > Token provisioning complete
        INFO  pal_pc::led                         > LED controller state: TokenProvisioningComplete
        INFO  pal_pc::led                         > LED controller state: TokenWaiting
        INFO  fobnail                             > disconnect client: 169.254.0.8:40625
        ```

---

## Summary and next steps

In this state Fobnail Token is provisioned and awaits platform provisioning.
Further instructions can be found in example applications, like [disk
encryption](/examples/disk_encryption/#guide).

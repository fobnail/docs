# TPM simulators

It is possible to use TPM simulators instead of real TPMs **for testing purposes
only**. Two simulators were tested and are known to be working with Fobnail:
[Microsoft's](https://github.com/microsoft/ms-tpm-20-ref) and
[IBM's](https://sourceforge.net/projects/ibmswtpm2/). Refer to their respective
documentations for build and launch instructions.

## Differences between simulated and real TPM

The main difference is that simulator is started after system is up. This means
that there will be no measurements of firmware nor OS. Attestation is based on
these measurements, without them there is nothing to attest to, which makes use
of simulated TPM useless for practical applications. It also means that nothing
called `TPM2_Startup` command. Attester is able to detect this and calls that
function automatically.

Hardware and firmware TPM vendors give assurance of proper functioning of their
TPMs. This is done by creating a certificate for each TPM separately,
specifically for their Endorsement Keys (EKs). These certificates are created
during TPM manufacturing and provisioning process, and are written to NVRAM.
They point to their signing certificates through `authorityInfoAccess` X509V3
extension. The chain continues until a self-signed root that is implicitly
trusted. That root must be known by Fobnail Token in advance in order to trust
given TPM.

For simulated TPM, such certificate has to be manually created and injected into
NVRAM. EK certificate is created for EK created from EPS (Endorsement Primary
Seed), which is randomly created when the simulator is started for the first
time. Nobody is issuing certificates for all instances of simulated TPMs. One of
the reasons is probably the fact that emulators are not fully compatible with
specification, especially when it comes to hardware protection mechanisms
(Physical Presence, protected storage etc.).

There are also other small differences, e.g. after non-orderly shutdown `safe`
field in `TPMS_CLOCK_INFO` structure is set after 12 seconds in simulator, while
specification allows for up to 2^22 milliseconds (around 70 minutes) to help
with NVRAM wear leveling. Note that this field is checked by Fobnail during
attestation.

## Provisioning simulated TPM

As mentioned, EK certificate must be written to TPM NVRAM. In addition, it must
point to valid CA certificate that will be downloaded during platform
provisioning. A script for simplifying this process and required configuration
are included in [Attester's repository](https://github.com/fobnail/fobnail-attester/tree/main/tools).
`tpm2-tools` and `openssl` are used by this script, so they must be installed.

To create root certificate, EK certificate and write the latter to NVRAM, it is
enough to call the following (`-s` tells to send `TPM2_Startup` command):

```
$ ./tools/tpm_manufacture.sh -s
Sending TPM2_Startup command
Generating a RSA private key
................................................+++++
................+++++
writing new private key to '/home/user/fobnail-attester/tools/keys_and_certs/ca_priv.pem'
-----
Signature ok
subject=C = PL, O = Fobnail, ST = State, CN = EK certificate
Getting CA Private Key


Done.
To test:
        tpm2_nvread -C o 0x01C00002 | openssl x509 -text -noout -inform DER
```

Check if certificate was written properly by executing suggested command:

```
$ tpm2_nvread -C o 0x01C00002 | openssl x509 -text -noout -inform DER
WARN: Reading full size of the NV index
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            66:e5:d8:5a:1f:4b:38:95:67:9c:2a:b7:b1:2d:e5:55:e9:b2:ee:2e
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = PL, O = Fobnail, ST = State, CN = CA certificate
        Validity
            Not Before: Jan  4 19:14:00 2023 GMT
            Not After : Feb  3 19:14:00 2023 GMT
        Subject: C = PL, O = Fobnail, ST = State, CN = EK certificate
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:b1:a0:41:73:ca:33:e4:be:78:bf:21:e1:19:43:
                    22:1c:50:90:44:23:bd:a1:0b:d6:1c:9e:c7:bd:07:
                    c0:ba:1e:be:f3:b1:c7:32:9d:8a:99:64:6c:4b:6e:
                    7f:fe:9a:2a:58:50:34:cd:4b:a1:38:c4:f3:a2:25:
                    01:87:00:9d:75:5a:b6:8d:46:a6:c9:b6:8c:62:5c:
                    8d:95:1a:06:d9:38:79:d5:41:80:ce:e1:4e:23:f0:
                    fe:b3:43:58:05:13:38:7b:cb:c5:8c:b6:ea:6b:b1:
                    75:10:79:7b:f0:1e:99:01:94:43:5d:4b:85:22:a5:
                    66:cd:9c:c4:36:49:97:df:03:26:9c:2c:28:5a:1c:
                    6b:fc:59:3d:e8:94:e7:dc:21:74:25:9a:32:d1:21:
                    2b:98:8d:e4:c3:84:39:cc:eb:c6:1b:b6:05:97:c2:
                    61:22:ed:f4:3a:3c:31:e5:e2:c8:b6:41:f9:33:6d:
                    de:9e:3d:bf:bf:11:d8:e6:65:d8:7e:24:d1:11:00:
                    54:a6:71:f8:8e:04:a8:81:a7:51:22:07:2f:67:ee:
                    b7:11:8f:d9:f6:c9:07:b8:61:9b:ee:45:c6:2e:ad:
                    b0:26:5e:88:52:3b:5c:3d:82:36:45:26:00:35:c0:
                    4c:4a:3d:c9:6a:4a:4a:ed:32:65:51:e5:b4:a4:1c:
                    65:eb
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Authority Key Identifier:
                keyid:7C:12:EB:3F:07:E3:82:43:57:7C:0D:17:84:40:E3:70:CE:39:C2:E4

            Authority Information Access:
                CA Issuers - URI:http://127.0.0.1:8080/ca_cert.der

            X509v3 Basic Constraints:
                CA:FALSE
            X509v3 Key Usage:
                Key Encipherment
    Signature Algorithm: sha256WithRSAEncryption
         5e:b2:f9:76:a6:fe:25:31:ec:b5:c9:27:7b:96:9b:48:00:e2:
         17:da:cf:6e:2a:40:f4:8f:76:2c:d8:75:eb:42:45:43:ef:c0:
         db:56:53:1b:f8:57:4e:3d:56:3c:6b:83:8a:55:a8:53:cd:4c:
         c6:71:ed:8f:d0:80:6d:6d:5b:08:6f:07:60:7d:6c:c6:7e:37:
         00:6b:00:41:22:6b:2f:06:10:07:f0:3a:d4:f4:26:ca:32:a4:
         9a:30:f0:d5:0f:48:a4:dd:fd:59:8c:35:b2:5c:62:9f:71:db:
         4e:f0:37:68:10:38:c3:eb:96:f2:85:fa:32:ba:e2:9b:a5:94:
         df:9d:bd:df:69:ff:d8:98:40:2d:0c:30:d4:b4:76:db:fb:e6:
         a8:04:9d:81:83:66:24:83:8f:eb:4d:c4:9e:7f:da:18:22:1a:
         99:4e:15:f1:cf:56:05:37:c7:be:98:44:be:d6:d5:ae:e2:f6:
         7e:40:a7:07:c9:c0:b1:da:c6:b4:7b:bf:0b:41:89:5e:d4:76:
         98:51:81:1e:4f:dd:6c:f5:aa:5b:32:ed:ea:de:8b:cd:ca:f1:
         36:0a:41:0a:46:ff:44:d7:8a:fe:fe:c4:0f:d2:7c:53:76:ad:
         0f:df:1b:65:51:ed:05:7b:be:bf:8a:4e:68:65:4f:6d:3f:14:
         27:d3:2c:f3
```

> There may be more lines with warnings and even errors printed before that.
> They come from TCTI (TPM Command Transmission Interface) and are caused by
> failed attempts to use another interface (e.g. physical TPM) before driver
> for simulator is tried. They can be safely ignored, as long as content of
> certificate is printed afterwards.

Certificate in `tools/keys_and_certs/ca_cert.der` is the one that has to be
passed to `build.sh` in `FOBNAIL_EXTRA_EK_ROOT` variable, see [building
instructions](/building/#environment-variables-common-for-both-targets)
for details.

Configuration assumes that CA certificate will be made available at address
`http://127.0.0.1:8080/ca_cert.der`, to change this please edit `ek_v3.ext` in
`tools` directory.

If you have to re-run the manufacturing process (e.g. EK root certificate is
lost or some changes were done to its extensions) start `tpm_manufacture.sh`
with additional `-f` flag that will overwrite EK certificate even if it is
already present. **Do not use it on real TPM, there is no way of recovering
original certificate if it was removed!**.

## Using simulated TPM

For most cases running TPM simulator is enough. Only platform provisioning needs
HTTP server in addition to TPM simulator. Easiest way to do so is to use Python:

```
$ cd tools/keys_and_certs
$ python -m http.server 8080
```

HTTP server can be closed after platform is successfully provisioned and never
started again, unless another provisioning is required.

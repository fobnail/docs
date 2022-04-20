# Review of Attester/Verifier

The project [Fraunhofer-SIT/charra](https://github.com/Fraunhofer-SIT/charra)
implements high-level functionality for CHARRA.

There are two binary executable files after compiling and building - bin/attester bin/verifier

[How it Works: Protocol Flow](https://github.com/Fraunhofer-SIT/charra#how-it-works-protocol-flow)
[Build and Run](https://github.com/Fraunhofer-SIT/charra#build-and-run)

* Available options for attester:
```shell
$ ./bin/attester --help

Usage: attester [OPTIONS]
     --help:                     Print this help message.
 -v, --verbose:                  Set CHARRA and CoAP log-level to DEBUG.
 -l, --log-level=LEVEL:          Set CHARRA log-level to LEVEL. Available are: TRACE, DEBUG, INFO, WARN, ERROR, FATAL. Default is INFO.
 -c, --coap-log-level=LEVEL:     Set CoAP log-level to LEVEL. Available are: DEBUG, INFO, NOTICE, WARNING, ERR, CRIT, ALERT, EMERG, CIPHERS. Default is INFO.
     --port=PORT:                Open PORT instead of port 5683.
DTLS-PSK Options:
 -p, --psk:                      Enable DTLS protocol with PSK. By default the key 'Charra DTLS Key' and hint 'Charra Attester' are used.
 -k, --key=KEY:                  Use KEY as pre-shared key for DTLS. Implicitly enables DTLS-PSK.
 -h, --hint=HINT:                Use HINT as hint for DTLS. Implicitly enables DTLS-PSK.
DTLS-RPK Options:
                                 Charra includes default 'keys' in the keys folder, but these are only intended for testing. They MUST be changed in actual production environments!
 -r, --rpk:                      Enable DTLS-RPK (raw public keys) protocol . The protocol is intended for scenarios in which public keys of either attester or verifier or both of them are pre-shared.
     --private-key=PATH:         Specify the path of the private key used for RPK. Currently only supports DER (ASN.1) format.
                                 By default 'keys/attester.der' is used. Implicitly enables DTLS-RPK.
     --public-key=PATH:          Specify the path of the public key used for RPK. Currently only supports DER (ASN.1) format.
                                 By default 'keys/attester.pub.der' is used. Implicitly enables DTLS-RPK.
     --peer-public-key=PATH:     Specify the path of the reference public key of the peer, used for RPK. Currently only supports DER (ASN.1) format.
                                 By default 'keys/verifier.pub.der' is used. Implicitly enables DTLS-RPK.
     --verify-peer=[0,1]:        Specify whether the peers public key shall be checked against the reference public key. 0 means no check, 1 means check. By default the check is performed.
                                 WARNING: Disabling the verification means that connections from any peer will be accepted. This is primarily intended for the verifier, which may not have
                                 the public keys of all attesters and does an identity check with the attestation response. Implicitly enables DTLS-RPK.

To specify TCTI commands for the TPM, set the 'CHARRA_TCTI' environment variable accordingly.

```

* Log trace of attester...

```shell
$ ./bin/attester
12:01:50 DEBUG src/attester.c:139: [attester] Attester Configuration:
12:01:50 DEBUG src/attester.c:140: [attester]     Used local port: 5683
12:01:50 DEBUG src/attester.c:141: [attester]     DTLS-PSK enabled: false
12:01:50 DEBUG src/attester.c:149: [attester]     DTLS-RPK enabled: false
12:01:50 INFO  src/attester.c:185: [attester] Initializing CoAP in block-wise mode.
12:01:50 INFO  src/attester.c:236: [attester] Creating CoAP server endpoint using UDP.
12:01:50 INFO  src/attester.c:248: [attester] Registering CoAP resources.
12:01:50 INFO  src/util/coap_util.c:192: [coap-util] Adding CoAP FETCH resource 'attest'.
12:01:50 DEBUG src/attester.c:253: [attester] Entering main loop.
12:01:57 INFO  src/attester.c:299: [attester] Resource 'attest': Received message.//<= Here we got request from verifier
12:01:57 INFO  src/attester.c:313: [attester] Received data of length 52.
12:01:57 INFO  src/attester.c:315: [attester] Received data of total length 52.
12:01:57 INFO  src/attester.c:320: [attester] Parsing received CBOR data. //Parsing request in libcbor library
12:01:57 DEBUG src/util/cbor_util.c:114: CBOR parser: found type CborArrayType.
12:01:57 DEBUG src/util/cbor_util.c:114: CBOR parser: found type CborFalseType.
12:01:57 DEBUG src/util/cbor_util.c:114: CBOR parser: found type CborByteStringType.
12:01:57 DEBUG src/util/cbor_util.c:114: CBOR parser: found type CborByteStringType.
12:01:57 DEBUG src/util/cbor_util.c:114: CBOR parser: found type CborArrayType.
12:01:57 DEBUG src/util/cbor_util.c:114: CBOR parser: found type CborArrayType.
12:01:57 DEBUG src/util/cbor_util.c:114: CBOR parser: found type CborIntegerType.
12:01:57 DEBUG src/util/cbor_util.c:114: CBOR parser: found type CborArrayType.
12:01:57 DEBUG src/util/cbor_util.c:114: CBOR parser: found type CborIntegerType.
12:01:57 DEBUG src/util/cbor_util.c:114: CBOR parser: found type CborIntegerType.
12:01:57 DEBUG src/util/cbor_util.c:114: CBOR parser: found type CborIntegerType.
12:01:57 DEBUG src/util/cbor_util.c:114: CBOR parser: found type CborIntegerType.
12:01:57 DEBUG src/util/cbor_util.c:114: CBOR parser: found type CborIntegerType.
12:01:57 DEBUG src/util/cbor_util.c:114: CBOR parser: found type CborIntegerType.
12:01:57 DEBUG src/util/cbor_util.c:114: CBOR parser: found type CborIntegerType.
12:01:57 DEBUG src/util/cbor_util.c:114: CBOR parser: found type CborIntegerType.
12:01:57 DEBUG src/util/cbor_util.c:114: CBOR parser: found type CborIntegerType.
12:01:57 DEBUG src/util/cbor_util.c:114: CBOR parser: found type CborByteStringType.
12:01:57 INFO  src/attester.c:330: [attester] Preparing TPM quote data.
12:01:57 INFO  src/attester.c:341: Received nonce of length 20:
                                   0x68e0b92acc18652a74f31add1319866af1fccc80
12:01:57 INFO  src/attester.c:369: [attester] Loading TPM key.v  //Looks like it doesn't generate any keys every time
12:01:57 INFO  src/core/charra_key_mgr.c:36: Loading key "PK.RSA.default".
12:01:57 INFO  src/util/tpm2_util.c:117: Primary Key created successfully.
12:01:57 INFO  src/attester.c:378: [attester] Do TPM Quote.
12:01:57 INFO  src/attester.c:386: [attester] TPM Quote successful.
12:01:57 INFO  src/attester.c:414: [attester] Preparing response.
12:01:57 INFO  src/attester.c:438: [attester] Marshaling response to CBOR.
12:01:57 TRACE src/core/charra_marshaling.c:362: <ENTER> charra_marshal_attestation_response()
12:01:57 TRACE src/core/charra_marshaling.c:342: <ENTER> charra_marshal_attestation_response_size()
12:01:57 TRACE src/core/charra_marshaling.c:290: <ENTER> charra_marshal_attestation_response_internal()
12:01:57 DEBUG src/core/charra_marshaling.c:382: Size of marshaled data is 1277 bytes.
12:01:57 DEBUG src/core/charra_marshaling.c:389: Allocated 1277 bytes of memory.
12:01:57 TRACE src/core/charra_marshaling.c:290: <ENTER> charra_marshal_attestation_response_internal()
12:01:57 INFO  src/attester.c:446: [attester] Size of marshaled response is 1277 bytes.
12:01:57 INFO  src/attester.c:453: [attester] Adding marshaled data to CoAP response PDU and send it.
```

## A phase 3 requrements and correspondence to source code of CHARRA project where it implemented (or could be if not implememnted)

- FOB3.1 The Attester MUST generate attestation identity key
  - the key is generated once inside of the TPM - to be done by the attester
    application, or maybe by some tpm scripts first (prior running application?)

  - then, it is simply retreived from TPM (reference to the code: TBD)

- FOB3.2 The Attester MUST sign attestation identity key

- FOB3.3 The Attester MUST provide the public key to Fobnail Token

- FOB3.4 The Fobnail Token MUST verify received attestation identity key

- FOB3.5 The Attester MUST obtain metadata and send the signed metadata to the Fobnail Token

- FOB3.6 The Fobnail Token MUST verify received metadata

- FOB3.7 The Fobnail Token MUST count checksum of the received metadata

- FOB3.8 The Fobnail Token MUST save the metadata hash in the Fobnail memory

- FOB3.9 The Attester MUST obtain RIM and send the signed RIM to the Fobnail Token

- FOB3.10 The Fobnail Token MUST receive and verify the Reference Integration Measurements

- FOB3.11 The Fobnail Token MUST save the RIM in the Fobnail token memory

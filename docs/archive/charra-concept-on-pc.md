# How to run CHARRA concept on PC

The detailed steps of how to build and run CHARRA applications (Attester and
Verifier) are described on corresponding GitHub repository:
 - [build and run CHARRA on
   PC](https://github.com/Fraunhofer-SIT/charra#build-and-run), please follow
   the steps listed there before running rest of this PoC

In general there are two common ways to reach the goal:
  - build docker container with binary packages
  - build neccessary packages from sources inside docker container

The first path is faster and more preferable. There are two scripts there:
  - `docker/build.sh` - to build and install docker container
  - `docker/run.sh` - to run docker container with installed packages for CHARRA

When the project is built one may run and test Attester and Verifier
application:

1. List the docker networks:

```shell
$ docker network ls
NETWORK ID     NAME             DRIVER    SCOPE
....
a13b7967a1d4   `charra_default`   bridge    local
....
```

2. Configure network for docker in order to allow containers to communicate.
E.g. edit file ./docker/run.sh and add internal virtual network to docker run
command:
```shell
--network=charra_default
```

3. Execute the `./docker.run.sh` script in two terminal windows

4. Specify IP address of Attester in file `src/verifier.c` (`char dst_host[16]`
in src/verifier.c) and rebuild the project in container.

> Note: check IP address inside docker container
```shell
(docker)$ cd charra
(docker)$ make
```

5. Run Attester in one terminal

```
(docker)$ ./bin/attester
10:53:23 DEBUG src/attester.c:139: [attester] Attester Configuration:
10:53:23 DEBUG src/attester.c:140: [attester]     Used local port: 5683
10:53:23 DEBUG src/attester.c:141: [attester]     DTLS-PSK enabled: false
10:53:23 DEBUG src/attester.c:149: [attester]     DTLS-RPK enabled: false
10:53:23 INFO  src/attester.c:185: [attester] Initializing CoAP in block-wise mode.
10:53:23 INFO  src/attester.c:236: [attester] Creating CoAP server endpoint using UDP.
Feb 21 10:53:23.959 DEBG created UDP  endpoint 0.0.0.0:5683
10:53:23 INFO  src/attester.c:248: [attester] Registering CoAP resources.
10:53:23 INFO  src/util/coap_util.c:192: [coap-util] Adding CoAP FETCH resource 'attest'.
10:53:23 DEBUG src/attester.c:253: [attester] Entering main loop.
```

6. Start Verifier in second terminal and get the result

* Attester output

```
(docker)$ ./bin/attester
11:00:50 INFO  src/attester.c:185: [attester] Initializing CoAP in block-wise mode.
11:00:50 INFO  src/attester.c:236: [attester] Creating CoAP server endpoint using UDP.
11:00:50 INFO  src/attester.c:248: [attester] Registering CoAP resources.
11:00:50 INFO  src/util/coap_util.c:192: [coap-util] Adding CoAP FETCH resource 'attest'.
11:00:54 INFO  src/attester.c:299: [attester] Resource 'attest': Received message.
11:00:54 INFO  src/attester.c:313: [attester] Received data of length 52.
11:00:54 INFO  src/attester.c:315: [attester] Received data of total length 52.
11:00:54 INFO  src/attester.c:320: [attester] Parsing received CBOR data.
11:00:54 INFO  src/attester.c:330: [attester] Preparing TPM quote data.
11:00:54 INFO  src/attester.c:341: Received nonce of length 20:
                                   0x00eda2a762c7b36ea0493814c98e076a45799119
11:00:54 INFO  src/attester.c:369: [attester] Loading TPM key.
11:00:54 INFO  src/core/charra_key_mgr.c:36: Loading key "PK.RSA.default".
11:00:54 INFO  src/util/tpm2_util.c:117: Primary Key created successfully.
11:00:54 INFO  src/attester.c:378: [attester] Do TPM Quote.
11:00:54 INFO  src/attester.c:386: [attester] TPM Quote successful.
11:00:54 INFO  src/attester.c:414: [attester] Preparing response.
11:00:54 INFO  src/attester.c:438: [attester] Marshaling response to CBOR.
11:00:54 INFO  src/attester.c:446: [attester] Size of marshaled response is 1277 bytes.
11:00:54 INFO  src/attester.c:453: [attester] Adding marshaled data to CoAP response PDU and send it.
```

* Verifier output

```
(docker)$ ./bin/verifier
11:00:54 INFO  src/verifier.c:244: [verifier] Initializing CoAP in block-wise mode.
11:00:54 INFO  src/verifier.c:252: [verifier] Registering CoAP response handler.
11:00:54 INFO  src/verifier.c:290: [verifier] Creating CoAP client session using UDP.
11:00:54 INFO  src/verifier.c:332: [verifier] Creating attestation request.
11:00:54 INFO  src/verifier.c:467: [verifier] Generated nonce of length 20:
                                                  0x00eda2a762c7b36ea0493814c98e076a45799119
11:00:54 INFO  src/verifier.c:342: [verifier] Marshaling attestation request data to CBOR.
11:00:54 INFO  src/verifier.c:352: [verifier] Adding CoAP option URI_PATH.
11:00:54 INFO  src/verifier.c:360: [verifier] Adding CoAP option CONTENT_TYPE.
11:00:54 INFO  src/verifier.c:370: [verifier] Creating request PDU.
11:00:54 INFO  src/verifier.c:384: [verifier] Sending CoAP message.
11:00:54 INFO  src/verifier.c:392: [verifier] Processing and waiting for response ...
11:00:54 INFO  src/verifier.c:514: [verifier] Resource 'attest': Received message.
11:00:54 INFO  src/verifier.c:531: [verifier] Received data of length 1277.
11:00:54 INFO  src/verifier.c:533: [verifier] Received data of total length 1277.
11:00:54 INFO  src/verifier.c:538: [verifier] Parsing received CBOR data.
11:00:54 INFO  src/verifier.c:565: [verifier] Starting verification.
11:00:54 INFO  src/verifier.c:585: [verifier] Loading TPM key.
ERROR:esys:src/tss2-esys/esys_iutil.c:389:iesys_handle_to_tpm_handle() Error: Esys invalid ESAPI handle (40000001).
WARNING:esys:src/tss2-esys/esys_iutil.c:410:iesys_is_platform_handle() Convert handle from TPM2_RH to ESYS_TR, got: 0x40000001
11:00:54 INFO  src/verifier.c:591: [verifier] External public key loaded.
11:00:54 INFO  src/verifier.c:595: [verifier] Preparing TPM Quote verification.
11:00:54 INFO  src/verifier.c:606: [verifier] Verifying TPM Quote signature with TPM ...
ERROR:esys:src/tss2-esys/esys_iutil.c:389:iesys_handle_to_tpm_handle() Error: Esys invalid ESAPI handle (40000001).
WARNING:esys:src/tss2-esys/esys_iutil.c:410:iesys_is_platform_handle() Convert handle from TPM2_RH to ESYS_TR, got: 0x40000001
11:00:54 INFO  src/verifier.c:612: [verifier]     => TPM Quote signature is valid!
11:00:54 INFO  src/verifier.c:622: [verifier] Converting TPM public key to mbedTLS public key ...
11:00:54 INFO  src/verifier.c:633: [verifier] Verifying TPM Quote signature with mbedTLS ...
11:00:54 INFO  src/verifier.c:639: [verifier]     => TPM Quote signature is valid!
11:00:54 INFO  src/verifier.c:660: [verifier] Verifying nonce ...
11:00:54 INFO  src/verifier.c:665: [verifier]     => Nonce in TPM Quote is valid! (matches the one sent)
11:00:54 INFO  src/verifier.c:678: [verifier] Verifying PCRs ...
11:00:54 INFO  src/verifier.c:680: [verifier] Actual PCR composite digest from TPM Quote is:
                                              0x2d5565fb483d8ea4525a7a9229677d1038ad34b6e22c8d5152e1d7f7b9817597
11:00:54 INFO  src/core/charra_rim_mgr.c:82: Found matching PCR composite digest at index 2 of the PCR sets.
11:00:54 INFO  src/verifier.c:691: [verifier]     => PCR composite digest is valid!
11:00:54 INFO  src/verifier.c:743: [verifier] +----------------------------+
11:00:54 INFO  src/verifier.c:746: [verifier] |   ATTESTATION SUCCESSFUL   |
11:00:54 INFO  src/verifier.c:751: [verifier] +----------------------------+
```

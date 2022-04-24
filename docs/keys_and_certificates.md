# Keys and certificates

Many different keys and certificates are used by all of Fobnail's components.
Most of them are created by the code, but some must be provided externally.

## List of keys and certificates used

### Platform Owner

#### Knows in advance

- Platform Owner key (signing and encryption)
    - Fobnail and remote platform provisioning
    - used for TLS and for signing certificates
- Platform Owner certificate
    - Fobnail and remote platform provisioning
    - has to be able to provide whole CA chain
    - both Fobnail Token and Platform must trust root CA in this chain
- root CA certificate for EK certificate chain
    - remote platform provisioning

#### Receives

- Fobnail Identity key
    - Fobnail provisioning
- Fobnail Encryption key
    - Fobnail provisioning
- EK certificate
    - remote platform provisioning
- AIK
    - remote platform provisioning

#### Produces

- Fobnail Identity certificate
    - Fobnail provisioning
- Fobnail Encryption certificate
    - Fobnail provisioning
- AIK certificate
    - remote platform provisioning
    - simple signature instead of certificate may be sufficient

### Fobnail Token

#### Knows in advance

- root CA certificate for EK certificate chain
    - local platform provisioning
    - possibly along with whole chain
- root CA for Platform Owner certificate chain
    - Fobnail provisioning

#### Receives

- Platform Owner certificate chain
    - Fobnail provisioning
    - saved in flash, used in local platform provisioning and attestation
- Fobnail Identity certificate
    - Fobnail provisioning
    - signed by Platform Owner
- Fobnail Encryption certificate
    - Fobnail provisioning
    - signed by Platform Owner
- EK certificate
    - local platform provisioning
- AIK
    - local platform provisioning
    - remote platform provisioning - signed by Platform Owner
    - stored in flash and used during attestation

#### Produces

- Fobnail Identity key (signing and encryption)
    - Fobnail provisioning
    - used for TLS
    - key and certificate signed by Platform Owner is stored for later use
- Fobnail Encryption key (encryption)
    - Fobnail provisioning
    - used for storage encryption

### Platform

#### Knows in advance

- root CA for Platform Owner certificate chain
    - remote platform provisioning
    - local platform provisioning and attestation - Fobnail Identity key is signed
    by Platform Owner, so Platform needs a way of verifying trust chain
- Endorsement Key (encryption)
    - remote and local platform provisioning
    - for this project may be considered immutable persistent key unique to TPM
- EK certificate
    - EK is unique so is its certificate
    - _usually_ saved in TPM NVRAM
    - signed by TPM manufacturer CA

#### Receives

- Platform Owner certificate chain
    - remote platform provisioning - directly from Platform Owner
    - local platform provisioning and attestation - from Fobnail Token
    - used for identification, TLS and verifying Fobnail Identity certificate
- Fobnail Identity certificate
    - local platform provisioning and attestation
    - used for identification and TLS

#### Produces

- Attester Identity Key (signing)
    - local and remote platform provisioning
    - created by TPM
    - trusted based on Make/ActivateCredential and pairing with EK instead of
    certificate

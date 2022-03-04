# Reference Integrity Manifests

Trusted Computing Group (TCG) specifies Reference Integrity Manifest (RIM) as a
structure to convey information about the software and firmware running on
given system. These manifests are used as an input to retrieve the reference
golden measurements which indicate the system is genuine, trusted and runs a
configuration and software approved by the RIM issuer (System Supplier, Value
Added Reseller, Maintenance Organization, etc.).

![RIMs](/images/rims.png)

The information described in this document is an interpretation of the
following TCG specifications for usage with fobnail:

* [TCG Reference Integrity Manifest (RIM) Information Model](https://trustedcomputinggroup.org/wp-content/uploads/TCG_RIM_Model_v1p01_r0p16_pub.pdf)
* [TCG PC Client Reference Integrity Manifest Specification](https://trustedcomputinggroup.org/wp-content/uploads/TCG_PC_Client_RIM_r1p04_pub.pdf)

These specifications also base on [NIST IR 8060](https://nvlpubs.nist.gov/nistpubs/ir/2016/NIST.IR.8060.pdf#%5B%7B%22num%22%3A100%2C%22gen%22%3A0%7D%2C%7B%22name%22%3A%22XYZ%22%7D%2C108%2C721%2Cnull%5D)

## RIMs used by fobnail

There are two types of RIMs (Base RIM and Support RIM) with different payload
types they hold, but there is always Base RIM at the beginning. Base RIM can
hold 3 different payload types:

* **direct** - contains reference integrity measurements, no Binding
  Specification is required for interpreting the payload contents
* **indirect** - contains the reference integrity measurements for Support RIM.
  A Binding Specification defines the Support RIM
* **hybrid** - contains references that are a combination of direct and
  indirect

The PC Client RIM specification mandates usage of indirect Base RIM with at
least one Support RIM. Each Base RIM starts with Software Identity header
(SWID) compliant with ISO/IEC19770-2 (SWID) specification following guidelines
from NIST IR 8060. This headers serves a purpose of identifying the creator of
the RIM and the product/system the RIM corresponds to. it contains a tagId
field which is a GUID that must be equal to `ReferenceManifestGuid` in
`TCG_Sp800-155-PlatformId_Event` from TPM Event Log of the system.

![RIMs](/images/rim_relationship.png)

SWID is followed by the

* Entity structure - describing the RIM creator
* Link structure - containing a reference/download URL for firmware/software
  the RIM corresponds to or an URL to the previous RIM superseded/patched by
  current RIM
* Metadata - various human readable strings describing versions of the RIM,
  platform information and firmware information
* Payload - description of the data held by the RIM
* Signature - digital signature of the RIM

Example Base RIM:

```
Software Identity Name: Example.com BIOS
  version : 01 (MUST BE BIOS version)
  tagId: 94f6b457-9ac9-4d35-9b3f-78804173b65as (from TPM event log)
  tagVersion:0
Entity (tagCreator) Name
  Regid: http://Example.com
  Role: softwareCreator tagCreator
Links:
  installation media url: https://Example.com/support/ProductA/firmware/installfiles
Meta:
  colloquialVersion: Firmware_2019
  Edition: 12
  Product: ProductA
  Revision: r2
  PayloadType: Indirect
  PlatformManufacturerStr: Example.com
  PlatformManufacturerId: 00201234
  PlatformModel: ProductA
  PlatformVersion:01
  FirmwareManufacturerStr: BIOSVendorA
  FirmwareManufacturerId: 00213022
  FirmwareModel:A0
  FirmwareVersion: 12
  BindingSpec: PC Client RIM
  BindingSpecVersion: 1.2
  pcURIlocal: /boot/tcg/manifest/swidtag
Payload:
  Directory: /boot/tcg/manifest/rim
    File1: Example.com.Notebook.3.rimel
     Version: 01.00
     size= 15400
     SHA256 hash: a314fc2dc663ae7a6b6bc6787594057396e6b3f569cd50fd5ddb4d1bbafd2b6a
     supportRIMFormat: TPM Event Log Assertions
    File2: Example.com.Notebook.3.rimpcr
     Version: 01.00
     size: 1024
     SHA256:hash: 532eaabd9574880dbf76b9b8cc00832c20a6ec113d682299550d7a6e0f345e25
     supportRIMFormat: TPM PCR Assertions
Signature: (Enveloped) 
  sigAlgorithm: rsa-sha256 (MUST be a TCG listed algorithm)
  hashAlgorithm: sha256 (MUST be a TCG listed algorithm)
  keyInfo: subjectKeyIdentifier OR X509Data
  digest: 8A4C98225D855D388B42FD8AA4FA1646CFAC666E43FD6A7B950BB3E0B12134AA
  signature <hex>
Table 1 RIM IM Element-Attribute table
```

The RIM above contains two files, one with TPM event log, second with reference
PCRs. These Support RIM formats are called TPM Event Log Assertions and TPM PCR
Assertions. These formats are mandated by PC Client RIM binding specification
(in other words the TCG PC Client Reference Integrity Manifest Specification).

The content of the files is simple:

* TPM Event Log Assertions - raw TPM event log dump
* TPM PCR Assertions - raw output from TPM2_PCR_Read command (without response
  header):
    * `UINT32 pcrUpdateCounter`
    * `TPML_PCR_SELECTION pcrSelectionOut`
    * `TPML_DIGEST pcrValues`

There may be multiple Support RIMs with PCRs or Event Logs.

There is a tool able to generate the RIMs in
[Host Integrity at Runtime and Start-up (HIRS) repository](https://github.com/nsacyber/HIRS/tree/master/tools/tcg_rim_tool)

## RIM packaging

The RIM format is not processor-friendly. The data is not structured. RIM can
be well described by XML, but in such form it is almost not consumable by a
small MCU. Thus a few modifications must be made in order to transfer such RIM
and its payload to fobnail. The following structures are proposed (as close to
specification as possible)to be used by fobnail firmware (all strings are NULL
terminated unless stated otherwise):

```C
struct rim_swid {
	char Name[]; // Product Name
	char Version[]; // BIOS version
	uint8_t tagId[16]; // `ReferenceManifestGuid` from TCG_Sp800-155-PlatformId_Event
	uint32_t tagVersion;
}

struct rim_entity {
	char Name[]; // RIM creator
	char RegID[]; // URI of tag creator
	uint8_t role; // bit field specifying the roles, one of
		      // softwareCreator (1 << 0)
		      // licensor (1 << 1)
		      // aggregator (1 << 2)
		      // tagCreator (1 << 3)
		      // adopted from ISO 19770-2
} __packed;

struct rim_metadata {
	char colloquialVersion[]; // the informal version of the product
	char edition[]; // the variation of the product (e.g., “Home”, “Enterprise”, “Student”),
	char product[]; // the base name of the product, exclusive of vendor,
			// colloquial version, edition, etc. 
	char revision[]; // the informal or colloquial representation of the sub-version
			// of the product (e.g., “SP1”, “R2”, “Beta 2”)
	char payloadType[]; // "direct", "indirect" OR "hybrid"
			    // MUST BE "indirect" for PC Client RIM
	char PlatformManufacturerStr[]; // from TCG_Sp800-155-PlatformId_Event[PlatformManufacturerStr]
	char PlatformManufacturerId[]; // from TCG_Sp800-155-PlatformId_Event[VendorID]
	char PlatformModel[]; // from TCG_Sp800-155-PlatformId_Event[PlatformModel]
	char PlatformVersion[]; // from TCG_Sp800-155-PlatformId_Event[PlatformVersion]
	char FirmwareManufacturerStr[]; // from TCG_Sp800-155-PlatformId_Event[FirmwareManufacturerStr]
	char FirmwareManufacturerId[]; // from TCG_Sp800-155-PlatformId_Event[FirmwareManufacturerId]
	char FirmwareModel[]; // from TCG_Sp800-155-PlatformId_Event[FirmwareModel]
	char FirmwareVersion[]; // from TCG_Sp800-155-PlatformId_Event[FirmwareVersion]
	char BindingSpec[]; // MUST BE "PC Client RIM"
	char BindingSpecVersion[]; // Currently "1.4"
} __packed;

struct rim_entry {
	char supportRIMFormat[]; // "TPM PCR Assertions" or "TPM Event LOG Assertions"
	uint32_t size; // size of the data
	uint8_t hash[32]; // SHA256 hash of the data
			  // (we probably won't use anything else than SHA256 here)
	uint8_t data[0]; // the data blob with PCRs or event log
} __packed;

struct rim_payload {
	uint32_t numEntries; // number of payload entries
	struct rim_entry entries[0]; // one or more entries
} __packed;

struct rim_signature {
	TPM_ALG_ID hashAlgorithm;
	TPM_ALG_ID sigAlgorithm;
	uint8_t keyInfoReference[]; // x509 data (signing certificate) or subjectKeyIdentifier
	uint8_t digest[]; // digest of size appropriate for hashAlgorithm
	uint8_t signature[]; // signature of size appropriate for sigAlgorithm
};

struct primary_rim {
	struct rim_swid SWID;
	struct rim_entity Entity;
	struct rim_metadata Meta;
	struct rim_payload Payload;
	struct rim_signature Signature;
};
```

There are a lot of strings in these structures which makes the RIM impossible
to be of fixed size. This may occur to be too spacy.

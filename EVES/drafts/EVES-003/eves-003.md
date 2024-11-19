---
eves-identifier: 003
title: ENVITED Asset Definition and Upload Process
author: Carlo van Driesten (@jdsika)
discussions-to: https://github.com/ASCS-eV/EVES/issues/4
status: Draft
type: Standards
created: 2024-11-19
requires: ["EVES-001"]
replaces: None
---

## Abstract

This specification defines the structure of an asset in the ENVITED-X Data Space and outlines the process for uploading assets to ensure compliance, security, and interoperability.
It leverages existing standards, such as the Gaia-X 4 PLC-AAD Manifest Ontology, and implements privacy layers, validation, and metadata mapping aligned with [Tezos TZIP-21](https://docs.tezos.com/architecture/governance/improvement-process#tzip-21-rich-contract-metadata).

## Motivation

Standardizing the definition and upload process for assets in the ENVITED-X Data Space ensures:

- Interoperability with existing Gaia-X data spaces.
- Security through CID-based identification and metadata validation.
- Scalability for integrating diverse asset types.

This EVES addresses the need for clear guidelines to onboard assets and synchronize data with ENVITED-X systems.

## Specification

### 1. Asset Definition

An asset is defined by the [Gaia-X 4 PLC-AAD Manifest Ontology](https://github.com/GAIA-X4PLC-AAD/ontology-management-base/tree/main/manifest/).
The example implementation [here](https://github.com/ASCS-eV/smart-contracts/tree/main/contracts/marketplace/metadata) is based on release v0.1.6 from the [HD-Map Asset Example](https://github.com/GAIA-X4PLC-AAD/hd-map-asset-example).

This EVES references the [Gaia-X Policy Rules Compliance Document (Release 24.11)](https://docs.gaia-x.eu/policy-rules-committee/compliance-document/24.11/). Compatibility with this release is **to be verified** in a future update of this EVES.

### 2. Pinata IPFS and CID Management

#### CID v1

- Assets uploaded via [Pinata](https://pinata.cloud/) MUST use [CID v1](https://docs.ipfs.tech/concepts/content-addressing/#version-1-v1).  
- This is achievable only through the API using the `pinataOptions` parameter, as outlined [here](https://docs.pinata.cloud/web3/pinning/pinata-metadata#pinataoptions).

#### File Naming

- Uploaded file names MUST exclude extensions (e.g., use `file` instead of `file.json`) to avoid issues such as double extensions during downloads (e.g., `file.json.json`).

### 3. Privacy Layer

The ENVITED-X Data Space implements a three-tiered privacy model:

| manifest:accessRole  | ENVITED-X Domain                                                      | Comment                               |
| -------------------- | --------------------------------------------------------------------- | ------------------------------------- |
| `owner`              | <https://assets.envited-x.net/Asset-CID>                              | CID v1, signed URLs, asset credential |
| `registeredUser`     | <https://metadata.envited-x.net/Asset-CID>                            | CID v1, signed URLs, DEMIM credential |
| `publicUser`         | >ipfs://Data-CID> to <https://ipfs.envited-x.net/Asset-CID/Data-CID>  | CID v1, public, indexer to new URL    |

### 4. Asset Validation and Upload Process

#### Step 1: Client-Side Pre-Validation

- Drag and drop `asset.zip` into the upload field.
- Validate the `manifest.json`:
  1. Ensure JSON structure matches the manifest SHACL constraints.
  2. Verify all referenced files exist locally or remotely as specified.
  3. Locate the `domainMetadata.json` file.
- Validate the `domainMetadata.json`:
  1. Extract SHACL constraints from the `domainMetadata.json` context.
  2. Validate JSON structure against domain-specific SHACLs.

#### Step 2: Upload Asset to ENVITED-X Data Space

- Trigger the upload process by clicking the "Upload" button.
- Calculate the CID of `asset.zip`.
- Rename `asset.zip` to `CID.zip` and store at `https://assets.envited-x.net/Asset-CID`.
- Store *registeredUser* metadata at `https://metadata.envited-x.net/Asset-CID`.
- Calculate CIDs for all `publicUser` data.
- Create a `tzip21_asset_manifest.json` by replacing relative paths in `manifest.json` with IPFS/envited-x.net URLs.

#### Step 3: Preview Data

- **TBD**: Define visualization and preview mechanisms for uploaded data.

#### Step 4: Mint Token

- Upload `publicUser` information and `tzip21_asset_manifest.json` to IPFS.
- Create `tzip21_token_metadata.json` and map metadata fields.
- Mint token with linked metadata.

#### Step 5: Listener and Database Synchronization

- Use a listener to detect mint events and synchronize data with the ENVITED-X database.
- Download `publicUser` data from IPFS to `https://ipfs.envited-x.net/Asset-CID`.

### 5. Database Synchronization

#### CID as the Primary Identifier

- The CID of the uploaded `asset.zip` serves as the unique identifier connecting data across all systems.  
- An additional UUID MAY be generated pre-mint to link the asset with the ENVITED-X database securely.

#### Pre-Mint Information

If additional non-public information needs to be stored in the database before minting, the CID can associate this data with the minted asset.

#### Synchronization and Security

The synchronization between the smart contract and the ENVITED-X database relies on:

1. The contract DID (current Ghostnet contract):  
   `did:tezos:NetXnHfVqm9iesp:KT1XC2fTBNqoafnrhEb7TuToRCzewgbHAhnA`
2. The user DID at the time of minting.
3. The transaction signature, validated against the user DID.
4. The generated `tzip21_asset_manifest.json` UUID:  
   `@id: "urn:uuid:cf1f329d-9c4c-458e-ba0a-a762a296b79c"`

## Backwards Compatibility

This specification introduces new processes for asset uploads and is fully compatible with existing ENVITED-X systems. No retroactive changes to previous assets are required.

## References

1. [Gaia-X 4 PLC-AAD Manifest Ontology](https://github.com/GAIA-X4PLC-AAD/ontology-management-base/tree/main/manifest/)
2. [HD-Map Asset Example](https://github.com/GAIA-X4PLC-AAD/hd-map-asset-example)
3. [Pinata Documentation](https://docs.pinata.cloud/web3/pinning/pinata-metadata#pinataoptions)
4. [RFC 2119: Key Words for Use in RFCs to Indicate Requirement Levels](https://datatracker.ietf.org/doc/html/rfc2119)
5. [Gaia-X Policy Rules Compliance Document (Release 24.11)](https://docs.gaia-x.eu/policy-rules-committee/compliance-document/24.11/)
6. [Reference Implementation](https://github.com/ASCS-eV/smart-contracts/tree/main/contracts/marketplace/metadata)

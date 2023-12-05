---
title: URAuth(Universal Resource Auth)
description:
keywords:
  - Register Ownership
  - DID
---

**UR-Auth** stands for *Universal Resource Auth*, a protocol that allows the ownership of web data and copyrighted tangible or intangible assets represented by URIs (Uniform Resource Identifiers) to be publicly registered on the internet using blockchain technology and Decentralized Identifiers (DID) technology. The URI, a widely-used identifier in web technology to uniquely identify logical and physical resources on and offline, registers its ownership on the blockchain using the DID of the URI's owner. This allows for a technical means for legitimate ownership claims when a large AI learns from it.

## Data Ownership Register

![Register Ownership](/media/images/docs/infrablockchain/service-chains/register-ownership.png)

The types of ownership registered on the UR-Auth blockchain can be broadly categorized into three areas:

- **Domain**: Represents general domains like `https://www.bc-labs.net`.
- **Web Service Accounts**: Represents accounts for a particular service like `https://instagram.com/user`.
- **Copyright of Data Files**: Represents ownership of local data. This could include the address value (e.g., CID) of distributed storage like IPFS.
- **Dataset**: Represents ownership of datasets created using URIs registered or not registered in the [URAuthTree](#ur-auth-tree). The location value of the storage where the dataset is stored can be included.

The registration process for ownership in each area is as follows:

### Domain

1. The domain owner creates their DID and sends a `request_register_ownership` transaction to the **UR-Auth** blockchain.
2. **UR-Auth** blockchain generates a challenge value.
3. The domain owner creates a `*.json` file containing domain verification information and uploads it to the `root` directory of that domain. For example, upload the `challenge.json` file to` bc-labs.net/root`.
4. Oracle nodes consisting of trusted nodes download the *.json file and send a verify_challenge transaction to the UR-Auth blockchain.
5. Once the verification is completed by a sufficient number of oracle nodes (e.g., more than 60%), the domain is registered in the [URAuth Tree](#ur-auth-tree), and ownership registration is complete.

### Web Service Accounts

*If ownership can be authenticated via OAuth:*

Register the URI in the UR-Auth Tree by sending a `claim_ownership` transaction.

*If OAuth authentication is not possible:*

The user posts a certificate on their feed proving ownership of the account.
Oracle nodes consisting of trusted nodes go through the verification process as described above.
Once verified by a sufficient number of oracle nodes, register in the [URAuthTree](#ur-auth-tree).

### Copyright of Data Files and Datasets

When registering a file or dataset, you receive a URI like `urauth://file/{cid}` or `urauth://dataset/{cid}`.
Register the URI in the [URAuthTree](#ur-auth-tree).

## UR-Auth Tree

![Ownership Claim on URAuth Tree](/media/images/docs/infrablockchain/service-chains/urauth-tree.png)

Web page URLs and subpage URLs within a website can be represented as nodes in a Tree data structure on the blockchain. One ***UR-Auth Tree*** corresponds to one website domain and its data identified by URIs. Each node of the ***UR-Auth Tree*** stores a ***UR-Auth Documen***t specifying ownership information, copyright information, data access rules, etc., for the URI.

## UR-Auth Tree Registration Rules

1. Registration of all sub-nodes in the ***URAuthTree*** must be done by one of the owners of the upper node unless verified by an oracle.
2. `http` or `https` is considered the same and should be omitted during registration. Any other protocol must be explicitly specified.
3. `www` should be omitted, and any other prefixes must be specified during registration.
4. The `root` node must be registered after verification by an oracle.


## Universal Resource Auth (UR-Auth) Document

Each node of the ***[URAuthTree](#ur-auth-tree)*** has a ***UR-Auth Document*** recording information such as ownership, copyright, data access rules, etc., that apply to the URI and all its sub-URIs. The UR-Auth Document can only be modified by the owner DID of that node.

```rust
#[derive(Encode, Decode, Clone, PartialEq, Debug, Eq, TypeInfo)]
pub struct URAuthDoc<Account> {
    pub id: DocId,
    pub created_at: u128,
    pub updated_at: u128,
    pub multi_owner_did: MultiDID<Account>,
    pub identity_info: Option<Vec<IdentityInfo>>,
    pub content_metadata: Option<ContentMetadata>,
    pub copyright_info: Option<CopyrightInfo>,
    pub access_rules: Option<Vec<AccessRule>>,
    pub asset: Option<MultiAsset>,
    pub data_source: Option<URI>,
    pub proofs: Option<Vec<Proof>>,
}
```

#### id
Identifier of the document.

#### created_at
Time the document was created.

#### updated_at
Time the document was updated.

#### multi_owner_did
List of owner DIDs for the document.

#### identity_info
List of identity information of the copyright holder.

#### contents_metadta
Information related to the content metadata associated with the URI.

#### access_rules
List of access rules for the URI.

#### asset
Information about tokenized datasets if the document is for a dataset.

#### data_source
URI information where the dataset is stored if the document is for a dataset.

#### proofs
List of electronic signatures for the document.

## Data Market

![Data Market Tracking Ownership](/media/images/docs/infrablockchain/service-chains/data-market.png)

Pre-packaged web datasets on the **URAuth** blockchain are also registered and traded in the Data Market, allowing web data consumers to pay legitimate data access costs to web data owners. This allows consumers to download and use pre-organized datasets for commercial AI machine learning, addressing data copyright issues.

### Tokenized of Dataset and AI Model

Ownership of datasets or AI models created with a URI can be tokenized since ownership information is specified on **URAuth** blockchain. When a dataset is actually sold in the Data Market, rewards can be distributed according to the tokenized stake.

## References

- [URAuth White Paper]()
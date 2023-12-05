---
title: System Token
description: Covers the general aspects of system tokens.
keywords:
    - System Token
---

**_InfraBlockchain_** does not have a single unique built-in native cryptocurrency token. Instead, every account created on the InfraBlockchain can issue system tokens, which are natively supported at the core level of the blockchain with standard token functionalities.

## Characteristics of System Tokens

-   System tokens are fiat-pegged tokens. They can only be issued on the chain to the extent that they are backed by real currency.

-   Therefore, system tokens do not have their own tokenomics.

-   Users can use system tokens to pay for transaction fees, which are [converted into votes for validators](./proof-of-transaction.md).

## System Token Identifiers

**_InfraBlockchain_** employs a multi-chain architecture, and each chain has identifiers for system tokens circulating within it.

```rust
pub struct SystemTokenId {
	#[codec(compact)]
	pub para_id: ParaId,
	#[codec(compact)]
	pub pallet_id: PalletId,
	#[codec(compact)]
	pub asset_id: AssetId,
}
```

`para_id`: Identifier of the parachain where the system token is circulating.

`pallet_id`: Identifier of the `Asset` pallet defined in the parachain.

`asset_id`: Local asset identifier mapped to the system token in the parachain.

## Types of System Tokens

### Original System Tokens

Represents system tokens created in the original ledger.

### Wrapped System Tokens

Refers to system tokens circulating in other ledgers.

For example, assume a `USD` token was created in parachain (1000). If this token is approved as a _system token_ for transaction fees by the governance of the Relay Chain, it can be used in parachain (2000) in the form of _Wrapped USD_.

## Transaction Fee Tokens Selected by Block Producers

![Managing System Tokens](/media/images/docs/infrablockchain/learn/protocol/system-token.png)

As briefly mentioned earlier, in InfraBlockchain, _system tokens_ can be used for transaction fees. For any arbitrary token created in another chain to be used as a _system token_, governance by the validators of the **_InfraRelayChain_** is necessary. Once a quorum is reached in governance, the token can be used as a _system token_ for transaction fees and blockchain transactions can occur using this token in other chains.

-   All state management of _system tokens_ (e.g., registration and deletion of system tokens, list of parachains using system tokens) takes place in the Relay Chain.

## Moving Forward

-   [Proof of Transaction](./proof-of-transaction.md)

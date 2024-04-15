---
title: System Token
description: Covers the overall content regarding the system token.
keywords:
  - System Token
---

**_InfraBlockchain_** does not have its own native virtual currency token (e.g., BITCOIN for Bitcoin, ETHER for Ethereum) and instead issues a **System Token** pegged to legal currency for use as transaction fees. It is also based on a multi-chain architecture, allowing it to use system tokens issued on other networks as transaction fees. To facilitate this, the system token's identification system is designed to reflect this concept in a multi-chain environment:

## System Token Identifier

All entities (e.g., tokens, accounts, etc.) existing in a multi-chain have relative positions. For example, the position of token X as seen from Parachain A and the position of token X as seen from the Relay Chain are expressed differently. To reflect this, the position of the system token issued on each chain is also expressed to reflect this concept.

```rust
pub struct MultiLocation {
	parents: u8,
	interior: Junctions,
}
```

- `parents`: Represents the depth of the network with different consensus, similar to the concept of a parent folder. For example, the position of token X is `parents = 1` within the Parachain, but `parents = 0` from the perspective of the Relay Chain. In InfraBlockchain, the Relay Chain always exists as a higher concept than the Parachain.

- `interior`: Represents the depth within the same network, similar to the concept of a subfolder. For example, the position of token X can be expressed as `interior = PalletInstance(0) -> GeneralIndex(0)`.

![SystemTokenId](/media/images/docs/infrablockchain/learn/protocol/system_token_id.png)

When considering the relative position of token X,

- Relay Chain:

```rust
MultiLocation {
	parents: 0,
	interior: Junctions::X2(PalletInstance(0), GeneralIndex(0))
};
```

- Parachain:

````rust
MultiLocation {
	parents: 1,
	interior: Junctions::X2(PalletInstance(1), GeneralIndex(0))
};```
````

## Features of the System Token

- The system token is a fiat-pegged token, and only the amount backed by real currency can be issued on the chain.

- The system token reflects different values for different legal currencies based on real-time exchange rate information provided by an oracle.

- Users can use the system token to pay transaction fees, and these fees are converted into a [voting form for validators](./proof-of-transaction.md).

## Wrapped System Token

![Wrapped System Token](/media/images/docs/infrablockchain/learn/protocol/wrapped.png)

The term "wrapped" refers to the system token adopted by the Relay Chain validators. To use wrapped tokens, approval from the Relay Chain governance is required. Once approved, a wrapped token is created for the network that wants to use the token, including metadata for the system token (e.g., decimal places, token symbol, token name, etc.).

### Foreign Asset

After the registration for wrapped tokens is completed, the token received through XCM is stored as a `ForeignAsset`. `ForeignAsset` is a module that manages external assets, not the token it created, and manages the token with `SystemTokenId(=MultiLocation)` as the ID. This module can accept any external asset, not just the system token.

## Oracle

![Oracle Node](/media/images/docs/infrablockchain/learn/protocol/oracle.png)

The system token, as a fiat-based token, has different values based on the exchange rate. The exchange rate information is off-chain data provided by an oracle selected by the Relay Chain governance. At specific intervals, it provides exchange rate information for the system token legal currency in circulation, and each time, it recalculates and reflects the [weight of each system token](./transaction-fee.md#system-token-weight).

The oracle node in the _Asset Hub_, the system chain of InfraBlockchain, periodically (usually daily) updates the exchange rate information for the token against the oracle node, and this information is then transmitted to the Relay Chain. The exchange rate is updated each time it is updated for the currency used by the chain. All matters related to the system token are managed in the Relay Chain's [`SystemTokenManager`](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/master/infrablockchain/runtime/parachains/src/system_token_manager.rs) pallet.

## Next Steps

eps

- [Proof of Transaction (PoT)](./proof-of-transaction.md)
- [SystemTokenManager Module](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/master/infrablockchain/runtime/parachains/src/system_token_manager.rs)

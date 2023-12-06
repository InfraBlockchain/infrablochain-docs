---
title: System Parachains
description:
keywords:
  - Asset Hub
  - Bridge Hub
---

**_System Parachain_** includes essential **_InfraBlockchain_** protocol functionalities, but these functionalities reside in the parachains rather than Relay Chain. By hosting core protocol logic in parachains instead of Relay Chain, **_InfraBlockchain_** utilizes its self-hosting capability through parallel execution, a unique extension technology. The System Parachain removes transactions performing protocol functions from Relay Chain, allowing the [_blockspace_](../architecture.md#block-space) to be more dedicated to its core functionalities (e.g., [parachain validity verification](../architecture.md#parachain-state-transition)).

The governance of the System Parachain is conducted by Relay Chain and serves as a special parachain for the entire **_InfraBlockchain_** network. The System Parachains within **_InfraBlockchain_** include:

- [Asset Hub](#asset-hub): A system chain governing functionalities related to tokens (e.g., [System Token](../../protocol/system-token.md)).
- [Bridge Hub](#bridge-hub): A system chain overseeing bridges between different networks.

## Asset Hub

### Overview

**_Asset Hub_** is a system chain within the **_InfraBlockchain_** ecosystem responsible for governing functionalities related to tokens (e.g., [System Token](../../protocol/system-token.md)). It assists token issuers (e.g., System Token issuer) in generating tokens within the **_InfraBlockchain_** network, tracking the total token supply (including tokens[ transferred to other parachains](../../../tutorials/build/transfer-assets-with-xcm.md)). Additionally, it facilitates the creation, minting, and burning of on-chain assets, managing on-chain assets deposited and created in accordance with [Uniques Pallet](https://github.com/InfraBlockchain/infrablockspace-sdk/tree/822bc6c9706774a98122eb432f412b871a98a4bd/substrate/frame/uniques) and the new [NFTs Pallet](https://github.com/InfraBlockchain/infrablockspace-sdk/tree/822bc6c9706774a98122eb432f412b871a98a4bd/substrate/frame/nfts).

Since the token-related logic is executed directly in the runtime rather than through smart contracts, the cost of executing transactions in parachains is approximately 1/10 of that on Relay Chain. This low fee level implies that **Asset Hub is well-suited for handling system token balances and transfers, as well as managing on-chain assets across the network.**

## Bridge Hub

**_Bridge Hub_** is a system chain responsible for functionalities related to bridging in the context of building a Federated Multi-BlockChain, where multiple **_InfraBlockchain_** ecosystems are interconnected.

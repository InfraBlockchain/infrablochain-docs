---
title: InfraBlockchain Architecture
description: This document covers the general aspects of multi-chain architecture.
keywords:
  - multi chain
  - relay chain
  - parachain 
---

![](/media/images/docs/infrablockchain/learn/architecture/relay-chain.png)

## Block Space

**Block Space** is an abstract space where blocks are finalized and executed. It is a crucial concept necessary for building decentralized systems and a cornerstone of blockchain existence. From the perspective of applications using blockchain, it enables continuous operation.

Ethereum was the first blockchain to provide block space. It introduced the concept of gas metering, where the resources available within a block space are limited based on the usage of the blockchain's virtual machine and blockchain resources. Building upon this, various blockchain protocols using block space emerged (e.g., Polkadot, Solana), each with its unique features.

Block Space parallelly executes multiple blocks generated from different blockchains (e.g., infrastructure blockchains or smart contracts), much like a multi-threaded approach resembling modern computer CPUs. The core of this is the Execution Core, capable of executing one block per core (Polkadot is developing Agile Core Time, allowing one core to be split into several). Validators in ***InfraBlockchain*** form shared security, providing block space and serving as service providers offering consistent quality block space.

For ***InfraBlockchain***, an enterprise blockchain for institutions and public agencies, providing secure block space is essential, and there exists a block space allocation mechanism for this purpose.

## InfraRelayChain

***InfraRelayChain*** is the central element of ***InfraBlockchain*** focusing on interoperability between different blockchains, scalability, and security through shared security. Its main role is to enable seamless connections between different blockchains. Thus, basic functions such as executing smart contracts or token transfers are delegated to parachains, working with minimal functionality (e.g., parachain block verification) for fundamental operations.

## Parachain

*Parachain* is a blockchain specialized for specific services (e.g., DID, STO) and gets its name from being a "chain" parallel to other chains in the relay chain. Parachains are generally blockchains, but they can take other forms (e.g., smart contracts). Parachains have the following characteristics:

- State Transition Machine: *Parachains* are deterministic state machines with individual states.

- Collator: Collators collect transactions from various parachain nodes, create blocks, and deliver them to relay chain nodes. Collators act as full nodes for parachains and perform the role of light nodes for Relay Chain.

### Parachain State Transition
Each parachain has a unique logic called the State Transition Function capable of changing states. These functions are stored in **WebAssembly (WASM)** format on the InfraRelayChain. Thus, all parachains connected to the Relay Chain are verified by Relay Chain validators.

### Proof-of-Validity (PoV)
*Parachains* submit a special block called ***Proof-of-Validity (PoV)*** for each state change in the form of a block. Relay Chain validators verify this PoV by comparing the final value obtained by executing the block using the parachain's state transition function with the value provided by the parachain. The maximum size for the PoV that ***InfraRelayChain*** can receive is currently 5MB.

```rust
#[derive(codec::Encode, codec::Decode, Clone)]
pub struct ParachainBlockData<B: BlockT> {
	/// The header of the parachain block.
	header: B::Header,
	/// The extrinsics of the parachain block.
	extrinsics: sp_std::vec::Vec<B::Extrinsic>,
	/// The data that is required to emulate the storage accesses executed by all extrinsics.
	storage_proof: sp_trie::CompactProof,
}

type BlockData = Vec<u8>;
pub struct PoV(BlockData);

// Create PoV
let pov = parachain_block_data.encode();
```

#### Header
Header of the parachain block

#### Extrinsics
State transition functions of the parachain block

#### Storage Proof
List of storage changes during state transition

## Shared Security

![](/media/images/docs/infrablockchain/learn/architecture/shared-security.png)

The most significant advantage of becoming a parachain is Shared Security. Each parachain can ensure block creation and finalization safety without independently configuring validators, thanks to relay chain validators. This allows parachains to focus solely on business logic tailored to each service without worrying about sensitive elements.


## Parachain Protocol

![](/media/images/docs/infrablockchain/learn/architecture/parachain-protocol.png)

The goal of the Parachain protocol is to perform processes iteratively in parallel, from parachain block creation to inclusion in the relay chain and approval. This protocol maintains strong security while efficiently operating parachains. Roles in the Parachain protocol are divided into:

- Validator: Validators verify the PoV received from the parachain and ensure availability for a certain period.

- Collator: Collators create PoV and deliver it to validators.

The Parachain protocol can be summarized into two main stages:

- **Inclusion Pipeline**: 1) Collators send blocks, 2) Validators verify them, 3) If they receive a sufficient number of valid blocks, they 4) back them, and the relay chain 5) includes them. However, this stage is not fully approved (pending approval).

- **Approval Process**: After the inclusion stage, randomly selected relay chain validators who did not participate in the process validate the parachain block. Once approved (approved), the parachain block is finalized.



## Further Readings

- [Blockspace over Blockchains](https://www.rob.tech/blog/polkadot-blockspace-over-blockchains/)

<!-- - [Agile Core Time]() -->
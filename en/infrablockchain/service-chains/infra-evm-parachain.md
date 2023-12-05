---
title: InfraBlockchain EVM
description: This document explains the overall content related to a blockchain compatible with EVM (Ethereum Virtual Machine).
keywords:
  - parachain
  - evm
---

## Before you begin

Before you begin, Make sure you have the following:

<!-- 
  When the document containing this content is created, please link to that document.
-->

- [Building a Parachain](../tutorials/build/build-a-parachain.md)

- [Building an InfraRelayChain](../tutorials/build/build-infra-relay-chain.md)

- [Using the Zombie Network](../tutorials/test/simulate-parachains.md)

## What is EVM?

EVM stands for Ethereum Virtual Machine. EVM is a runtime environment for executing smart contracts on the Ethereum network. Its key features and functions include:

- Smart Contract Execution: The primary role of EVM is to execute smart contracts. Smart contracts are self-executing contracts with conditions written in code. Ethereum primarily uses the programming language Solidity for this purpose.

- Turing Completeness: EVM is Turing complete, meaning it can execute any algorithm given sufficient time and resources. This allows developers to write complex logic and operations within smart contracts.

- Gas: Execution of operations in EVM requires computational power, and users pay for this computation in units called "gas." Gas prevents spam and malicious activities on the Ethereum network by imposing costs on operations. The amount of gas required depends on the complexity of the operation being performed.

- Consensus: When a smart contract is executed and the state is modified in EVM, all nodes in the Ethereum network must agree on the result. This ensures consistency of the Ethereum ledger across all nodes.

- Bytecode: Developers do not write smart contracts in a language directly understood by EVM. Instead, languages like Solidity are compiled into EVM bytecode, which is what EVM executes.

- Decentralization: Unlike centralized systems, EVM runs on thousands of devices worldwide, decentralizing Ethereum. This decentralization provides Ethereum with censorship resistance and high fault tolerance.

EVM plays a crucial role in the Ethereum ecosystem, providing the infrastructure necessary to run decentralized applications (dApps) and manage cryptographic assets like Ether (ETH).

## Next steps

- [Building the Infra EVM Chain](../tutorials/service-chains/infra-evm-parachain/README.md)
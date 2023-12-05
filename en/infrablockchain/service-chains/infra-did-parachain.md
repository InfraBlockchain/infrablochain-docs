---
title: Infra DID
description: This document explains the overall content related to a blockchain specialized for DID (Decentralized Identifier).
keywords:
  - parachain
  - did
---

## Before you begin

Before you begin, Make sure you have the following:

<!-- 
  When the document containing this content is created, please link to that document.
-->

- [Building a Parachain](../tutorials/build/build-a-parachain.md)

- [Building an InfraRelayChain](../tutorials/build/build-infra-relay-chain.md)

- [Using the Zombie Network](../tutorials/test/simulate-parachains.md)

## What is DID?

**DID** stands for Decentralized Identifier, representing a new type of identifier that is created, owned, and controlled by the subject of digital identity.

![did-method](/media/images/docs/infrablockchain/service-chains/did-method.png)

DID is entirely under the control of the DID subject, independent of centralized systems, authorities, or intermediaries. This capability is made possible through blockchain and distributed ledger technologies.

The key features and benefits of DID include:

- Decentralization: Unlike traditional identifiers issued and managed by centralized systems, DID is issued and managed within a distributed network.

- Self-Sovereign: DID subjects have complete control over their identifiers. They can create, update, or delete their DID without requiring permission from any authority.

- Security: DID inherits cryptographic security properties from blockchain or distributed ledger platforms. Additionally, DID works alongside DPKI (Decentralized Public Key Infrastructure) to securely maintain authentication and communication.

- Interoperability: DID is designed to be interoperable across various systems and networks. This means that a DID generated in one network can be used and recognized in another.

## Infra DID

One of the parachains in the InfraBlockchain, the *Infra DID Parachain*, provides a DID system, and the DIDs offered by the *Infra DID Parachain* are referred to as *Infra DID*.

![infra-did-method](/media/images/docs/infrablockchain/service-chains/infra-did-method.png)

*Infra DID Parachain* includes various palettes based on its functionality.

![infra-did-pallet](/media/images/docs/infrablockchain/service-chains/infra-did-pallet.png)

There is an npm library called [infra-did-js](https://github.com/InfraBlockchain/infra-did-js/tree/main) that can communicate with Infra DID parachains. You can use it to work with Infra DID in node.js-based systems.

## Next steps

- [Building the Infra DID Chain](../tutorials/service-chains/infra-did-parachain/)

- [infra-did-js Repository](https://github.com/InfraBlockchain/infra-did-js/tree/main)
---
title: Registering System Tokens and Trying Transaction Fee Payments
description: This tutorial covers the process of registering tokens as system tokens and using them as transaction fees.
keywords:
  - Assets
  - System token manager
  - Governance
  - Transaction fee
---

This tutorial will teach you how to register tokens issued in a multi-chain environment as system tokens and use them as gas fees.

## Tutorial Objectives

By completing this tutorial, you will be able to:

- Build a `RelayChain <> Parachain` test network using a zombie network.
- Propose governance to register tokens issued in a parachain as system tokens in the relay chain.
- After governance approval, pay gas fees with the system tokens issued in the parachain.

## Before You Start

Before you start, make sure to:

- [Simulate a Parachain Network Using a Zombie Network](./test/simulate-parachains.md)

- Complete the preparation to pay transaction fees with the system token through the [System Token Registration Process](./how-to-interact-with-system-token.md).

## Using System Tokens as Gas Fees

Let's call the `transfer` transaction of Assets as follows:

![parachain_asset_transfer](/media/images/docs/infrablockchain/tutorials/parachain_asset_transfer.png)

Specify the token to be used as the transaction fee.

![send_transaction](/media/images/docs/infrablockchain/tutorials/send_transaction.png)

When the transaction is executed successfully, you can see the event that the transaction fee was paid with the system token as follows:

![tx_fee_paid_event](/media/images/docs/infrablockchain/tutorials/tx_fee_paid_event.png)

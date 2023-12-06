---
title: Offchain workers
description: Quick reference guides that illustrate how to use offchain workers.
keywords:
---

The _How-to_ guides in the _Offchain workers_ category illustrate common use cases for offchain operations.

- [Make offchain HTTP requests](./offchain-http-requests.md)
- [Offchain local storage](./offchain-local-storage.md)
- [Offchain indexing](./offchain-indexing.md)

It is important to note that offchain storage is separate from on-chain storage.
You can't send data collected or processed by an offchain worker directly to on-chain storage.
To store any data collected or processed by an offchain worker—that is, to modify the state of the chain—you must enable the offchain worker to send transactions that modify the on-chain storage system.
For examples of how to prepare an offchain worker to send transactions, see [Add offchain workers](../../../tutorials/build-application-logic/add-offchain-workers.md).

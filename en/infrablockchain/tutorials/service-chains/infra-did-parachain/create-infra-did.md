---
title: Creating InfraDID
description: This tutorial explains how to create a DID on InfraDID parachain.
keywords:
  - Parachain
  - DID
---

## Before You Begin

Before you start, make sure to do the following:

- [InfraDID parachain](../../../service-chains/infra-did-parachain.md)

## Creating an InfraDID

Creating an InfraDID doesn't require communication with the blockchain itself.

![infra-did-method](/media/images/docs/infrablockchain/service-chains/infra-did-method.png)

The format of an InfraDID involves composing the `DID method-specific identifier` part as an SS58 address. In other words, if you structure it as `did:infra:{networkID}:{SS58 address}`, it becomes your own DID. You can use the public key and its corresponding private key, which can be obtained from the SS58 address, as the key pair you can use by default.

If you want to register multiple public keys or other data using the same DID or want to use other data, you'll need to go through the process of registering the DID on the chain.

## Registering on the InfraDID Chain

To register InfraDID on the chain, follow these steps:

1. Access the [\*InfraBlockchain Explorer](https://portal.infrablockspace.net) and follow the steps below:

   - Go to `Developer` - `Extrinsics` - select `newOnchain` extrinsic of the `didModule` palette.

     Configure as shown below and execute the extrinsic.

     ![new-onchain](/media/images/docs/infrablockchain/tutorials/service-chains/infra-did-parachain/new-onchain.png)

2. Check the events to ensure that the DID was created successfully.

   ![new-onchain-success](/media/images/docs/infrablockchain/tutorials/service-chains/infra-did-parachain/new-onchain-success.png)

3. Query the storage to confirm that the DID was registered correctly on the chain.

   ![new-onchain-storage](/media/images/docs/infrablockchain/tutorials/service-chains/infra-did-parachain/new-onchain-storage.png)

## Creating and Registering InfraDID using the infra-did-js Library

You can create InfraDID using the [\*InfraBlockchain Explorer](https://portal.infrablockspace.net), but you can also create InfraDID using the `infra-did-js` JavaScript library.

To create InfraDID using the `infra-did-js` library, follow these steps:

1. Install the [`infra-did-js`](https://github.com/InfraBlockchain/infra-did-js) library.

   ```shell
   yarn add infra-did-js
   ```

2. Write the code as shown below to set up the basic configuration to access InfraDID chain using the `infra-did-js` library.

   ```typescript
   import { InfraSS58, CRYPTO_INFO } from "infra-did-js";

   const txfeePaterAccountKeyPair = await InfraSS58.getKeyPairFromUri(
     "//Alice",
     "sr25519"
   );
   const confBlockchainNetwork = {
     networkId: "space",
     address: "ws://localhost:9944",
     // seed or keyPair required
     txfeePayerAccountKeyPair,
     // or txfeePayerAccountSeed: 'TX_FEE_PAYER_ACCOUNT_SEED'
   };
   const conf = {
     ...confBlockchainNetwork,
     did: "did:infra:space:5CRV5zBdAhBALnXiBSWZWjca3rSREBg87GJ6UY9i2A7y1rCs",
     // seed or keyPair required
     seed: "DID_SEED",
     // keyPair: keyPair,
     controllerDID:
       "did:infra:space:5HdJprb8NhaJsGASLBKGQ1bkKkvaZDaK1FxTbJRXNShFuqgY",
     controllerSeed: "DID_CONTROLLER_SEED",
     // or controllerKeyPair: controllerKeyPair
   };
   const infraApi = await InfraSS58.createAsync(conf);
   ```

3. Write the code to create an InfraDID.

   ```typescript
   DIDSet = await InfraSS58.createNewSS58DIDSet(networkId);
   console.log({ DIDSet });
   ```

   The `console.log` output will be as follows:

   ```json
   {
     "DIDSet": {
       "did": "did:infra:space:5Cq2Za1Z4HJx5eTvxT5iFyXZLM1XTwVZSafQsEuK4ujNKJEF"
       // ... other information ...
     }
   }
   ```

4. Write the code to register the created InfraDID on the chain.

   ```typescript
   // Register DID
   await infraApi.didModule.registerOnchain();
   ```

## Next Steps

- [Registering Additional Keys to InfraDID](./add-keys.md)

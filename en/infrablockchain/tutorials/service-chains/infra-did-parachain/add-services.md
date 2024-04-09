---
title: Registering Service Endpoints with InfraDID
description: This tutorial explains how to add service endpoints to a DID registered with InfraDID.
keywords:
  - Parachain
  - DID
---

## Before you begin

Before you begin, Make sure you have the following:

- [Creating InfraDID](./create-infra-did.md)

## Adding Service Endpoints to InfraDID

To make a list of service endpoints available for a specific DID and store it on the chain, making it public for others to inspect the Document of a specific DID, follow these steps:

1. Visit the [InfraBlockchain Explorer](https://portal.infrablockspace.net) and follow the steps below.

- Go to `Developer` - `Extrinsic` and select the `addServices` extrinsic of the `didModule` palette.

  Configure it as shown below and trigger the extrinsic:

  ![add-services](../../../../../media/images/docs/infrablockchain/tutorials/service-chains/infra-did-parachain/add-services.png)

## Removing Service Endpoints from InfraDID

To remove service endpoints from InfraDID, follow these steps:

1. Visit the [InfraBlockchain Explorer](https://portal.infrablockspace.net) and follow the steps below.

- Go to `Developer` - `Extrinsic` and select the `removeServices` extrinsic of the `didModule` palette.

  Configure it as shown below and trigger the extrinsic:

  ![remove-services](../../../../../media/images/docs/infrablockchain/tutorials/service-chains/infra-did-parachain/remove-services.png)

## Adding and Removing Service Endpoints from InfraDID Using the `infra-did-js` Library

To add and remove service endpoints from InfraDID using the `infra-did-js` library:

1. Install the [`infra-did-js`](https://github.com/InfraBlockchain/infra-did-js) library.

   ```shell
   yarn add infra-did-js
   ```

2. Write code to set up the basic configuration code for accessing InfraDID chain, as shown below:

   ```typescript
   import  {InfraSS58, CRYPTO_INFO} from 'infra-did-js';

   const txfeePaterAccountKeyPair = await InfraSS58.getKeyPairFromUri('//Alice', 'sr25519');
   const confBlockchainNetwork = {
     networkId: 'space',
     address: 'ws://localhost:9944',
     // seed or keyPair required
     txfeePayerAccountKeyPair,
     // or txfeePayerAccountSeed: 'TX_FEE_PAYER_ACCOUNT_SEED'
   };
   const conf = {
     ...confBlockchainNetwork,
     did: 'did:infra:space:5CRV5zBdAhBALnXiBSWZWjca3rSREBg87GJ6UY9i2A7y1rCs',
     // seed or keyPair required
     seed: 'DID_SEED',
     // keyPair: keyPair,
     controllerDID: 'did:infra:space:5HdJprb8NhaJsGASLBKGQ1bkKkvaZDaK1FxTbJRXNShFuqgY'
     controllerSeed: 'DID_CONTROLLER_SEED',
     // or controllerKeyPair: controllerKeyPair
   };
   const infraApi = await InfraSS58.createAsync(conf);
   ```

3. Write code to add service endpoints to InfraDID:

   ```typescript
   // Add Service Endpoint
   /*
   addServiceEndpoint(
     originsTexts: string[],
     endpointType?: ServiceEndpointType,
     endpointIdText?: string,
   )
   */
   await infraApi.didModule.addServiceEndpoint(SOME_SERVICE_ENDPOINT_URLS);
   ```

4. Alternatively, write code to remove service endpoints from InfraDID:

   ```typescript
   // Remove Service Endpoint
   /*
   removeServiceEndpoint(
     endpointIdText?: string
   )
   */
   await infraApi.didModule.removeServiceEndpoint();
   ```

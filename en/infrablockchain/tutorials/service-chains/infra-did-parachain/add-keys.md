---
Title: Adding Public Keys to InfraBlockchain DID
Description: This tutorial explains how to add public keys to a DID registered on the InfraBlockchain DID parachain.
Keywords:
  - parachain
  - DID
---

## Before you begin

Before you begin, Make sure you have the following:

- [Creating *InfraBlockchain DID*](./create-infra-did.md)

## Adding Public Keys to *InfraBlockchain DID*

*InfraBlockchain DID* registered on the chain allows management of various items, one of which is the list of public keys. By storing a list of public keys that a specific DID can use on the chain, it makes it public for other users to inspect the Document of a specific DID.

To add public keys to *InfraBlockchain DID*, follow these steps:

1. Visit the [*InfraBlockchain Explorer*](https://portal.infrablockspace.net) and follow the steps below.

  - Go to `Developer` - `Extrinsic` and select the `addKeys` extrinsic of the `didModule` palette.

    Configure it as shown below and trigger the extrinsic:

    ![add-keys](/media/images/docs/infrablockchain/tutorials/service-chains/infra-did-parachain/add-keys.png)

    Encode call data:
    ```shell
    0x3404d43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d04012e7222343997d83b3571b176837092986630c3de8fcdc91ba74d31bbb11c1181000000000000d43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d0100000001f62b5a39b0fceeb51d16b2a189da0e73f15f384497dcdd63a1f9a7c626f7649941b4a8cb66fed18c65574117a2f1ed607591dda6f6ec3bdfff40a09a71e202803
    ```

## Removing Public Keys from *InfraBlockchain DID*

To remove public keys from *InfraBlockchain DID*, follow these steps:

1. Visit the [*InfraBlockchain Explorer*](https://portal.infrablockspace.net) and follow the steps below.

  - Go to `Developer` - `Extrinsic` and select the `removeKeys` extrinsic of the `didModule` palette.

    Configure it as shown below and trigger the extrinsic:

    ![remove-keys](/media/images/docs/infrablockchain/tutorials/service-chains/infra-did-parachain/remove-keys.png)

## Adding and Removing Public Keys from InfraBlockchain DID Using the infra-did-js Library

To add and remove public keys from InfraBlockchain DID using the `infra-did-js` library:

1. Install the [`infra-did-js`](https://github.com/InfraBlockchain/infra-did-js) library.

    ```shell
    yarn add infra-did-js
    ```

2. Write code to set up the basic configuration code for accessing the InfraBlockchain DID chain, as shown below:

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

3. Write code to add public keys to InfraBlockchain DID:

    ```typescript
    // Add keys
    await infraApi.didModule.addKeys(SOME_DID_KEY)
    ```

4. Alternatively, write code to remove public keys from InfraBlockchain DID:

    ```typescript
    // Remove Keys
    await infraApi.didModule.removeKeys(DID_KEY_IDS)
    ```

## Next stepss

- [Registering Service Endpoints with *InfraBlockchain DID*](./add-services.md)

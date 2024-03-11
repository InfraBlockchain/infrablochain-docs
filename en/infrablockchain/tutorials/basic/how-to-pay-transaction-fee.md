---
title: Registering and Using System Token as Transaction Fees
description: This tutorial covers the entire process from token creation to governance for registering System Token, and using System Token for transactions.
keywords:
  - Assets
  - System token manager
  - Governance
  - Transaction fee
---

This tutorial will teach you how to register tokens issued in a multi-chain environment as System Token and use them as gas fees.

## Tutorial Goals

By completing this tutorial, you will be able to:

- Set up a `Relay Chain <> Parachain` test network using ZombieNet.
- Propose governance on the Relaychain to register a token issued on a Parachain as a system token.
- Pay gas fees with System Token issued on the Parachain after governance approval.

## Before You Start

Before starting, ensure you have:

- [Simulating a Parachain Network using ZombieNet](./test/simulate-parachains.md)

## Creating and Issuing Tokens on a Parachain

Before registering a system token on the Relay Chain, you need to create a token on the Parachain. _You must have test System Token or System Token for the gas fee required for Parachain token creation and issuance._

The procedure for creating and issuing tokens on a Parachain is as follows:

1. Open the [InfraBlockchain Explorer](https://portal.infrablockspace.net/#/explorer/) and connect to the Parachain endpoint.

2. In [Developer > Extrinsic], select [create](https://github.com/InfraBlockchain/infrablockspace-sdk/blob/599828207489db1d2b4633473c15c9be9dd97253/substrate/frame/assets/src/lib.rs#L625) from the [Assets](https://github.com/InfraBlockchain/infrablockspace-sdk/tree/master/substrate/frame/assets) pallet to create a token.

   - id: A value to identify the token. Here, we insert 1.
   - admin: Token management account. Here, we specify `alice`.
   - minBalance: The minimum token deposit unit. Here, we insert 1.

   ![Creating a Token](/media/images/docs/infrablockchain/tutorials/create_token.png)

3. Immediately select [mint](https://github.com/InfraBlockchain/infrablockspace-sdk/blob/599828207489db1d2b4633473c15c9be9dd97253/substrate/frame/assets/src/lib.rs#L801C7-L801C14) from the Assets pallet to issue the token.

   - id: A value to identify the token. Here, insert the previously created token identifier.
   - beneficiary: The account to receive the issued tokens. Here, we issue them to `alice`.
   - amount: The amount of tokens to issue. Here, insert 1000000000000.

   ![Issuing Tokens](/media/images/docs/infrablockchain/tutorials/mint_token.png)

## Registering `register_system_token` Governance on Relay Chain

### Preparing preimage for submitting `register_system_token` to governance

1. Open the [InfraBlockchain Explorer](https://portal.infrablockspace.net/#/explorer/) and connect to the Relay Chain endpoint.

2. To put `register_system_token` to a governance vote, you must first register it in the preimage.

3. Press [Governance > Preimages > Add preimage].
   (Anyone can register a preimage.)

![Preimage Button](/media/images/docs/infrablockchain/tutorials/preimage_button.png)

4. Create a preimage for `register_system_token` based on the token information created in the Parachain.

![Registering System Token (continued below)](/media/images/docs/infrablockchain/tutorials/register_system_token1.png)

![Registering System Token](/media/images/docs/infrablockchain/tutorials/register_system_token2.png)

- `systemTokenType`: The type of system token. For more details on system token types, see [here](../learn/protocol/system-token.md). Here, select `Original`.
  ```text
  - paraId: Identifier of the Parachain where the token was created
  - palletId: Identifier of the *Assets pallet* where the token was created
  - assetId: Token identifier
  ```
- `systemTokenWeight`: The weight of the token. Here, set to 1_000_000(default).
- `wrappedForRelayChain`: Wrapped system token for use on the Relay Chain.
- `systemTokenMetadata`: Metadata for System Token.
- `assetMetadata`: Metadata for the token.

5. It will be registered successfully with a hash value as follows.

![Preimage Result](/media/images/docs/infrablockchain/tutorials/preimage_result.png)

### Registering the submitted preimage to governance

Once the _preimage_ for `register_system_token` is successfully registered, it can now be registered in governance.

1. Open the [InfraBlockchain Explorer](https://portal.infrablockspace.net/#/explorer/) and connect to the Relay Chain endpoint.

2. In the Explorer tab, click [Developer > Extrinsic], then use [propose](https://github.com/InfraBlockchain/infrablockspace-sdk/blob/599828207489db1d2b4633473c15c9be9dd97253/substrate/frame/collective/src/lib.rs#L519) from the [Council](https://github.com/InfraBlockchain/infrablockspace-sdk/tree/master/substrate/frame/collective) pallet to submit a motion on the registered preimage to the council governance.
   (Note: Only Relaychain validators can submit and vote on council propose.)

```text
- threshold: Minimum number of votes to conclude the motion
- Legacy-hash: The hash of the registered preimage
- lengthBound: Byte limit of the preimage's length.
```

![Governance Proposal](/media/images/docs/infrablockchain/tutorials/council_propose.png)

## Passing the Governance

If successfully registered in governance, the motion will appear in [Governance > Council > Motion] as shown below.

![Governance Voting](/media/images/docs/infrablockchain/tutorials/governance_voting.png)

In the screen above, click Vote and vote with the validators forming the Council (alice_stash, bob_stash).

If the threshold number of people vote, the vote can be immediately concluded, and if the quorum agrees (60% or more), the motion will be enacted immediately.

![Vote Close](/media/images/docs/infrablockchain/tutorials/vote_close.png)

The motion is successfully passed, and you can see that `sufficient` has changed to `true` in the Parachain.

![Motion Enactment](/media/images/docs/infrablockchain/tutorials/enact_motion.png)

![Change in Parachain Token Status](/media/images/docs/infrablockchain/tutorials/parachain_sufficient_true.png)

## Using System Token as Gas Fee

Now, the token initially issued on the Parachain has become a system token,
and you can pay transaction gas fees with this token.

Let's call the `transfer` transaction of the _Assets_ as follows:

![Parachain Asset Transfer](/media/images/docs/infrablockchain/tutorials/parachain_asset_transfer.png)

In the extra information of the transaction, if you click the toggle for `include an optional asset id`, a field appears where you can enter the registered system token information.

![System Token](/media/images/docs/infrablockchain/tutorials/system_token_id.png)

You can confirm that this system token can be used to pay the gas fee.

![System Token Gas Fee](/media/images/docs/infrablockchain/tutorials/system_token_paid.png)

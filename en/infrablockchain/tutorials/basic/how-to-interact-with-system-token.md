---
title: Registering and Using System Tokens
description: This tutorial covers the process of registering and using system tokens in InfraBlockchain.
keywords:
  - System Token
  - Registration
---

**_InfraBlockchain_** provides a differentiated blockchain system that uses system tokens as gas fees.
The following content is organized to help developers easily understand and develop the process of registering, using, and disposing of system tokens.
You can practice this content in [how-to-pay-transaction-fee](./how-to-pay-transaction-fee.md).

## Bootstrap Stage

Infra Blockchain Runtime has two runtime states. When the network is first started, it is in the `Bootstrap` stage, where the system token is not yet registered. There are limitations on the transactions that can be performed in the bootstrap stage.

List of transactions that can be performed:

- `Assets::create`: Transaction to create a token
- `Assets::set_metadata`: Transaction to set metadata for the created token
- `Assets::mint`: Transaction to issue a token (at least one token that can be transacted must exist for registration)
- `InfraParaCore::request_register_system_token`: Transaction to request registration of the system token

_Notation `_::_`: The part before `::` is the palette name, and the part after is the transaction name within the palette._

## Registering System Tokens

### Registration Request Process

- **Token Creation**

  ![create_token](/media/images/docs/infrablockchain/tutorials/create_token.png)

- **Metadata Setting**

  ![set_metadata](/media/images/docs/infrablockchain/tutorials/set_metadata.png)

- **Token Issuance**

  ![mint_token](/media/images/docs/infrablockchain/tutorials/mint_token.png)

- **System Token Registration Request**

  This transaction can be called by an approved account of the relay chain or the governance of the parachain itself.

  ![request_register](/media/images/docs/infrablockchain/tutorials/request_register.png)

  Event

  ![request_register_event](/media/images/docs/infrablockchain/tutorials/request_register_event.png)

  _`exp` represents the expiration time for the registration request._

Once this event is confirmed, the information for the registration request is reflected in the relay chain, and the approval status is determined by the governance of the relay chain validators.

### Relay Chain Governance Process

**Get Exchange Rate Information**

All system tokens are based on fiat currency, and exchange rate information must be reflected for registration to proceed. First, a data request is sent from the relay chain to the oracle node side through the `requestExchangeRate` transaction. This transaction is carried out through governance.

![request_exchange_rate](/media/images/docs/infrablockchain/tutorials/request_exchange_rate.png)

This transaction is a request for exchange rate data for the system token being used as a reference against the current system token that has been registered or the system token for which registration is requested. When the transaction is executed successfully, the relay chain requests data from the oracle node, and the following event occurs on the oracle node.

![request_exchange_rate_event](/media/images/docs/infrablockchain/tutorials/request_exchange_rate_event.png)

When exchange rate data is obtained from an external source, the following event occurs after the relay chain submits the data.

![exchange_rate_submit_event](/media/images/docs/infrablockchain/tutorials/exchange_rate_submit_event.png)

When the relay chain receives the data, it updates the exchange rate data for each fiat currency, and the following event occurs.

![exchange_rate_update_event](/media/images/docs/infrablockchain/tutorials/exchange_rate_update_event.png)

**Register System Token**

**Governance Submission**: The owner of the token submits relay chain governance to register the token as a system token.

![register_system_token](/media/images/docs/infrablockchain/tutorials/register_system_token.png)

_Currency Type is an option when registering a relay chain token as a system token._

_Extended Metadata is an option when additional metadata for the system token is desired._

**Governance Vote**: Relay chain validators participate in governance, and if more than 2/3 agree, the system token is registered.

When approved and registered as a system token, the following event occurs on the chain that issued the token, and the registration as a system token is completed.

![register_event](/media/images/docs/infrablockchain/tutorials/register_event.png)

**End Bootstrap**

Once the system token registration is completed, the bootstrap stage is completed, and it is possible to pay transaction fees with the system token. The relay chain performs a transaction to end the bootstrap stage for the chain.

![end_bootstrap](/media/images/docs/infrablockchain/tutorials/end_bootstrap.png)

_dest represents the chain to end the bootstrap._

When the transaction is executed successfully, the following event occurs on the _dest_ chain:

![end_bootstrap_event](/media/images/docs/infrablockchain/tutorials/end_bootstrap_event.png)

## Using Wrapped System Tokens

**Approval to Use Wrapped System Tokens**: When you want to use system tokens as gas fees on another chain, you need to submit a governance proposal for using wrapped system tokens.

![register_wrapped](/media/images/docs/infrablockchain/tutorials/register_wrapped.png)

_The para_id option below represents the chain id to use the wrapped system token._

Once approved, you can confirm that the wrapped system token is successfully registered in the `ForeignAssets` of the chain where you want to use it.

_Even after registration is complete, the token can be used as a transaction fee by receiving it through XCM from the chain that issued the token._

## Stopping System Token Usage

- **Action in Case of Problems**: If there is a problem with the parachain or the system token, you can temporarily suspend or permanently delete the system token or the wrapped system token to prevent harm to the ecosystem.

  ![suspend](/media/images/docs/infrablockchain/tutorials/suspend.png)

  ![derigster](/media/images/docs/infrablockchain/tutorials/deregister.png)

  The suspension of the system token requires approval from governance, and once approved, the system token and the wrapped system token cannot be used as gas fees.

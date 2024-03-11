---
title: Learning System Token Management Process
description: Covers various procedures related to System Token.
keywords:
---

**_InfraBlockchain_** offers a distinctive blockchain system that uses System Token as gas fees.
Below is an outline to help developers easily understand and develop the registration, usage, and disposal processes of System Token.
Practical exercises on this content can be found in [how-to-pay-transaction-fee](./how-to-pay-transaction-fee.md).

## Bootstrap Process

- **Using Test Tokens**: During the Bootstrap phase, "test tokens" exist that can be used as gas fees.

  ![test-token](/media/images/docs/infrablockchain/tutorials/test-token.png)

- **Approval of the First System Token**: Bootstrap is completed after the first system token is approved and circulated.
- **Burning Test Tokens**: After Bootstrap completion, test tokens are burned and can no longer be used as gas fees.

## System Token Registration

- **Creating and Issuing Tokens**: Tokens are created and issued in a specific chain.

  ![create_token](/media/images/docs/infrablockchain/tutorials/create_token.png)

  ![mint_token](/media/images/docs/infrablockchain/tutorials/mint_token.png)

- **Submitting for Governance**: The token owner submits the token for registration as a system token to the relay chain governance.

  ![register_system_token1](/media/images/docs/infrablockchain/tutorials/register_system_token1.png)

- **System Token Voting**: Validators of the relay chain participate in governance, and the token is registered as a system token if more than 2/3 agree.

  ![governance_voting](/media/images/docs/infrablockspace/tutorials/governance_voting.png)

- **Using for Transaction Fees**: Once the token is registered as a system token, it can be used as transaction fees in the chain where the token was issued.

  ![parachain_sufficient_true](/media/images/docs/infrablockchain/tutorials/parachain_sufficient_true.png)

## Using System Token in Other Parachains

- **Approval for Using Wrapped System Token**: To use System Token as gas fees in another chain, a proposal for using the wrapped system token must be submitted for governance.

  ![register-wrapped](/media/images/docs/infrablockchain/tutorials/register-wrapped.png)

- **Governance Approval**: Once the proposal is approved in governance, the chain can use the Wrapped System Token as gas fees.

## Suspending and Deregistering System Token

- **Measures in Case of Issues**: If a problem occurs with a parachain or system token, to prevent harm to the ecosystem, System Token or Wrapped System Token can be temporarily suspended or permanently deleted.

  ![suspend](/media/images/docs/infrablockchain/tutorials/suspend.png)

  ![deregister](/media/images/docs/infrablockchain/tutorials/deregister.png)

- **Governance Approval Required**: The suspension of a system token requires approval from governance, and once approved, System Token and Wrapped System Token can no longer be used as gas fees.

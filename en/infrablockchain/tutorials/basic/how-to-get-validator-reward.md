---
title: How to get Validator Rewards
description: This tutorial explains how validators in the InfraBlockchain receive rewards.
keywords:
  - System Token
  - Validators
---

## Before you begin

Before you begin, make sure to do the following:

- [Register and Use System Tokens for Transaction Fees](./how-to-pay-transaction-fee.md)

## Overview

In the InfraBlockchain, validators are rewarded for their contributions to block creation and verification. The rewards received by validators are composed of transaction fees submitted by users.

![validator-reward-process](/media/images/docs/infrablockchain/tutorials/validator-reward-process.png)

This document explains how validators can receive rewards.

## Checking Validator Rewards

You can check how much reward a validator can receive by querying the `ValidatorReward` storage.

1. Visit [Portal](https://portal.infrablockspace.net).

2. Go to `Developers` > `Chain State` > `Storage`.

3. Select `validatorReward` under the `validatorRewardManager` palette.

4. Enter the address you want to check.

![storage](/media/images/docs/infrablockchain/tutorials/validator-reward-storage.png)

## Receiving Validator Rewards

Once you have confirmed that a validator is eligible for rewards through the above process, you can initiate a transaction to claim the rewards.

1. Visit [Portal](https://portal.infrablockspace.net).

2. Go to `Developers` > `Extrinsics` > `Submission`.

3. Select `claim` under the `validatorRewardManager` palette.

4. Input the appropriate values and submit the transaction.

![claim](/media/images/docs/infrablockchain/tutorials/reward-claim.png)

The `claim` extrinsic under the `validatorRewardManager` palette accepts the validator's account and a system token ID as input. 

The `claim` extrinsic can only be executed by the validator themselves.

Additionally, a validator can receive rewards for only one type of system token ID in a single extrinsic. If a validator is eligible for multiple types of tokens, they will need to initiate multiple extrinsics.
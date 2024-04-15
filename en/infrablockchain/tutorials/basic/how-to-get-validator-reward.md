---
title: How to Receive Validator Rewards
description: This tutorial explains how validators in InfraBlockchain can receive rewards.
keywords:
  - System Token
  - Validator
---

## Before You Start

Before you start, make sure to:

- [Register and use the system token as a fee](./how-to-pay-transaction-fee.md)

## Overview

In InfraBlockchain, validators are rewarded for block creation and verification. The reward for validators is composed of the fees submitted by users.

![validator-reward-process](/media/images/docs/infrablockchain/tutorials/validator-reward-process.png)

This document explains how validators receive rewards.

## Reward System

In Infra RelayChain, validators receive a portion of the transaction fees generated in each chain as a reward. Each time a reward is generated, the relay chain can confirm the following event. This event provides information about when (era) and how much reward was generated for which token (asset).

![validator_reward_event](/media/images/docs/infrablockchain/tutorials/validator_reward_event.png)

## Checking Validator Rewards

You can check how much reward a validator can receive by querying the `ValidatorManagement` storage.

1. Access the [Portal](https://portal.infrablockspace.net).

2. Go to `Developer` > `Chain State` > `Storage`.

3. Select `RewardInfo` in the `validatorManagement` palette.

4. Enter the address you want to check and confirm.

![reward_info](/media/images/docs/infrablockchain/tutorials/reward_info.png)

## Receiving Validator Rewards

If it is confirmed that a validator can receive a reward through the above process, you can generate a transaction to actually receive the reward.

1. Access the [Portal](https://portal.infrablockspace.net).

2. Go to `Developer` > `Extrinsic` > `Submission`.

3. Select `claim` in the `validatorManagement` palette.

4. Enter the appropriate value and submit the transaction.

![claim_reward](/media/images/docs/infrablockchain/tutorials/claim_reward.png)

_A validator can only receive a reward for one type of system token ID in one extrinsic. If a validator can receive multiple types of tokens, you need to generate multiple extrinsics._

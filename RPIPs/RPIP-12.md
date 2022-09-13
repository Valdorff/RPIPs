---
rpip: 12
title: Atlas Update
description: Describes the features that will be included in the upcoming Atlas update.
author: Valdorff (@Valdorff), Kane Wallmann (@kanewallmann)
discussions-to: TODO
status: Draft
type: Protocol
category: Core
created: 2022-08-31
requires: 8, 13
---

## Abstract
The team has been working on several features for the upcoming Atlas update. We are looking to
define the high level features that the community is looking for as part of this update.

As there are multiple significant features, they'll each get their own subsection within
the Specification, Implementation, and Security Consideration sections.

Atlas will implement the following high-level features:
- LEB8s
- Use queue ETH
- Include queue capacity in maximum deposit size
- Scale assignments with deposit size
- SaaS
- Minor cleanup
- Removal of total effective RPL stake
- Referral contract
- Miscellaneous

## Specification

### LEB8s
Please see [RPIP-8](RPIP-8.md). Note that a vote on RPIP-12 (this RPIP) includes RPIP-8, since that
is one of the several features specified.

### Use queue ETH

Right now the minipool queue holds a fair amount of idle ETH. Instead, that ETH could be used to 
start validators and earn rewards
- When a minipool is created
  - The minimum beacon chain deposit (1 ETH) SHALL be made
  - All remaining ETH SHALL be used to pair against the queue
  - The protocol MUST be able to calculate the amount of ETH in use from the queue; this ETH SHALL
    not be included in the ETH balance used for calculating `RocketTokenRETH.getEthValue` or
    `RocketTokenRETH.getRethValue` 

Notes:
- This means that more validators will be started than would otherwise be possible given the minted
rETH. More NOs will be earning rewards, and some of those rewards will be split to rETH holders.
  - The effect is similar, but directionally opposite to a full deposit pool; rather than being a
    small drag on rETH apr, this will provide a small boost.
- As a benefit, this means that all minipools in the queue will need 31 ETH to launch, regardless
  of how many ETH were deposited. This fact can simplify several pieces of code.

### Include queue capacity in maximum deposit size
- The maximum deposit size SHALL be the sum of:
  - The total space available in the deposit pool
  - The total capacity available in the minipool queue

### Scale assignments with deposit size
- There SHALL be a maximum number of socialized assignments (eg, deposit.assign.socializedmaximum = 2)
- There SHALL be a maximum number of assignments (eg, deposit.assign.maximum = 90)
- The total number of assignments shall be `min(deposit.assign.socializedmaximum + scaling_ct, total_eth_ct, queue_ct, deposit.assign.maximum)`, where
  - `scaling_ct` is the number of minipools next in the queue(s), whose capacity can be fully filled from the deposited ETH
  - `total_eth_ct` is the number of minipools next in queue(s), whose capacity can be fully filled from ETH the sum of deposited ETH and ETH in the deposit pool
  - `queue_ct` is the number of minipools in the queue(s)

### SaaS
Please see [RPIP-13](RPIP-13.md). Note that a vote on RPIP-12 (this RPIP) includes RPIP-13, since that
is one of the several features specified.

### Removal of total effective RPL stake
With the redstone upgrade replacing the old reward system, we no longer require the total effective RPL
stake value. It also removes the requirement for the blocker we currently have on creating/finalising
minipools while network is "not in consensus". It is a low effort change to simplify the protocol
reducing gas and improving UX with the following changes:

- Total effective RPL stake SHALL be removed from oracle DAO submissions
- All total effective RPL stake calculations and checks SHALL be removed from the smart contracts

### Commission Contract
Now that rETH no longer has a time lock, RP is able to provide partners with a way to get a
commission when they refer a mint to RP. For example, this would allow Ledger to integrate rETH
minting and take a small cut to incentivize their support.

### Miscellaneous
- The Full deposit option SHALL be removed
- The Empty deposit option SHALL be removed
- Existing queued Half and Full deposit minipools SHALL be assigned before assigning any minipools
  that are created after the Atlas smart contract is in effect
- More gas-efficient minipool deployment

## Implementation

### LEB8s
Please see [RPIP-8](RPIP-8.md). Note that a vote on RPIP-12 (this RPIP) includes RPIP-8, since that
is one of the several features specified.

### Use queue ETH

See branch [improved-queue](https://github.com/rocket-pool/rocketpool/tree/improved-queue) for WIP implementation.

### Include queue capacity in maximum deposit size

See branch [include-queue-in-capacity](https://github.com/rocket-pool/rocketpool/tree/include-queue-in-capacity) for WIP implementation.

### Scale assignments with deposit size

See branch [scaled-assignments](https://github.com/rocket-pool/rocketpool/tree/scaled-assignments) for WIP implementation.

### SaaS
Please see [RPIP-13](RPIP-13.md). Note that a vote on RPIP-12 (this RPIP) includes RPIP-13, since that
is one of the several features specified.

### Removal of total effective RPL stake

See branch [redstone-cleanup](https://github.com/rocket-pool/rocketpool/tree/redstone-cleanup) for WIP implementation.

- Remove `_effectiveRplStake` argument from `rocketNetworkPrices.submitPrices` and `rocketNetworkPrices.executePrices` methods
- Remove `inConsensus` method from `rocketNetworkPrices`
- Remove `updateTotalEffectiveRPLStake` method from `rocketMinipoolManager`
- Remove calls to `updateTotalEffectiveRPLStake` from `decrementNodeStakingMinipoolCount` and `incrementNodeStakingMinipoolCount`
- Remove calls to `updateTotalEffectiveRPLStake` in methods `slashRPL`, `withdrawRPL`, `_stakeRPL` in `rocketNodeStaking`
- Remove `calculateTotalEffectiveRPLStake` and `getTotalEffectiveRPLStake` in `rocketNodeStaking`
- Update smartnode software to no longer calculate and submit the value for `_effectiveRplStake`

### Referral Contract
The team has a high level design in hand for this, which had been blocked by the rETH time lock.
Details to follow.

### Miscellaneous

See branch [efficient-minipool-delegate](https://github.com/rocket-pool/rocketpool/tree/efficient-minipool-delegate) for WIP implementation.

## Security Considerations

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

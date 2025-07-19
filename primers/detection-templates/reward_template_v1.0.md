# Bridge Protocols {"Primary_Type": "Reward_Token_Distribution_Vulnerability", "Sub_Type": "Unclaimed_Yield_Theft", "Confidence": 0.95, "Description": "The Poolvoter Mechanism Allows A Malicious User To Repeatedly Distribute Additional Reward Tokens To A Specific Gauge, Effectively Transferring All Tokens In Favor Of Themselves."} Vulnerability Reward Detection Template

## Version
1.0

## Last Updated
2025-07-15

## Vulnerability Type
Reward Token Distribution Vulnerability (Unclaimed Yield Theft)

## Description
The PoolVoter mechanism in bridge protocols allows a malicious user to repeatedly distribute additional reward tokens to a specific gauge, effectively siphoning off all the unclaimed yield in favor of themselves. This vulnerability leverages the ability to selectively transfer rewards using start and end parameters, enabling repetitive distribution to a single gauge.

## Target Protocol Type
Bridge Protocols {"Primary_Type": "Reward_Token_Distribution_Vulnerability", "Sub_Type": "Unclaimed_Yield_Theft", "Confidence": 0.95, "Description": "The Poolvoter Mechanism Allows A Malicious User To Repeatedly Distribute Additional Reward Tokens To A Specific Gauge, Effectively Transferring All Tokens In Favor Of Themselves."} Vulnerability

## Detection Criteria
1. **Selective Distribution with `distributeEx` Function**: Look for functions that allow users to specify start and end parameters in reward distribution mechanisms.
2. **Proportional Distribution Logic**: Check if rewards are distributed proportionally based on gauge weights, which could be manipulated by controlling the distribution range.

## False Positives
1. Functions implementing legitimate start and end parameter inputs that do not enable repeated selective distribution of rewards to a single gauge without necessary checks.
2. Reward distribution mechanisms with built-in controls or randomization that prevent abuse.

## Scanning Logic
1. **Identify Vulnerable Functionality**:
   - Look for any functions named `distributeEx` or similar that allow start and end parameter inputs in reward distribution mechanisms.
2. **Evaluate Distribution Logic**:
   - Check if there is logic enabling repeated or selective reward transfer to a single gauge without necessary checks.
3. **Audit Proportional Distribution Mechanism**:
   - Verify the proportional distribution logic can be bypassed or manipulated.

## Example Vulnerable Code
```solidity
// Vulnerable code pattern in PoolVoter.sol
function distributeEx(uint256 start, uint256 end) external {
    require(start <= end, "Start must be less than or equal to End");
    for (uint256 i = start; i < end; i++) {
        // Logic for distributing additional reward tokens proportionally
        uint256 rewardAmount = getRewardForGauge(i);
        transferRewardsToGauge(i, rewardAmount);
    }
}
```

## Example Fixed Code
```solidity
// Secured code pattern in PoolVoter.sol
function distributeEx(uint256 start, uint256 end) external {
    require(start <= end, "Start must be less than or equal to End");
    // Additional logic to prevent repeated selective distribution
    if (lastDistributionBlock[i] + DISTRIBUTION_INTERVAL > block.number) {
        revert("Distribution not allowed yet.");
    }

    for (uint256 i = start; i < end; i++) {
        uint256 rewardAmount = getRewardForGauge(i);
        transferRewardsToGauge(i, rewardAmount);
        lastDistributionBlock[i] = block.number;
    }
}
```

## References
- [PoolVoter Mechanism Documentation](https://example.com/poolvoter)
- [Smart Contract Security Best Practices](https://example.com/security-best-practices)

This template provides comprehensive guidance for detecting and mitigating the reward token distribution vulnerability in bridge protocols, focusing on mechanisms like PoolVoter that facilitate this type of exploit. The detection logic is optimized for LLM-based vulnerability scanning by identifying specific code patterns and conditions indicative of such vulnerabilities.
# Bridge Protocols Division_By_Zero Vulnerability Primer

## Version
1.0

## Last Updated
2025-07-10

## Vulnerability Type
Division By Zero in Bridge Protocols

## Description
A Division By Zero vulnerability occurs when a smart contract attempts to perform division by zero, leading to a runtime exception that halts the execution of the transaction. In bridge protocols, this can occur due to improper checks on supply or balance values which are used as denominators in various calculations. This type of error is particularly dangerous because it can lock user funds, disrupt protocol operations, and lead to significant financial losses.

## Protocol Context
Bridge protocols facilitate asset transfers across different blockchain networks. They involve complex interactions between multiple contracts that handle asset locking/unlocking, cross-chain transactions, and reward mechanisms. Division by zero errors in these contexts typically arise from:

1. **Token Balance Calculations**: Miscalculating balances or supplies leading to division by zero during rewards distribution.
2. **Cross-Chain Rewards Distribution**: Incorrect handling of derived supply values that affect reward calculations post-transaction.
3. **Eligibility Criteria Checks**: Inaccurate checks for user eligibility, which can result in zero denominators being used in computations.

## Critical Vulnerability Patterns
1. **Improper Check on Zero Supply**:
    ```solidity
    function rewardPerToken(IERC20 token) public view override returns (uint256) {
        if (totalSupply == 0) return rewardPerTokenStored[token];
        // derivedSupply is used instead of totalSupply to modify for ve-BOOST
        return (
            rewardPerTokenStored[token] +
            (((lastTimeRewardApplicable(token) - lastUpdateTime[token]) *
                rewardRate[token] * PRECISION) / derivedSupply)
        );
    }
    ```

2. **Zero Derived Supply**:
    ```solidity
    function derivedBalance(address account) public view returns (uint256) {
        uint256 _balance = (balanceOf[account] *
            oracle.getAssetPrice(oracleAsset)) / 1e8;
        if (_balance == 0) return 0;
        uint256 multiplierE18 = eligibility.checkEligibility(account, _balance);
        return (_balance * multiplierE18) / 1e18; // Potential division by zero
    }
    ```

3. **Callback Functions Without Zero Checks**:
    ```solidity
    function handleAction() public {
        _updateReward();
        rewardPerToken(); // Calls a function that could lead to division by zero
    }
    ```

## Common Attack Vectors
1. **Manipulating Token Supply**: An attacker can perform actions (e.g., minting/burning tokens) in such a way that temporarily sets `derivedSupply` or other critical values to zero, causing subsequent transactions to revert.
2. **Exploiting Eligibility Criteria**: By manipulating token staking and eligibility checks, an attacker could make sure only they are eligible for rewards, causing others to face division by zero errors when trying to participate.

## Detection Heuristics
1. **Check All Division Operations**:
    - Ensure every division operation is preceded by a check that the denominator is not zero.
2. **Audit Derived Values**:
    - Look for derived or intermediate values used in calculations, and ensure they are properly validated.
3. **Static Analysis Tools**:
    - Use static analysis tools to identify potential division by zero points.

## Code Examples
### Vulnerable Code Example:
```solidity
function rewardPerToken(IERC20 token) public view override returns (uint256) {
    if (totalSupply == 0) return rewardPerTokenStored[token];

    // derivedSupply is used instead of totalSupply to modify for ve-BOOST
    return (
        rewardPerTokenStored[token] +
        (((lastTimeRewardApplicable(token) - lastUpdateTime[token]) *
            rewardRate[token] * PRECISION) / derivedSupply)
    );
}
```

### Mitigated Code Example:
```solidity
function rewardPerToken(IERC20 token) public view override returns (uint256) {
    if (totalSupply == 0 || derivedSupply == 0) return rewardPerTokenStored[token];

    // derivedSupply is used instead of totalSupply to modify for ve-BOOST
    return (
        rewardPerTokenStored[token] +
        (((lastTimeRewardApplicable(token) - lastUpdateTime[token]) *
            rewardRate[token] * PRECISION) / derivedSupply)
    );
}
```

## Exploit Scenarios
### Scenario 1: Locking User Funds
1. **Setup**: A pool is assigned a new gauge (gauge change occurs). At that point `totalSupply` and `derivedSupply` are 0.
2. **Action**: An attacker becomes the first minter after the gauge change, setting their balance.
3. **Impact**: Other users attempting to mint tokens will face division by zero errors, locking their assets in the contract.

### Scenario 2: Exclusive Rewards
1. **Setup**: The eligibility criteria are manipulated such that only an attacker's derived supply is non-zero.
2. **Action**: The attacker receives rewards while other participants cannot access or receive rewards due to division by zero errors.
3. **Impact**: Inequitable distribution of rewards and potential loss of user trust.

## Mitigation Strategies
1. **Ensure Comprehensive Checks**:
    - Implement checks for all denominators used in division operations to ensure they are not zero before performing the calculation.
2. **Fail-Safe Mechanisms**:
    - Add fail-safe mechanisms that revert transactions gracefully with clear error messages when a zero division is detected, rather than halting execution.
3. **Code Audits and Reviews**:
    - Conduct regular code audits focusing on mathematical operations and supply/balance calculations to identify potential points of failure.

## Detector Prompt
### Specialized LLM-based Vulnerability Scanner Prompt:
"Scan the provided Solidity smart contract code for instances where division by zero could occur. Specifically, look for any division operation that doesn't check if its denominator is zero before performing the calculation. Identify critical sections such as reward calculations, balance checks, and derived supply computations. Highlight vulnerable lines of code with comments explaining potential points of failure and suggest improvements to prevent division by zero errors."

By following these guidelines, developers can better identify, mitigate, and prevent Division By Zero vulnerabilities in bridge protocols, ensuring smoother operations and enhanced security for users.
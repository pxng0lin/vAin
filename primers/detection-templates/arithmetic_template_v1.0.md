# Bridge Protocols {"Primary_Type": "Arithmetic_Error", "Sub_Type": "Incorrect_Math_Operation", "Confidence": 0.95, "Description": "The Calculatebonus Function Incorrectly Divides Bonusbps By 100, Leading To An Over-Accrual Of Bonuses."} Vulnerability Arithmetic Detection Template

## Version
1.0

## Last Updated
2025-07-15

## Vulnerability Type
Arithmetic Error: Incorrect Math Operation in Bridge Protocols

## Description
This vulnerability occurs when a smart contract's arithmetic calculations are incorrectly implemented, leading to unintended behavior and potentially significant financial loss. Specifically in bridge protocols, such errors can result in over-accrual of bonuses or rewards, causing misallocation of funds and possible exploitation by malicious actors.

## Target Protocol Type
Bridge Protocols {"Primary_Type": "Arithmetic_Error", "Sub_Type": "Incorrect_Math_Operation", "Confidence": 0.95, "Description": "The Calculatebonus Function Incorrectly Divides Bonusbps By 100, Leading To An Over-Accrual Of Bonuses."} Vulnerability

## Detection Criteria
- **Incorrect Division**: Look for instances where basis points (bps) values are divided by 100.
- **Improper Bonus Calculation**: Scan for flawed arithmetic logic used in bonus calculations.

**Example Code Pattern**:
```solidity
function calculateBonus(
        uint256 amount
    ) public view override returns (uint256) {
        uint256 bonus = (amount * bonusBps) / 100; // Incorrect: should use bonusBps directly
        if (zero.balanceOf(address(this)) < bonus) return 0;
        return (amount * bonusBps) / 100;
    }
```

## False Positives
- Scenarios where division by 100 is intentional and correctly implemented according to the business logic.
- Code comments explicitly stating that the division by 100 is intended.

## Scanning Logic
1. **Static Code Analysis**:
   - Scan for instances where `bps` values are divided by 100 within bonus calculation functions.
2. **Dynamic Testing**:
   - Unit tests with boundary cases to ensure correct calculation logic.
3. **Formal Verification**:
   - Use formal verification tools to mathematically prove the correctness of arithmetic operations.

## Example Vulnerable Code
```solidity
function calculateBonus(
        uint256 amount
    ) public view override returns (uint256) {
        uint256 bonus = (amount * bonusBps) / 100; // Incorrect: should use bonusBps directly
        if (zero.balanceOf(address(this)) < bonus) return 0;
        return (amount * bonusBps) / 100;
    }
```

## Example Fixed Code
```solidity
function calculateBonus(
        uint256 amount
    ) public view override returns (uint256) {
        uint256 bonus = (amount * bonusBps) / 1_0000; // Correct usage of basis points
        if (zero.balanceOf(address(this)) < bonus) return 0;
        return bonus;
    }
```

## References
- [Smart Contract Security Best Practices](https://consensys.github.io/smart-contract-best-practices/)
- [Formal Verification Tools for Solidity](https://docs.soliditylang.org/en/latest/formal-verification.html)

This detection template is designed to identify and mitigate arithmetic errors in bridge protocols, with a specific focus on incorrect math operations that can lead to over-accrual of bonuses. It is optimized for LLM-based vulnerability scanning, ensuring thorough analysis and accurate detection.
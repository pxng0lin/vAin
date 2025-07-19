# Bridge Protocols {"Primary_Type": "Access-Control", "Sub_Type": Null, "Confidence": 0.95, "Description": "The `Createcollateraltoken` Function In The `Playcollateraltokenfactory` Contract Allows Any Address To Create New Tokens With Arbitrary Supply And Become The Owner, Bypassing Intended Access Controls."} Vulnerability Access Detection Template

## Version
1.0

## Last Updated
2025-07-15

## Vulnerability Type
Unauthorized Token Creation and Minting Vulnerability in Bridge Protocols

## Description
The `createCollateralToken` function in the `PlayCollateralTokenFactory` contract allows any address to create new tokens with arbitrary supply and become the owner, thereby bypassing intended access controls. This vulnerability fundamentally undermines the security model of bridge protocols by enabling unrestricted token creation and ownership, which can lead to unauthorized minting of tokens and potential exploitation.

## Target Protocol Type
Bridge Protocols {"Primary_Type": "Access-Control", "Sub_Type": Null, "Confidence": 0.95, "Description": "The `Createcollateraltoken` Function In The `Playcollateraltokenfactory` Contract Allows Any Address To Create New Tokens With Arbitrary Supply And Become The Owner, Bypassing Intended Access Controls."} Vulnerability

## Detection Criteria
1. **Public Accessibility**: Identify functions in bridge protocol contracts that are publicly accessible and allow token creation/minting without access control checks.
2. **Arbitrary Supply Specification**: Look for parameters related to `initialSupply` or similar variables, checking if they can be set arbitrarily by the function callers.
3. **Owner Assignment**: Ensure that the caller specifies the owner address, enabling them to gain full control over newly created tokens.

## False Positives
Patterns that might look like this vulnerability but are actually safe:
1. Token creation functions with proper access controls (e.g., using `onlyOwner` or similar modifiers).
2. Functions where `initialSupply` is restricted to a predefined range or requires administrative approval.
3. Mechanisms ensuring only authorized entities can assign token ownership.

## Scanning Logic
1. **Public Function Check**:
   - Identify functions in bridge protocol contracts that are publicly accessible and allow token creation/minting without access control checks.
2. **Owner Assignment Verification**:
   - Look for functions where caller-specified owner addresses gain full control over newly created tokens.
3. **Supply Parameters Analysis**:
   - Examine parameters related to `initialSupply` or similar variables, checking if they can be set arbitrarily by the function callers.

## Example Vulnerable Code
```solidity
function createCollateralToken(
    string memory name,
    string memory symbol,
    uint256 initialSupply,
    address owner
) external returns (address) {
    PlayCollateralToken newToken = new PlayCollateralToken(
        name,
        symbol,
        initialSupply,
        CONDITIONAL_TOKENS,
        owner
    );

    emit PlayCollateralTokenCreated(address(newToken));
    return address(newToken);
}
```

## Example Fixed Code
```solidity
function createCollateralToken(
    string memory name,
    string memory symbol,
    uint256 initialSupply,
    address owner
) external onlyOwner returns (address) {
    require(initialSupply <= MAX_SUPPLY, "Initial supply exceeds limit");
    PlayCollateralToken newToken = new PlayCollateralToken(
        name,
        symbol,
        initialSupply,
        CONDITIONAL_TOKENS,
        owner
    );

    emit PlayCollateralTokenCreated(address(newToken));
    return address(newToken);
}
```

## References
1. "Access-Control Vulnerabilities in Bridge Protocols"
2. "Smart Contract Security Best Practices for Token Creation Functions"

By following this detection template, developers and auditors can effectively identify and mitigate unauthorized token creation vulnerabilities in bridge protocol smart contracts.
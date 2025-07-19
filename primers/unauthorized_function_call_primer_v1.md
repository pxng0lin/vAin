# Bridge Protocols Unauthorized_Function_Call Vulnerability Primer

## Version
1.0

## Last Updated
2025-07-13

## Vulnerability Type
Unauthorized Function Call in Bridge Protocols

## Description
Unauthorized function call vulnerabilities in bridge protocols occur when critical functions intended to be executed only by authorized contracts or users can be accessed and invoked by unauthorized entities. This often results from insufficient access control checks, leading to potential exploitation where unauthorized actors perform actions that they shouldn't, such as deploying tokens without proper authorization.

## Protocol Context
In the context of bridge protocols, which facilitate the transfer of assets across different blockchain networks, functions related to asset deployment, token creation, and cross-chain transactions are particularly sensitive. These operations must be restricted to authorized entities to prevent misuse and ensure security. Failure in access control can lead to unauthorized deployments or transfers that compromise the integrity and financial security of the bridge.

## Critical Vulnerability Patterns
- Functions that lack proper access control checks.
- Use of `msg.sender` for authorization without verifying if it is an authorized entity.
- Functions that are supposed to be called only by specific contracts but have no restrictions enforcing this rule.
- Inconsistent use of access control mechanisms across similar functions.

## Common Attack Vectors
1. **Cross-Chain Arbitrage**: Exploiting the vulnerability to deploy tokens without paying for gas in the destination chain.
2. **Token Spoofing**: Deploying a token with the same identifier as an authorized token, causing confusion and potential loss of assets.
3. **Unauthorized Access**: Bypassing access control checks to call restricted functions.

## Detection Heuristics
- Check if there are any functions that do not have access control mechanisms in place.
- Look for functions where `msg.sender` is used but not verified against a list or map of authorized addresses/contracts.
- Review the presence and consistency of access control checks across similar function definitions.
- Use static analysis tools to identify functions with missing or inconsistent access controls.

## Code Examples
```solidity
// Example of an unauthorized call to deployInterchainToken
function deployInterchainToken(address tokenAddress, bytes32 salt) public {
    // Missing check for authorized caller
    address tokenId = keccak256(abi.encode(tokenAddress, salt));
    _deployToken(tokenId);
}
```

## Exploit Scenarios
1. **Cross-Chain Token Deployment Without Gas Payment**:
   - Alice deploys an interchain token in Chain A.
   - She uses the same `salt` value to deploy the same token in Chain B without paying gas fees for remote deployment by exploiting the unauthorized function call.

2. **Token Spoofing Across Chains**:
   - An attacker deploys a malicious token with the same identifier as an authorized interchain token, causing users to mistakenly interact with the malicious token.
   - The attacker exploits this confusion to drain assets or manipulate token supply.

## Mitigation Strategies
1. **Implement Access Control Mechanisms**: Ensure that only authorized contracts or addresses can call sensitive functions.
2. **Use Modifiers for Function Access Control**:
    ```solidity
    modifier onlyInterchainTokenFactory() {
        require(msg.sender == interchainTokenFactoryAddress, "Unauthorized caller");
        _;
    }

    function deployInterchainToken(address tokenAddress, bytes32 salt) public onlyInterchainTokenFactory {
        address tokenId = keccak256(abi.encode(tokenAddress, salt));
        _deployToken(tokenId);
    }
    ```
3. **Consistent Access Control Checks**: Apply access control checks consistently across all functions with similar sensitivity levels.
4. **Regular Audits and Code Reviews**: Conduct regular security audits focusing on access controls for bridge protocols.

## Detector Prompt
"Identify functions in a bridge protocol smart contract that do not have proper access control mechanisms. Specifically, look for public or external functions related to token deployment, interchain transactions, or sensitive operations where `msg.sender` is used without verification against an authorized list or map."

By following this primer and incorporating the suggested strategies, you can enhance the security of bridge protocols by mitigating unauthorized function call vulnerabilities.

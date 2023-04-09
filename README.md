# Weset-Audit-Report
# Missing of NatPec for functions


# [H-02] Centralization Risk: Admin Can Block Users from Claiming Tokens

## Impact
The `claim()` function in WesetDrop.sol allows users who are on the AllowlistProof to claim tokens under certain conditions set by the protocol admin using the `setClaimConditions()` function within the same contract, at any time. However, a compromised admin could launch a Denial of Service (DoS) attack on users by changing the claim conditions to any values before all users on the AllowlistProof are able to claim their tokens. This would cause affected users to lose their claim tokens until the next claim phase, which could potentially last for an unlimited period of time(currentClaimPhase.startTimestamp)

## Proof of Concept
```solidity

function setClaimConditions(
        uint256 _tokenId,
        ClaimCondition calldata _condition,
        bool _resetClaimEligibility,
        uint256[] calldata _prices
    ) external override {
        if (!_canSetClaimConditions()) {
            revert("Not authorized");
        }
        //..........

```
https://github.com/wesetio/weset-contracts/blob/main/contracts/WesetDrop.sol#L37
https://github.com/wesetio/weset-contracts/blob/main/contracts/WesetDrop.sol#L101

## Recommended Mitigation Steps
Consider adding time-lock functionalities to the setClaimConditions() function and communicate the change period to Weset users to increase transparency and security. This will ensure that all users have claimed their tokens under certain conditions before any changes are made by the Admin.




# [M-01] WesetProtocol.sol: Sanity check bypass in burn() & burnBatch() functions

## Impact
The burn() and burnBatch() functions in WesetProtocol.sol do not explicitly check the balance of the caller when caller == _owner. This means that anyone can call the functions, even if they don't own any NFTs, by setting the amount parameter to 0. This allows an attacker to bypass both checks. Although this vulnerability currently does not affect the protocol, it is unsafe to leave it open, as a malicious attacker could potentially exploit it to cause unexpected behavior in the future.

## Proof of Concept
```solidity

function burn(

        address _owner,

        uint256 _tokenId,

        uint256 _amount

    ) external virtual {

        address caller = msg.sender;
​
        require(caller == _owner || isApprovedForAll[_owner][caller], "Unapproved caller");

        require(balanceOf[_owner][_tokenId] >= _amount, "Not enough tokens owned");
​

        _burn(_owner, _tokenId, _amount);

    }

```
## Recommended Mitigation Steps
Consider adding an explicit check to ensure that `_amount` is greater than 0, so that the function only allows burning of NFTs by callers who actually own them. This can be done by adding a simple `require` statement at the beginning of the functions

Make the following changes:
```
// burn function
require(_amount > 0, "_amount cannot be zero");
// burnBatch function
for (uint256 i = 0; i < _tokenIds.length; i += 1) {
    // To add
    uint256 amount  = _amounts[i]
     require(amount > 0, "amount cannot be zero");
 require(balanceOf[_owner][_tokenIds[i]] >= amount, "Not enough tokens owned");
}

```



## /******** LOW  *********/

# [L-01] Undeclared events emitted
There are 3 instances of this issue:
https://github.com/wesetio/weset-contracts/blob/main/contracts/WesetDrop.sol#L144
https://github.com/wesetio/weset-contracts/blob/main/contracts/WesetDrop.sol#L87
https://github.com/wesetio/weset-contracts/blob/main/contracts/WesetDrop.sol#L144

## Impact
This can lead to confusion for external parties trying to interact with the contract.

## Recommended Mitigation Steps
Make sure to declare all events in the contract, including their parameter types, before emitting them.

----------------------------------------------------------------------------------------------------------------
# [L-02] Pragma Version
In the contracts, floating pragmas should not be used. Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

## Proof of Concept
https://swcregistry.io/docs/SWC-103

All Contracts

## Recommended Mitigation Steps
It is recommended to lock the pragma version in the contract from `^0.8.0` to a specific version, such as `0.8.0` Additionally, it is also recommended to use the latest version of the Solidity compiler `0.8.19` to benefit from the latest security features and bug fixes.

----------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------
# [L-02] Missing zero-address check
Missing checks for zero-addresses may lead to unxpected bahaviors, if the variable addresses are updated incorrectly.

## Proof of Concept
There are 2 instances of this issue:
https://github.com/wesetio/weset-contracts/blob/main/contracts/WesetDrop.sol#L38


## Recommended Mitigation Steps
It is recommended to lock the pragma version in the contract from `^0.8.0` to a specific version, such as `0.8.0` Additionally, it is also recommended to use the latest version of the Solidity compiler `0.8.19` to benefit from the latest security features and bug fixes.

----------------------------------------------------------------------------------------------------------------

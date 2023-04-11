# Weset-Audit-Report


# [H-01] ClaimCondition Struct data only updated in Memory, not in Storage
## Impact
The setClaimConditions() and claim() functions in WesetDrop.sol update the ClaimCondition struct data under certain conditions but only on the memory level and not on the storage level. Since ClaimCondition contains sensitive information, this can lead to data loss, inconsistency and a potential centralization risk.

When calling setClaimConditions() to update new conditions for minting, it allows users to not mint under the new conditions set by the admin, as the values aren't updated on the storage level. This could lead to a centralization risk, as the admin could inadvertently DOS himself by not being able to update any data for the new minters.

## Proof of Concept
```solidity
// from claim function
ClaimCondition memory condition = claimCondition[_tokenId];
        bytes32 activeConditionId = conditionId[_tokenId];
        verifyClaim(
            _tokenId,
            _dropMsgSender(),
            _quantity,
            _currency,
            _allowlistProof
        );

        // Update contract state.
        condition.supplyClaimed += _quantity;
        supplyClaimedByWallet[activeConditionId][_dropMsgSender()] += _quantity;
        claimCondition[_tokenId] = condition;

```

```solidity
// from setClaimConditions() function
ClaimCondition memory condition = claimCondition[_tokenId];
        bytes32 targetConditionId = conditionId[_tokenId];

        uint256 supplyClaimedAlready = condition.supplyClaimed;

        // Set price array for the token
        setPrice(_tokenId, _prices);

        if (_resetClaimEligibility) {
            supplyClaimedAlready = 0;
            targetConditionId = keccak256(
                abi.encodePacked(_dropMsgSender(), block.number)
            );
        }

        if (supplyClaimedAlready > _condition.maxClaimableSupply) {
            revert("max supply claimed");
        }

        ClaimCondition memory updatedCondition = ClaimCondition({
            startTimestamp: _condition.startTimestamp,
            maxClaimableSupply: _condition.maxClaimableSupply,
            supplyClaimed: supplyClaimedAlready,
            quantityLimitPerWallet: _condition.quantityLimitPerWallet,
            merkleRoot: _condition.merkleRoot,
            pricePerToken: _condition.pricePerToken,
            currency: _condition.currency,
            metadata: _condition.metadata
        });

        claimCondition[_tokenId] = updatedCondition;
        conditionId[_tokenId] = targetConditionId;
```
https://github.com/wesetio/weset-contracts/blob/main/contracts/WesetDrop.sol#L37  
https://github.com/wesetio/weset-contracts/blob/main/contracts/WesetDrop.sol#L57-L72  
https://github.com/wesetio/weset-contracts/blob/main/contracts/WesetDrop.sol#L110-L148  


## Recommended Mitigation Steps
Make sure to use the `storage` instead of `memory` when updating values on the storage level/blockchain


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
        require(caller == _owner || isApprovedForAll[_owner][caller], "Unapproved caller");
        require(balanceOf[_owner][_tokenId] >= _amount, "Not enough tokens owned");
        _burn(_owner, _tokenId, _amount);
    }

```
## Recommended Mitigation Steps
Consider adding an explicit check to ensure that `_amount` is greater than 0, so that the function only allows burning of NFTs by callers who actually own them. This can be done by adding a simple `require` statement at the beginning of the functions

Make the following changes:
```solidity
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

# [L-01] Undeclared events emitted
There are 3 instances of this issue:  
https://github.com/wesetio/weset-contracts/blob/main/contracts/WesetDrop.sol#L144  
https://github.com/wesetio/weset-contracts/blob/main/contracts/WesetDrop.sol#L87  
https://github.com/wesetio/weset-contracts/blob/main/contracts/WesetDrop.sol#L144  

## Impact
This can lead to confusion when interacting with the contract through the user interface (UI) as it does not emit any events to provide or signal changes in contract state.

## Recommended Mitigation Steps
Make sure to declare all events in the contract, including their parameter types, before emitting them.

----------------------------------------------------------------------------------------------------------------
# [L-02] Pragma Version
In the contracts, floating pragmas should not be used. Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

## Proof of Concept
https://swcregistry.io/docs/SWC-103

All Contracts

## Recommended Mitigation Steps
It is recommended to lock the pragma version in the contract from `^0.8.0` to a specific version, such as `0.8.0`.

----------------------------------------------------------------------------------------------------------------
# [L-02] Missing zero-address check
Missing checks for zero-address.

## Proof of Concept
There are 1 instance of this issue:
https://github.com/wesetio/weset-contracts/blob/main/contracts/WesetDrop.sol#L38


## Recommended Mitigation Steps
Make sure to add a `require` statement to check if the _receiver parameter address is not equal to address(0);

```solidity
require(_receiver != address(0));

```

----------------------------------------------------------------------------------------------------------------

# [QA-01] use the latest version of the Solidity compiler
It's recommended to use the latest version of the Solidity compiler `0.8.19` to benefit from the latest security features and bug fixes.

# [QA-02] unused of NatSpec
It is best practice to use descriptive comments that comply with NatSpec to provide clear and comprehensive documentation for contracts, functions, and return variables.

----------------------------------------------------------------------------------------------------------------

# [QA-03] TWString is used but uncomment which will make the TWString not benefiting from the library  
https://github.com/wesetio/weset-contracts/blob/main/contracts/WesetProtocol.sol#L19  
https://github.com/wesetio/weset-contracts/blob/main/contracts/WesetProtocol.sol#L39  

----------------------------------------------------------------------------------------------------------------

# [G-03] The msg.sender == owner() check in functions can be consolidated into a single modifier to improve code readability and  gas efficiency.
https://github.com/wesetio/weset-contracts/blob/main/contracts/WesetProtocol.sol#L351-L417  

## Recommended Mitigation Steps
```solidity
modifier onlyOwner() {
    require(msg.sender == owner(), "Not authorized");
}
```
----------------------------------------------------------------------------------------------------------------


# Weset-Audit-Report

# [M-01] At WesetProtocol.sol sanity check in at burn() & burnBatch() can be bypass

## Impact
The function burn() and burnBatch don't check explicitly in case the caller == _owner as Anyone cann call this token 

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

## Recommendation

Make the following change:

```diff
- user.amount = user.amount - _shares;
+ user.amount = user.amount - r;
```

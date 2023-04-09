# Weset-Audit-Report

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
Consider adding an explicit check to ensure that `_amount` is greater than 0, so that the function only allows burning of NFTs by callers who actually own them. This can be done by adding a simple `require` statement at the beginning of the function burn and in the inside for loop of burnBatch function

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

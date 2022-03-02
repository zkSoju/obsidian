## Pull Request Breakdowns

[ERC721A issue](https://github.com/chiru-labs/ERC721A/issues/92) solved in this [PR](https://github.com/chiru-labs/ERC721A/pull/131)
 
There is an existence of a reentrancy attack in the follow code in the `_mint()` function:

```solidity
unchecked {
    _addressData[to].balance += uint128(quantity);
    _addressData[to].numberMinted += uint128(quantity);

    _ownerships[startTokenId].addr = to;
    _ownerships[startTokenId].startTimestamp = uint64(block.timestamp);

    uint256 updatedIndex = startTokenId;

    for (uint256 i; i < quantity; i++) {
        emit Transfer(address(0), to, updatedIndex);
        if (safe && !_checkOnERC721Received(address(0), to, updatedIndex, _data)) {
            revert TransferToNonERC721ReceiverImplementer();
        }

        updatedIndex++;
    }

    _currentIndex = updatedIndex;
}
```

 `_checkOnERC721Received()` is a function that follows these rules:
 - If the target address isn't a contract (EOA), this call is not executed. 
 - However, if it is a contract then it calls the contract's `onERC721Received` 

This raises issues because an IERC721Receiver can override `onERC721Received` for malicious purposes and reenter the minting contract.

### Implemented Solution

In order to prevent misuse of the contract due to lack of understanding of `_safeMint()` not being safe, `_mint()` is modified to be reentrancy safe. The overhead cost is about 100 gas for an extra warm `SLOAD`, meaning for a ~60K gas `_safeMint()` the overhead is about 0.2% for an important safety check.

`_safeMint()` in OpenZeppelin's implementation checks that the receiving address is aware of the ERC721 protocol. However, for a majority of mints, since projects expect individual owners, a lot of issues like botting can be avoided by simply checking that the caller is an EOA through a simple `msg.sender == tx.origin`. This restricts any calls from an external contract and therefore prevents any reetrancy attacks.

### Additional Notes

-   If **YOU** are paying for the minting of tokens, use `_mint`. The `_safeMint` might cost you an arbitrary amount of money because of choices made by the recipient of the tokens. This is enough to deter you from considering it.
-   If **THEY** are paying for the minting of tokens and you expect buyers to be composing functionality with smart contracts, use `_safeMint`. There is some marginal benefit of allowing the extra features with smart contracts this allows.
-   If **THEY** are paying and you expect INDIVIDUAL PEOPLE to buy tokens, then use `_mint`. The extra features in `_safeMint` are not expected. (note: this is the case for a majority of projects)

[Reference](https://ethereum.stackexchange.com/questions/115280/mint-vs-safemint-which-is-best-for-erc721)

---

[ERC721A pull request](https://github.com/chiru-labs/ERC721A/pull/109)

Over-optimization is **bad.** 

Packing together variables will save gas in situations where the variables are commonly accessed together.

* However in this   `_currentSupply` and `_burnCounter` were given type `uint128` to allow compiler to pack together
* Both variables were more commonly accessed separately
* Only one case was this setup justifiable in `totalSupply()` where both variables were and accessed and calculation performed

---


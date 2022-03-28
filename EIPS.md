## EIP 165 (Standard Interface Detection)
https://eips.ethereum.org/EIPS/eip-165

**How can I know which interfaces a contract supports (without knowing the exact source code)?**

Creates a standard method to public and detect interfaces that smart contracts implement. 

```solidity

interface ERC165 {
    /// @notice Query if a contract implements an interface
    /// @param interfaceID The interface identifier, as specified in ERC-165
    /// @dev Interface identification is specified in ERC-165. This function
    ///  uses less than 30,000 gas.
    /// @return `true` if the contract implements `interfaceID` and
    ///  `interfaceID` is not 0xffffffff, `false` otherwise
    function supportsInterface(bytes4 interfaceID) external view returns (bool);
}

```

- ERC721 and ERC1165 compliant contracts must implement the ERC165 interface
---
## EIP 2612 (ERC20 EIP712 Permit Extension)
https://eips.ethereum.org/EIPS/eip-2612

**The ability for users to interact with Ethereum without ETH is a goal. How can we abstract the `approve` method so the function is not defined in terms of msg.sender?**

This ERC extends the ERC20 standard with a new function `permit` that allows users to modify `allowance` mapping using a signed message, instead of through `msg.sender`.

```solidity
function permit(address owner, address spender, uint value, uint deadline, uint8 v, bytes32 r, bytes32 s) external
function nonces(address owner) external view returns (uint)
function DOMAIN_SEPARATOR() external view returns (bytes32)
```
---
## EIP712 (Ethereum Typed Structured Data Hashing and Signing)
https://eips.ethereum.org/EIPS/eip-712

**Signing bytestrings are trivial. However, in the real world, we want to represent more complex messages. How do we hash this data structure without data loss and errors?** 

This EIP aims to improve usability of off-chain message signing for use on-chain, since this a gas efficient method; however, currently signing messages are displayed as a hex string with little context about the items that make up the message.

Reference [[Signatures and Hashing]] for an overview of Ethereum's signatures and hashing mechanisms and algorithms.

![[Pasted image 20220327192944.png|250]]
![[Pasted image 20220327193307.png|250]]



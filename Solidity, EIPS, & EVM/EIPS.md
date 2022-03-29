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
<img src="assets/eip712-meta-p1.png" width="250"/>

<img src="assets/eip712-meta-p2.png" width="250"/>

## Arbitrary Messages
- Just encoding structs is not enough, different dapps could use identical structs and signature for one dapp would be valid for another
	- This could be an intended behavior if the dapp too replay attacks into consideration
	- Otherwise, this could be a security flaw
- Solution to this problem is to introduce **domain separators**, which is a 256-bit number that is "mixed in" to the signature to make different domain signatures incompatible 
	- The domain separator can contain details like name of dapp, validator contract, expected dapp domain name name 
	- This information can be utilized to prevent malicious parties and phising attacks, where a malicious party could trick a user into signing a message for another dapp

### Specification
- Set of signable messages is extended for structs
-   `encode(domainSeparator : 𝔹²⁵⁶, message : 𝕊) = "\x19\x01" ‖ domainSeparator ‖ hashStruct(message)` 
	- appended version byte `0x01` causes the encoding to not collide with other encodings `\x19` and `RLP_encode(transaction)`
	- `domainSeparator` is defined above as a 32 byte variant containing details that can provide context and prevent phising
		- ```domainSeparator = hashStruct(eip712Domain)```
			- eip712Domain is a struct named `EIP721Domain` with one or more of the following fields
				- `string name`
				- `string version`
				- `uint256 chainId`
				- `address verifyingContract`
				- `bytes32 salt`
	- `hashStruct` is the data to sign
		-  `hashStruct(s : 𝕊) = keccak256(typeHash ‖ encodeData(s))` where `typeHash = keccak256(encodeType(typeOf(s)))`
			- `typehash` is a constant that represents given struct type
			- `encodeType`
				- Singular Structs
					- `Mail(address from,address to,string contents)`
				- Nested Structs
					- `Transaction(Person from,Person to,Asset tx)Asset(address token,uint256 amount)Person(address wallet,string name)
			- `encodeData` is the concatenation of encoded member values in the order they appear in type
				- `boolean` are encoded in `uint256`
				- `address`  are encoded in `uint160`
				- `bytes` and `string` are encoded as keccak256 hash of contents
				- `array` are encoded as if they were structs (ex. `SomeType[5]` is equivalent to a struct containing 5 of SomeType)
#### Specification of `eth_signTypedData` JSON RPC

`eth_signTypedData` is added to JSON RPC which parallels the `eth_sign` method

Parameters:
- `Address` - address of account signing message
- `TypedData` - structured data being signed, below is json-schema definition
```JavaScript
{
  type: 'object',
  properties: {
    types: {
      type: 'object',
      properties: {
        EIP712Domain: {type: 'array'},
      },
      additionalProperties: {
        type: 'array',
        items: {
          type: 'object',
          properties: {
            name: {type: 'string'},
            type: {type: 'string'}
          },
          required: ['name', 'type']
        }
      },
      required: ['EIP712Domain']
    },
    primaryType: {type: 'string'},
    domain: {type: 'object'},
    message: {type: 'object'}
  },
  required: ['types', 'primaryType', 'domain', 'message']
}
```

Returns:
- `Data` - similar to `eth_sign` returns hex encoded 129 byte array starting with 0x encoding r, s, v

### Key Takeaways
- `encode` function extended with a new case for new types
- first byte of encoding distinguishes between cases
- starting with a typehash or domain separator is not safe because while unlikely it is possible to construct a typehash that is also the prefix of a RLP encoded transaction
- domain separator prevents collision because it is possible for separate dapps to have the same struct

### Ecosystem Adoption
- Led to ERC20 transction-less token approvals in EIP-2612
- Metamask utilizes this EIP-712 to provide users with a more readible output for signing
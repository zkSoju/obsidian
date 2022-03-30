## EIP 165 (Standard Interface Detection)
https://eips.ethereum.org/EIPS/eip-165

**How can I know which interfaces a contract supports (without knowing the exact source code)?**

Creates a standard method to public and detect interfaces that smart contracts implement. Interface IDs are defined by the XOR of all function selectors defined in the ABI.

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

- Implementing contracts of the ERC165 interface will have the above function that returns the required specifications 

How to check if a contract implements ERC165:
1.  The source contract makes a `STATICCALL` to the destination address with input data: `0x01ffc9a701ffc9a700000000000000000000000000000000000000000000000000000000` and gas 30,000. This corresponds to `contract.supportsInterface(0x01ffc9a7)`.
2.  If the call fails or return false, the destination contract does not implement ERC-165.
3.  If the call returns true, a second call is made with input data `0x01ffc9a7ffffffff00000000000000000000000000000000000000000000000000000000`.
4.  If the second call fails or returns true, the destination contract does not implement ERC-165.
5.  Otherwise it implements ERC-165.

How to check if a contract implements a specific interface:
1. Use the `supportsInterface()` method 
2. If it does not support ERC165, then you will have to do it the old fashion way

**Note:** With three or more supported interfaces (including ERC165 itself as a required supported interface), the mapping approach (in every case) costs less gas than the pure approach (at worst case).

UPDATE: `type(ITest).interfaceId` = xor of all selectors and params, return type does not matter

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

--- 
## EIP 2981 (NFT Royalties)
https://eips.ethereum.org/EIPS/eip-2981

**How can we provide a standardize way for all marketplace to interface with and manage royalty payouts that are baked into the contract?**

```solidity
   function royaltyInfo(
        uint256 _tokenId,
        uint256 _salePrice
    ) external view returns (
        address receiver,
        uint256 royaltyAmount
    );
```

This EIP allows contracts to signal royalty payout to NFT creator every time the item is sold or re-sold. The intended use case is for marketplaces that intend to support ongoing funding of creators and artists. This royalty payout is not baked into the transfer function because transfer does not signal a sale. Details like the payout mechanism and notifying recipient are not described in this EIP to keep things minimal and gas efficient (intended to be expanded upon in other EIPs).

Incompatibility is an issue that plagued ERC20 royalty implementations and early NFT contracts making it difficult to implement across several marketplaces. A universal royalty standard is needed to provide ongoing funding for creators to promote adoption and innovation. This EIP only describes the royalty percentage and recipient, payouts are managed by the marketplace or platform utilizing this interface.

Royalty payouts are always a percentage of sale price and it is considered an voluntary standard. If a contract implements ERC2981 but the marketplace decides not to support it, there will be zero royalty payouts. Buyers will assess royalty payouts as a factor of NFT purchasing decisions.

### Specification

- Implementers must payout `royaltyAmount` in the same currency as `_salePrice`
- `royaltyInfo()` is unaware of the unit of exchange; therefore, the `royaltyAmount` or calculation of `royaltyAmount` should not be constant because that makes assumptions of the unit of exchange
- For the reason above, the `royaltyAmount` **must** be a percantage of `_salePrice`
	- Percentage may be reliant on other variables that don't make assumptions on unit of exchange
		- Should not be reliant unpredictable variables like `block.timestamp`
		- Could use the number of NFT transfers or `_tokenId` as a way to calculate `royaltyAmount`
- Implementing marketplaces that respect the EIP **must** pay royalties no matter where sale occurred (OTC, on-chain, off-chain) and regardless of currency

### Key Takeaways
- Optional royalty payments - impossible to know which transfers are sales, so it is the responsibility of marketplace to voluntarily implement
- Simple payments to single address -  it is up to the payment receiver to split fees, handle multiple recipients, taxes, etc because adding it into this EIP would make it gas-inefficient and difficult to cover all use cases
- Royalty payment percentage calculation - `royaltyAmount` is calculated as a percentage of `_salePrice` and rather than returning a percentage and allowing marketplace to manage exact payout, a specific amount is returned to avoid marketplace disputes
- Unit-less across marketplaces on-chain + off-chain
- Universal payments
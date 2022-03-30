## Repository Breakdowns

### Zora V3

ZoraModuleManager.sol
https://github.com/ourzora/v3/blob/main/contracts/ZoraModuleManager.sol

```solidity
/// @notice The EIP-712 type for a signed approval

/// @dev keccak256("SignedApproval(address module,address user,bool approved,uint256 deadline,uint256 nonce)")

bytes32 private constant SIGNED_APPROVAL_TYPEHASH = 0x8413132cc7aa5bd2ce1a1b142a3f09e2baeda86addf4f9a5dacd4679f56e7cec;

/// @notice The EIP-712 domain separator

bytes32 private immutable EIP_712_DOMAIN_SEPARATOR = keccak256(abi.encode(keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"), keccak256(bytes("ZORA")), keccak256(bytes("3")), _chainID(), address(this)));

mapping(address => uint256) public sigNonces;

...

/// @notice Sets approval for a module given an EIP-712 signature

/// @param _module The module to approve

/// @param _user The user to approve the module for

/// @param _approved A boolean, whether or not to approve a module

/// @param _deadline The deadline at which point the given signature expires

/// @param _v The 129th byte and chain ID of the signature

/// @param _r The first 64 bytes of the signature

/// @param _s Bytes 64-128 of the signature

function setApprovalForModuleBySig(

address _module,

address _user,

bool _approved,

uint256 _deadline,

uint8 _v,

bytes32 _r,

bytes32 _s

) public {

require(_deadline == 0 || _deadline >= block.timestamp, "ZMM::setApprovalForModuleBySig deadline expired");

bytes32 digest = keccak256(abi.encodePacked("\x19\x01", EIP_712_DOMAIN_SEPARATOR, keccak256(abi.encode(SIGNED_APPROVAL_TYPEHASH, _module, _user, _approved, _deadline, sigNonces[_user]++))));

address recoveredAddress = ecrecover(digest, _v, _r, _s);

require(recoveredAddress != address(0) && recoveredAddress == _user, "ZMM::setApprovalForModuleBySig invalid signature");

_setApprovalForModule(_module, _user, _approved);

}
```

1. Utilizes EIP-712 referenced in [EIPS](Solidity,%20EIPS,%20&%20EVM/EIPS.md) to sign a typed structured data constructed of a `domainSeparator` and `typeHash` and `encodedData`.
2. Breaks up the signature into `r`, `s`, `v`
3. Stores the signature nonces for approvals in a mapping
4. Requires recovered address to be the address of the user the module is being approved for
5. Approves individual modules using signature

AsksV1_1.sol (Module contract)
https://github.com/ourzora/v3/blob/main/contracts/modules/Asks/V1.1/AsksV1_1.sol
- Composed of all of the below utility contracts
- Each tokenID per token address contains a singular ask

OffersV1_1.sol (Module contract)
https://github.com/ourzora/v3/blob/main/contracts/modules/Offers/V1/OffersV1.sol

```solidity
...

/// @notice The metadata of an offer
/// @param maker The address of the user who made the offer
/// @param currency The address of the ERC-20 offered, or address(0) for ETH
/// @param findersFeeBps The fee to the referrer of the offer
/// @param amount The amount of ETH/ERC-20 tokens offered

struct Offer {
	address maker;
	address currency;
	uint16 findersFeeBps;
	uint256 amount;
}

/// ------------ STORAGE ------------

/// @notice The metadata for a given offer
/// @dev ERC-721 token address => ERC-721 token ID => Offer ID => Offer
mapping(address => mapping(uint256 => mapping(uint256 => Offer))) public offers;

/// @notice The offers for a given NFT
/// @dev ERC-721 token address => ERC-721 token ID => Offer IDs
mapping(address => mapping(uint256 => uint256[])) public offersForNFT;
```

- Finders fee

IncomingTransferSupportV1.sol (Utility contract)
https://github.com/ourzora/v3/blob/main/contracts/common/IncomingTransferSupport/V1/IncomingTransferSupportV1.sol

```solidity
...

/// @notice Handle an incoming funds transfer, ensuring the sent amount is valid and the sender is solvent
/// @param _amount The amount to be received
/// @param _currency The currency to receive funds in, or address(0) for ETH

function _handleIncomingTransfer(uint256 _amount, address _currency) internal {
	if (_currency == address(0)) {
		require(msg.value >= _amount, "_handleIncomingTransfer msg value less than expected amount");
	} else {
		// We must check the balance that was actually transferred to this contract,
		// as some tokens impose a transfer fee and would not actually transfer the
		// full amount to the market, resulting in potentally locked funds
		IERC20 token = IERC20(_currency);
		uint256 beforeBalance = token.balanceOf(address(this));
		erc20TransferHelper.safeTransferFrom(_currency, msg.sender, address(this), _amount);
		
		uint256 afterBalance = token.balanceOf(address(this));
		
		require(beforeBalance + _amount == afterBalance, "_handleIncomingTransfer token transfer call did not transfer expected amount");
}

}
```

- `beforeBalance` and `afterBalance` check used to ensure standard ERC20 implementation since there are tokens that enforce a tax on transfer like SafeMoon

ModuleNamingSupport
https://github.com/ourzora/v3/blob/main/contracts/common/ModuleNamingSupport/ModuleNamingSupportV1.sol

- Contains a simple string for module name

UniversalExchangeEventV1 (Utility contract)
https://github.com/ourzora/v3/blob/main/contracts/common/UniversalExchangeEvent/V1/UniversalExchangeEventV1.sol

```solidity
/// @notice The metadata of a token exchange
/// @param tokenContract The address of the token contract
/// @param tokenId The id of the token
/// @param amount The number of tokens sent

struct ExchangeDetails {
	address tokenContract;
	uint256 tokenId;
	uint256 amount;
}

/// @notice Emitted when a token exchange is executed
/// @param userA The address of user A
/// @param userB The address of a user B
/// @param a The metadata of user A's exchange
/// @param b The metadata of user B's exchange
event ExchangeExecuted(address indexed userA, address indexed userB, ExchangeDetails a, ExchangeDetails b);
```

- Includes simple struct definition containing one component of an exchange and an event for indexing exchanges

FeePayoutSupportV1 (Utility contract)
https://github.com/ourzora/v3/blob/main/contracts/common/FeePayoutSupport/FeePayoutSupportV1.sol

```solidity
/// @notice Update the address of the Royalty Engine, in case of unexpected update on Manifold's Proxy
/// @dev emergency use only – requires a frozen RoyaltyEngineV1 at commit 4ae77a73a8a73a79d628352d206fadae7f8e0f74
/// to be deployed elsewhere, or a contract matching that ABI
/// @param _royaltyEngine The address for the new royalty engine

function setRoyaltyEngineAddress(address _royaltyEngine) public {
	require(msg.sender == registrar, "setRoyaltyEngineAddress only registrar");
	require(ERC165Checker.supportsInterface(_royaltyEngine, type(IRoyaltyEngineV1).interfaceId), "setRoyaltyEngineAddress must match IRoyaltyEngineV1 interface");
	
	royaltyEngine = IRoyaltyEngineV1(_royaltyEngine);

}
```

- Rarible allows payouts on token sale to multiple royalty recipients in an array; however, the gas costs are high
- In order to support all of these royalty mechanisms like Foundation, Rarible, ZORA, etc., Manifold's `IRoyaltyEngineV1` incurs a high gas fee
- IERC2981 (referenced in [EIPS](../Solidity,%20EIPS,%20&%20EVM/EIPS.md)) standardizes payouts to a singular address and if you want to handle split payouts then specify the singular address as a payment splitter
- OZ's `ERC165Checker` checks for invalid cases, ERC165 support, and specific interface ID support

```solidity
/// @notice Pays out the protocol fee to its fee recipient
/// @param _amount The sale amount
/// @param _payoutCurrency The currency to pay the fee
/// @return The remaining funds after paying the protocol fee
function _handleProtocolFeePayout(uint256 _amount, address _payoutCurrency) internal returns (uint256) {

	// Get fee for this module
	uint256 protocolFee = protocolFeeSettings.getFeeAmount(address(this), _amount);
	
	// If no fee, return initial amount
	if (protocolFee == 0) return _amount;
	
	// Get fee recipient
	(, address feeRecipient) = protocolFeeSettings.moduleFeeSetting(address(this));
	
	// Payout protocol fee
	_handleOutgoingTransfer(feeRecipient, protocolFee, _payoutCurrency, 50000);
	
	// Return remaining amount
	return _amount - protocolFee;

}
```

- `protocolFeeSettings` references the NFT representation of the fee module which is used to fetch `protocolFee` and `feeRecipient`
- this internal function is called first to extract fees for protocol then returns remaining funds from sale after paying fees

```solidity
/// @notice Pays out royalties for given NFTs
/// @param _tokenContract The NFT contract address to get royalty information from
/// @param _tokenId, The Token ID to get royalty information from
/// @param _amount The total sale amount
/// @param _payoutCurrency The ERC-20 token address to payout royalties in, or address(0) for ETH
/// @param _gasLimit The gas limit to use when attempting to payout royalties. Uses gasleft() if not provided.
/// @return The remaining funds after paying out royalties
function _handleRoyaltyPayout(address _tokenContract, uint256 _tokenId, uint256 _amount, address _payoutCurrency, uint256 _gasLimit) internal returns (uint256, bool) {
	// If no gas limit was provided or provided gas limit greater than gas left, just pass the remaining gas.
	uint256 gas = (_gasLimit == 0 || _gasLimit > gasleft()) ? gasleft() : _gasLimit;
	// External call ensuring contract doesn't run out of gas paying royalties
	try this._handleRoyaltyEnginePayout{gas: gas}(_tokenContract, _tokenId, _amount, _payoutCurrency) returns (uint256 remainingFunds) {
		// Return remaining amount if royalties payout succeeded
		return (remainingFunds, true);
	} catch {
		// Return initial amount if royalties payout failed
		return (_amount, false);
	}
}
```

- an optional gasLimit can be passed in to give module users of the library more control over how much gas to allocate to royalty payouts
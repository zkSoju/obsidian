## Repository Breakdowns

### Zora V3

ZoraModuleManager.sol

```solidity
/// @notice The EIP-712 type for a signed approval

/// @dev keccak256("SignedApproval(address module,address user,bool approved,uint256 deadline,uint256 nonce)")

bytes32 private constant SIGNED_APPROVAL_TYPEHASH = 0x8413132cc7aa5bd2ce1a1b142a3f09e2baeda86addf4f9a5dacd4679f56e7cec;

/// @notice The EIP-712 domain separator

bytes32 private immutable EIP_712_DOMAIN_SEPARATOR =

keccak256(

abi.encode(

keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),

keccak256(bytes("ZORA")),

keccak256(bytes("3")),

_chainID(),

address(this)

)

);

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

bytes32 digest = keccak256(

abi.encodePacked(

"\x19\x01",

EIP_712_DOMAIN_SEPARATOR,

keccak256(abi.encode(SIGNED_APPROVAL_TYPEHASH, _module, _user, _approved, _deadline, sigNonces[_user]++))

)

);

address recoveredAddress = ecrecover(digest, _v, _r, _s);

require(recoveredAddress != address(0) && recoveredAddress == _user, "ZMM::setApprovalForModuleBySig invalid signature");

_setApprovalForModule(_module, _user, _approved);

}
```

- utilizes EIP-712 referenced in [EIPS](Solidity,%20EIPS,%20&%20EVM/EIPS.md) to sign a typed structured data constructed of a `domainSeparator` and `typeHash` and encodedData.
- stores the signature nonces for approvals in a mapping

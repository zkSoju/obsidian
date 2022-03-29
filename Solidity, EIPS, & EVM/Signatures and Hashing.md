Signature algorithm in Ethereum is `secp256k1` and the hashing algorithm is `keccak256`. A good hashing algorithm should satisfy security properties like:

- **Determinism** - an algorithm that will always produce the same output given a particular input
- **Pre-image resistance** - a hash function that is hard to invert (computationally infeasible)
- **Collision resistance** - difficult to find two inputs that hash to the same output

`Keccak256` contains all of these properties; however, it only applies to bytestrings. Therefore, in order to transpose these properties to different sets we need to map the set to bytestrings. `Keccak256` is deterministic in the meaning a signature hash remains the same regardless of execution time, otherwise a signature may be incorrectly rejected. `Keccak256` is injective (1-to-1 function) meaning two different inputs would not map to the same signature hash, otherwise a signature may incorrectly accepted for an unrelated message.

Ethereum has two types of messages: `bytestrings` and `transactions`.

Initially the encodings of transactions and bytestrings were defined as:
-   `encode(t : 𝕋) = RLP_encode(t)`
-   `encode(b : 𝔹⁸ⁿ) = b`

While individually each satisfy the required properties, together they do not. In order to avoid collision of `b = RLP_encode(t)` - PR 2940 introduces a new leg `encode(b : 𝔹⁸ⁿ) = "\x19Ethereum Signed Message:\n" ‖ len(b) ‖ b` to the encoding function. `\x19` represents the lenth of the prefix length 

However, in the definition above, the encoding function is non deterministic. For example, a 4 byte string with `len(b) = "4"` or `len(b)="004"` are both valid. It raises the question for `"\x19Ethereum Signed Message:\n42a…"` , does 42a represent length `4` and `2a` or `42` length and `a`. However, interestingly enough this conflict does not cause collisions because the total length of the string is enough to distinguish between cases.


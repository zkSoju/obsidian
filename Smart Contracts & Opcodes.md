The EVM executes a smart contract by reading and executing each instruction one by one. If an instruction cannot be executed due to lack of values on stack or insufficient gas remaining, the execution reverts.

In order to deploy a contract or program, an Ethereum transaction must be sent by providing no data in the `to` address field.

### Ethereum Transactions

#### Fields 
- Nonce (number of transactions sent)
- Gas price (price in wei per unit gas)
- Gas limit (maximum unit gas spent)
- To (receiver address)
- Value (amount of ether)
- Data (supplementary data in bytecode for function calls and potentially unrelated messages)
- `v,r,s`

#### Types 
- **Sending a normal transaction** - fill in `to`, `value`, and `gas limit` fields (21K base gas required to send tx). (optional) provide `data` to submit/publish to blockchain but does not affect world state
- **Deploying a contract** - leave `to` address field empty and fill `data` field with contract deployment bytecode to signal to Ethereum node to spin up EVM and create context for contract 
- **Function Call** - data field is encoded function selector and parameters
	- ex. `abi.encodeSelector(bytes4(keccak("mint()")), 4)`

Now that you know the inner workings of how to deploy a contract, let's explore what happens under the hood.

### Contract Deployment

Here we will breakdown what is consumed by the Ethereum node / supplied into the data field of an Ethereum contract deployment transaction. The data supplied is the bytecode representation of the **init code fragrament** and the **contract code**.

![[image 2.png]]

In order to better understand the following section, get familiar with [EVM opcodes](https://www.evm.codes/) One thing to note is that init code is not stored on the blockchain it is only consumed during deployment.

```solidity
PC: 0x0 opcode: PUSH1 0x80 // pushes 1 byte 0x80 on top of the stack
PC: 0x2, opcode: PUSH1 0x40 // pushes 0x40 on top of the stack
PC: 0x4, opcode: MSTORE // stores 0x80 at free memory pointer 0x40
PC: Ox5, opcode: CALLVALUE // push supplied call value in wei
PC: 0x6, opcode: DUP1 // duplicate top of stack (CALLWVALUE)
PC: 0x7, opcode: ISZERO // pops top of stack and pushes 1 if 0, 0 otherwise
PC: 0x8, opcode: PUSH2 0x0010 // pushes 2 byte 0x0010
PC: 0xb, opcode: JUMPI // jumps to 0x0010 if stack[1] is 1 
PC: 0xc, opcode: PUSH1 0x00 
PC: 0xe, opcode: DUP1 
PC: 0xf, opcode: REVERT // revert at stack[0] as memory location and stack[1] as reason (code here is entered when value was supplied in unpayable constructor)
PC: 0x10, opcode: JUMPDEST // indicates valid jump destinations
```

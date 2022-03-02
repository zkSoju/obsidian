## High Level Overview

When you write Solidity code it goes through a process like this:

- `myContract.sol`  → solc compiler → `.bin` (raw bytecode) → `.abi`  (interface file for Web3 frontends and applications to interact with contract)

The ABI is then used by a Web3 library to interact with a local or remote Ethereum node.

- web3 application → web3 library (ex. web3js/ethers) → Ethereum Client (instance of software running on a node on the network) → EVM

## VM Deep-dive

The EVM (Ethereum Virtual Machine) is a stack based VM in comparison to register based VMs like (x86 and ARM). All VMs at a low level run instructions/opcodes by going through a an execution loop that looks like:

1. Fetch instruction at PC (program counter)
2. Execute
3. If JUMP, set PC to new target
4. Else, increment PC 

![s](assets/image%204.png)

Characteristics of a stack based machine:

- Instructions take paramters from the stack and write results to the stack
- Each instruction has stack inputs, parameters, stack outputs, and return values
- All instructions are encoded on 1 bytes ([useful reference](obsidian://open?vault=Ethereum&file=Bytes%2C%20Bits%2C%20and%20Hex))
- Every instruction (aka. opcode) instruction is assigned a value between 0 and 255

EVM is a stack based computer, which can be thought of as a global dectralized computer with millions of execution objects. It is considered **quasi-Turing-complete** meaning that it has the capabilities of a **Turing-complete** system with limitations on computation by the amount of gas supplied.

Limiting the gas supplied solves two big problems:

- **Halting problem** which refers to the case where programs might halt
- Avoids situations where bad actors can deploy programs that run forever (DOS attack)

It is important to note the a transfer of value between one externally owned account (EOA) does not involve the use of EVM, but changes to global state do (particularlly with smart contracts). 

The EVM has special functionality and features that are geared towards usage on the Ethereum network. These features include:

- Addressing the halting problem (mentioned later) with gas
- Storage and memory primitives
- Execution contexts (caller, owner, balance, etc)
- Quality of life functions (type casting and keccak256)
- Cross contract communication 
	- `call()`
	-  `delegatecall()`
	- `return()`
	- `revert()`

## Smart Contracts and Opcodes

The EVM executes a smart contract by reading and executing each instruction one by one. If an instruction cannot be executed due to lack of values on stack or insufficient gas remaining, the execution reverts.

In order to deploy a contract or program, an Ethereum transaction must be sent by providing no data in the `to` address field.

### Ethereum Transactions

#### Types 
- **Sending a normal transaction**
- **Deploying a contract**
- **Function Call**
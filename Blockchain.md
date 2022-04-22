#### Blockchain
- Deterministic state machine replicated on full-nodes that retains conensus safety as long as less than 1/3 of maintainers are malicious
	- State machine - program that holds state and maintains it based on inputs
		- contains many things ex. token balances, txs
	- Deterministic - replaying same transactions of genesis block will result in same final state
	- Consensus - every honest node has a replicate of state 

**Application layer** - responsibile for updating state (ex. smart contracts and processing tx)
**Network layer** - propogates transactions and consensus messages
**Consensus layer** - enables nodes to agree on current state


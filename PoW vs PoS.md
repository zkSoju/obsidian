## PoW

- Rewards miner with higher hash rates and computing power because they will have a higher chance of creating the next block
- **Mining pools** - miners come together to combine hashing power and reward is distributed across everyone in pool
- Results in massive energy consumption and centralization through mining pools
- Users rely on the longest chain as source of truth because the most computational work is done
- Transactions are considered final when it is part of a block that can't change
	- It is possible for two blocks to be mined at the same time causing a temporary fork in the blockchain
	- A transaction can revert if the transaction in both blocks don't match (ex. rejected in one and accepted in another)
	- Therefore, a transaction is considered final after about 6 blocks

## PoS 

- To become a validator, a node has to stake a certain amount of tokens (32 ETH security deposit)
- Bigger the stake, higher the chance of being selected as the next validator
- Validators lose part of stake of fraudulent transactions are approved (slashing)
	- As long as stake is higher than sum of transaction fees in the block, then there is no incentive to submit fraudulent transactions
- Validators job is to order txs and create new blocks that all nodes can agree on 
- Vulnerable to 51% attack as with PoW; however, it is harder to obtain 51% of staking for temporary control or 51% of all coins for complete control
- Key improvements
	- Better energy efficiency - no competition to compute same block
	- Lower barrier to entry + reduced hardware requirement
	- More nodes, stronger immunity to centralization
	- Stronger support for sharding and shard chains
		- Separate blockchains that will need validators
		- Coordination between blockchain through a beacon chain
			- Runs in parallel with ETH chain
			- Purpose is to receive and sync information between shards
			- Registers validators and issues rewards and penalties
			- Psuedorandomly selects a validator based on stake and remaining nodes become attesters
			- Validator only proposes a solution where attesters vote on vs PoW where blocks are forged
		- Two options:
			- Fully executable shard chains
			- Shard chains will no support for transactions and smart contracts only extra data + use of layer 2 solutions where only proofs are submitted/stored
	- Docking
		- Migrate current chain from PoW to PoS
		- Fully executable PoS model and retain current ethereum state
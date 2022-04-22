# Cosmos
#Blockchain

https://v1.cosmos.network/intro

Inter-Blockchain Communication (IBC) aka. Blockchain Interoperability

Competing for throughput on a single blockchain results in high tx fees and network congestion. With Cosmos, developers are able to build autonomous application-specific blockchains that easily interconnect.

**Organizations** - enables communities to transparently dictate where funds/resoources are allocated through token voting

**Social Networks** - centralized platforms are censoring and filtering content, which extracts value from users; on Cosmos, social networks can be built giving people the voice

**Marketplaces** - enables creation of permissionless global trade through autonomous app-specific blockchains freeing users of high tx fees

**Games** - developers are able to create collectibles without third party approval and  permenance

## Cosmos Hub

- First blockchain to launch on Cosmos
- Intended to be service provider to chains connecting to it (think of it as a port city, which is valued by the trade that flows through it)
	- Offers a wealth of value-added services
		- Shared security
		- Interchain exchange
		- ETH-BTC bridge
		- Secure custodianship of digital assets
- Secured by ATOM
	- ATOM is the native token on Cosmos hub
	- Security provided by ATOM staking
		- Lock up digital asset (ATOM) to provide economic security for public blockchain
		- Earn staking rewards through validators
			- Transaction fees collected
			- Newly created ATOM through yearly supply inflation
		- Vote for proposals to make decisions on future of network
		- **Staking is not risk free**
			- Selecting multiple validators will minimize risk due to potential for validator to have downtime/underperform

## Cosmos Technology
- Cosmos is a decentralized network of independent parallel blockchain powered by BFT consensus algorithms like Tendermint
- Prior to cosmos, blockchain were difficult to build and siloed, unable to communite with each other

[Blockchain](../Blockchain.md)

## Internet of Blockchains

- Underlying technology uses open-source tooling Tendermint for the consensus and networking layer and Cosmos SDK and IBC for application layer
- Build custom, secure, scalable, and interoperable blockchain applications quickly
	- Monorepos like Bitcoin and Go-Ethereum are difficult to fork and manage
	- Tendermint BFT 
		- Combines networking and consensus layer into a generic engine allowing developers to focus on application
		- Connects to applications through a ABCI (Application Blockchain Interface)
		- Features
			- Public or private blockchain ready
				- Application layer defines the validator candidates
					- If elected by stake, then can be defined by PoS
					- If defined by restricted set of pre-authorized entities, then can be consider a private blockchain
			- High performance - 1000s TPS
			- Instant finality - transactions finalized as soon as block is created
			- Security
	- Cosmos SDK
		- Modularity
			- Ecosystem of modules to create application specific blockchains
		- Capablility-based security

### Why create a blockchain instead of deploying on a VM blockchain?

- Prior it was more difficult to develop blockchains than smart contracts
- Provides security, flexibility, sovereignty, and performance

## Connecting Blockchains together (IBC)

- Leverages instant finality of Tendermint BFT to allow heterogeneous chains to transfer value or data to each other
	- Meaning Bitcoin and Ethereum PoW is not supported because they have probabilistic finality 
	- Heterogeneous chains must
		- Have instant finality
		- Be maintained by a set of validators to agree on next block to commit

### How does it work?

**Example**: User from Chain A sends 10 ATOM to a user on Chain B

- Tracking
	- Each chain runs a light client of the other chains
		- Contains only the headers of blocks to verify the result of its queries
		- Lightweight alternative to full-nodes with good security guarantees 

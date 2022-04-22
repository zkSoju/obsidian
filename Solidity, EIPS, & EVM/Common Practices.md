## Re-entrancy Attack 

[Reference](https://solidity-by-example.org/hacks/re-entrancy/)

Reentrancy occurs when contract `A` calls contract `B`, but contract `B` is able to "re-enter" contract A prior to the execution completing. Observe the attack in the following:

Contract `A` 

```solidity
function deposit() public payable { 
	balances[msg.sender] += msg.value; 
} 

function withdraw() public { 
	uint bal = balances[msg.sender]; 
	require(bal > 0); 
	(bool sent, ) = msg.sender.call{value: bal}(""); 
	require(sent, "Failed to send Ether"); 
	balances[msg.sender] = 0; 
}
```

Contract `B` (malicious contract)

```solidity
fallback() external payable { 
	if (address(etherStore).balance >= 1 ether) { 
	    etherStore.withdraw(); 
	} 
}

function attack() external payable { 
	require(msg.value >= 1 ether); 
	etherStore.deposit{value: 1 ether}(); 
	etherStore.withdraw(); 
}
```

The reetrancy attack occurs when:
- Contract `B` calls contract A's `withdraw` function 
* Contract `A` forwards the balance through contract `B` `fallback()` function using `call()` before reducing the balance. 
* Contract `B`  fallback then calls Contract `A` withdraw again. Creating a loop enabling contract B to withdraw all of Contract `A`'s ether balance

For this reason it is advisable to avoid low level calls to contracts due to this exploitable behavior.

Preventative techniques
- Ensure state changes (like reducing balance) happen before calling external contract 
- Use function modifiers
- Strongly consider techniques when making calls to external contracts

Refer to [[../Code Review/Repository Breakdowns]] for repository deep dives and just reading good codes.
Refer to [[../Code Review/Pull Request Breakdowns]] for pull requests.

## Damn Vulnerable DeFi

**What is a flash loan?**
A flash loan allows users to borrow any available amount of assets without putting up collateral as long as transaction is returned in one block transaction.

1. Unstoppable Lender
	- https://www.damnvulnerabledefi.xyz/challenges/1.html
	- Find a way to stop the "unstoppable" lender from offering flash loans
	- **Vulnerability** is in the `assert(poolBalance == balanceBefore);` statement where poolBalance can be altered by sending tokens directly to the contract without modifying the poolBalance vs. using the deposit function
2. Naive Receiver
	- https://www.damnvulnerabledefi.xyz/challenges/2.html
	- Find a way to extract all the ether from the user deployed contract 
	- **Vulnerability** is in the `flashLoan(address borrower, uint256 amount)` being able to be called by any user and the amount being able to be 0; thereforre, and any user can call `flashLoan` multiple times on the user contract to extract the fee from the contract
	- **Solution**: Ensure that the person initiating is a trusted user of the deployer contract 
3. Truster
	- https://www.damnvulnerabledefi.xyz/challenges/3.html
	- Find a way to extract all the ether from the lending pool offering flash loans
	- **Vulnerability** is in the `target.call(data)` where you can call any arbitrary function from any target, which you can use to call an external contract to approve of total quantity and extract all ether
		- Use `bytes memory data = abi.encodeWithSignature("approve(address,uint256)", address(this), uint(-1))` for the data field
4. Side Entrance
	 - https://www.damnvulnerabledefi.xyz/challenges/4.html
	 - Find a way to extract all the ether from the lending pool offering flash loans
	 - **Vulnerability** is in the sequence of require and calls that allows caller to use `execute()` to `deposit()` any amount of ETH received from the lending pool back into the lending pool to withdraw later
5. The Rewarder
	- https://www.damnvulnerabledefi.xyz/challenges/5.html
	- Find a way to claim majority of rewards from reward pool for yourself in the next round
	- **Vulnerability** is that you can use the flash loaner pool to receive a flash loan, deposit loan into reward pool, withdraw funds, deposit funds back into flash loan contract all in a single transaction  and then withdraw rewards after.
6. Selfie
	- https://www.damnvulnerabledefi.xyz/challenges/6.html
	- Find a way to claim all the funds in the selfie pool with ERC20 token governance
	- **Vulnerability** is to:
		- use the flash loan to borrow more than half of the total supply 
		- take a snapshot
		- queue a withdraw of all funds
		- withdraw all funds in 2 days when action can be executed
7. Compromised
	- https://www.damnvulnerabledefi.xyz/challenges/7.html
	- Find a way to drain NFT exchange contract, where NFT prices are determined by a price oracle
	- **Vulnerability** is that pk of a trusted oracle is exposed in web service allowing you to take control, modify price to full balance, and sell an NFT for that price
8. Puppet
	- https://www.damnvulnerabledefi.xyz/challenges/8.html
	- There is a huge lending pool where you need to deposit twice the borrow amount as collateral, find a way to steal all tokens from lending pool with supplied ETH, DVTS, and Uniswap Exchange
	- **Vulnerability** is to manipulate the oracle to compute the ETH price of the token to 0 by skewing the Uniswap pool (large DVT value, small ETH value rounds down to 0), enabling the user to withdraw all funds
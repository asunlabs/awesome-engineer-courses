# Smart contract bad use cases

# Bad Randomness

Pseudorandom number generation on the blockchain is generally unsafe. There are a number of reasons for this, including:

- The blockchain does not provide any cryptographically secure source of randomness. Block hashes in isolation are cryptographically random, however, **a malicious miner can modify block headers**, introduce additional transactions, and choose not to publish blocks in order to influence the resulting hashes. Therefore, miner-influenced values like **block hashes and timestamps should never be used as a source of randomness**.

- Everything in a contract is publicly visible. Random numbers cannot be generated or stored in the contract until after all lottery entries have been stored.

- Computers will always be faster than the blockchain. Any number that the contract could generate can potentially be precalculated off-chain before the end of the block.

A common workaround for the lack of on-chain randomness is using a commit and reveal scheme. Here, each user submits the hash of their secret number.
When the time comes for the random number to be generated, each user sends their secret number to the contract, which then verifies it matches the hash submitted earlier and xors them together. Therefore no participant can observe how their contribution will affect the end result until after everyone has already committed to a value. 

However, this is also vulnerable to DoS attacks, since the last person to reveal can choose to never submit their secret. Even if the contract is allowed to move forward without everyone's secrets, this gives them influence over the end result. In general, we do not recommend commit and reveal schemes.

## Attack Scenarios

- A lottery where people bet on whether the hash of the current block is even or odd. A miner that bets on even **can throw out blocks whose hash are even**.
- A commit-reveal scheme where users don't necessarily have to reveal their secret (to prevent DoS). A user has money riding on the outcome
of the PRG and submits a large number of commits, allowing them to choose the one they want to reveal at the end.

## Mitigations

There are currently not any recommended mitigations for this issue.
Do not build applications that require on-chain randomness.
In the future, however, these approaches show promise

- [Verifiable delay functions](https://eprint.iacr.org/2018/601.pdf): functions which produce a pseudorandom number and take a fixed amount of sequential time to evaluate
- [Randao](https://github.com/randao/randao): A commit reveal scheme where users must stake wei to participate

## Examples

- The `random` function in [theRun](theRun_source_code/theRun.sol) was vulnerable to this attack. It used the blockhash, timestamp and block number to generate numbers in a range to determine winners of the lottery. To exploit this, **an attacker could set up a smart contract that generates numbers in the same way** and submits entries when it would win. As well, the miner of the block has some control over the blockhash and timestamp and would also be able to influence the lottery in their favor.

## Sources

- https://ethereum.stackexchange.com/questions/191/how-can-i-securely-generate-a-random-number-in-my-smart-contract
- https://blog.positive.com/predicting-random-numbers-in-ethereum-smart-contracts-e5358c6b8620

# Denial of Service

**A malicious contract** can permanently stall another contract **by failing in a strategic way**. In particular, contracts that bulk perform transactions or updates using a `for` loop can be DoS'd if a call to another contract or `transfer` fails during the loop. 

## Attack Scenarios

- Auction contract where frontrunner must be reimbursed when they are outbid. If the call refunding the frontrunner continuously fails, the auction is stalled and they become the de facto winner.

- Contract iterates through an array to pay back its users. **If one `transfer` fails in the middle of a `for` loop all reimbursements fail**.

- **Attacker spams contract, causing some array to become large**. Then `for` loops iterating through the array might **run out of gas and revert**.

## Examples

- Both [insecure](auction.sol#L4) and [secure](auction.sol#L26) versions of the auction contract mentioned above

- Bulk refund functionality that is [suceptible to DoS](list_dos.sol#L3), and a [secure](list_dos.sol#L29) version

## Mitigations

- Favor pull over push for external calls
- If iterating over a dynamically sized data structure, **be able to handle the case where the function takes multiple blocks to execute**. One strategy for this is storing iterator in a private variable and using `while` loop that exists when gas drops below certain threshold.

## References

- https://www.reddit.com/r/ethereum/comments/4ghzhv/governmentals_1100_eth_jackpot_payout_is_stuck/
- https://github.com/ConsenSys/smart-contract-best-practices#dos-with-unexpected-revert
- https://vessenes.com/ethereum-griefing-wallets-send-w-throw-considered-harmful/

# Contracts can be forced to receive ether

In certain circunstances, contracts can be forced to receive ether without triggering any code. This should be considered by the contract developers in order to avoid breaking important invariants in their code.

## Attack Scenario

An attacker can use a specially crafted contract to forceful send ether using `suicide` / `selfdestruct`:

```solidity
contract Sender {
  function receive_and_suicide(address target) payable {
    suicide(target);
  }
}
```

## Example

- The MyAdvancedToken contract in [coin.sol](coin.sol#L145) is vulnerable to this attack. It will stop the owner to perform the migration of the contract.

## Mitigations

There is no way to block the reception of ether. The only mitigation is to avoid assuming how the balance of the contract increases and implement checks to handle this type of edge cases.

## References

- https://solidity.readthedocs.io/en/develop/security-considerations.html#sending-and-receiving-ether

# Incorrect interface

A contract interface defines functions with a different type signature than the implementation, causing two different method id's to be created.
As a result, when the interfact is called, the fallback method will be executed.

## Attack Scenario

- The interface is incorrectly defined. `Alice.set(uint)` takes an `uint` in `Bob.sol` but `Alice.set(int)` a `int` in `Alice.sol`. The two interfaces will produce two differents method IDs. As a result, Bob will call the fallback function of Alice rather than of `set`.

## Mitigations

Verify that type signatures are identical between inferfaces and implementations.

## Example

We now walk through how to find this vulnerability in the [Alice](Alice.sol) and [Bob](Bob.sol) contracts in this repo.

First, get the bytecode and the abi of the contracts:
```̀bash 
$ solc --bin Alice.sol
6060604052341561000f57600080fd5b5b6101158061001f6000396000f300606060405236156051576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff1680633c6bb436146067578063a5d5e46514608d578063e5c19b2d1460ad575b3415605b57600080fd5b5b60016000819055505b005b3415607157600080fd5b607760cd565b6040518082815260200191505060405180910390f35b3415609757600080fd5b60ab600480803590602001909190505060d3565b005b341560b757600080fd5b60cb600480803590602001909190505060de565b005b60005481565b806000819055505b50565b806000819055505b505600a165627a7a723058207d0ad6d1ce356adf9fa0284c9f887bb4b912204886b731c37c2ae5d16aef19a20029
$ solc --abi Alice.sol
[{"constant":true,"inputs":[],"name":"val","outputs":[{"name":"","type":"int256"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"new_val","type":"int256"}],"name":"set_fixed","outputs":[],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"new_val","type":"int256"}],"name":"set","outputs":[],"payable":false,"type":"function"},{"payable":false,"type":"fallback"}]


$ solc --bin Bob.sol
6060604052341561000f57600080fd5b5b6101f58061001f6000396000f30060606040526000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff1680632801617e1461004957806390b2290e14610082575b600080fd5b341561005457600080fd5b610080600480803573ffffffffffffffffffffffffffffffffffffffff169060200190919050506100bb565b005b341561008d57600080fd5b6100b9600480803573ffffffffffffffffffffffffffffffffffffffff16906020019091905050610142565b005b8073ffffffffffffffffffffffffffffffffffffffff166360fe47b1602a6040518263ffffffff167c010000000000000000000000000000000000000000000000000000000002815260040180828152602001915050600060405180830381600087803b151561012a57600080fd5b6102c65a03f1151561013b57600080fd5b5050505b50565b8073ffffffffffffffffffffffffffffffffffffffff1663a5d5e465602a6040518263ffffffff167c010000000000000000000000000000000000000000000000000000000002815260040180828152602001915050600060405180830381600087803b15156101b157600080fd5b6102c65a03f115156101c257600080fd5b5050505b505600a165627a7a72305820f8c9dcade78d92097c18627223a8583507e9331ef1e5de02640ffc2e731111320029
$ solc --abi Bob.sol
[{"constant":false,"inputs":[{"name":"c","type":"address"}],"name":"set","outputs":[],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"c","type":"address"}],"name":"set_fixed","outputs":[],"payable":false,"type":"function"}]
```

The following commands were tested on a private blockchain

```javascript
$ get attach

// this unlock the account for a limited amount of time
// if you have an error:
// Error: authentication needed: password or unlock
// you can to call unlockAccount again
personal.unlockAccount(eth.accounts[0], "apasswordtochange")

var bytecodeAlice = '0x6060604052341561000f57600080fd5b5b6101158061001f6000396000f300606060405236156051576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff1680633c6bb436146067578063a5d5e46514608d578063e5c19b2d1460ad575b3415605b57600080fd5b5b60016000819055505b005b3415607157600080fd5b607760cd565b6040518082815260200191505060405180910390f35b3415609757600080fd5b60ab600480803590602001909190505060d3565b005b341560b757600080fd5b60cb600480803590602001909190505060de565b005b60005481565b806000819055505b50565b806000819055505b505600a165627a7a723058207d0ad6d1ce356adf9fa0284c9f887bb4b912204886b731c37c2ae5d16aef19a20029'
var abiAlice = [{"constant":true,"inputs":[],"name":"val","outputs":[{"name":"","type":"int256"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"new_val","type":"int256"}],"name":"set_fixed","outputs":[],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"new_val","type":"int256"}],"name":"set","outputs":[],"payable":false,"type":"function"},{"payable":false,"type":"fallback"}]

var bytecodeBob = '0x6060604052341561000f57600080fd5b5b6101f58061001f6000396000f30060606040526000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff1680632801617e1461004957806390b2290e14610082575b600080fd5b341561005457600080fd5b610080600480803573ffffffffffffffffffffffffffffffffffffffff169060200190919050506100bb565b005b341561008d57600080fd5b6100b9600480803573ffffffffffffffffffffffffffffffffffffffff16906020019091905050610142565b005b8073ffffffffffffffffffffffffffffffffffffffff166360fe47b1602a6040518263ffffffff167c010000000000000000000000000000000000000000000000000000000002815260040180828152602001915050600060405180830381600087803b151561012a57600080fd5b6102c65a03f1151561013b57600080fd5b5050505b50565b8073ffffffffffffffffffffffffffffffffffffffff1663a5d5e465602a6040518263ffffffff167c010000000000000000000000000000000000000000000000000000000002815260040180828152602001915050600060405180830381600087803b15156101b157600080fd5b6102c65a03f115156101c257600080fd5b5050505b505600a165627a7a72305820f8c9dcade78d92097c18627223a8583507e9331ef1e5de02640ffc2e731111320029'
var abiBob = [{"constant":false,"inputs":[{"name":"c","type":"address"}],"name":"set","outputs":[],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"c","type":"address"}],"name":"set_fixed","outputs":[],"payable":false,"type":"function"}]

var contractAlice = eth.contract(abiAlice); 
var txDeployAlice = {from:eth.coinbase, data: bytecodeAlice, gas: 1000000}; 
var contractPartialInstanceAlice = contractAlice.new(txDeployAlice); 

// Wait to mine the block containing the transaction

var alice = contractAlice.at(contractPartialInstanceAlice.address);

var contractBob = eth.contract(abiBob); 
var txDeployBob = {from:eth.coinbase, data: bytecodeBob, gas: 1000000}; 
var contractPartialInstanceBob = contractBob.new(txDeployBob); 

// Wait to mine the block containing the transaction

var bob = contractBob.at(contractPartialInstanceBob.address);

// From now, wait for each transaction to be mined before calling
// the others transactions

// print the default value of val: 0
alice.val() 

// call bob.set, as the interface is wrong, it will call
// the fallback function of alice
bob.set(alice.address, {from: eth.accounts[0]} )
// print val: 1
alice.val()

// call the fixed version of the interface
bob.set_fixed(alice.address, {from: eth.accounts[0]} )
// print val: 42
alice.val()
```

# Integer Overflow

It is possible to cause `add` and `sub` to overflow (or underflow) on any type of integer in Solidity.  

## Attack Scenarios

- Attacker has 5 of some ERC20 token. They spend 6, but because the token doesn't check for underflows,
they wind up with 2^256 tokens.

- A contract contains a dynamic array and an unsafe `pop` method. An attacker can underflow the length of
the array and alter other variables in the contract.

## Mitigations

- Use openZeppelin's [safeMath library](https://github.com/OpenZeppelin/openzeppelin-solidity/blob/master/contracts/math/SafeMath.sol)
- Validate all arithmetic

## Examples

- In [integer_overflow_1](interger_overflow_1.sol), we give both unsafe and safe version of
the `add` operation.

- [A submission](https://github.com/Arachnid/uscc/tree/master/submissions-2017/doughoyte) to the Underhanded Solidity Coding Contest that explots the unsafe dynamic array bug outlined above

# Race Condition
There is a gap between the creation of a transaction and the moment it is accepted in the blockchain.
Therefore, an attacker can take advantage of this gap to put a contract in a state that advantages them.

## Attack Scenario

- Bob creates `RaceCondition(100, token)`. Alice trusts `RaceCondition` with all its tokens. Alice calls `buy(150)`
Bob sees the transaction, and calls `changePrice(300)`. The transaction of Bob is mined before the one of Alice and
as a result, Bob received 300 tokens.

- The ERC20 standard's `approve` and `transferFrom` functions are vulnerable to a race condition. Suppose Alice has
approved Bob to spend 100 tokens on her behalf. She then decides to only approve him for 50 tokens and sends
a second `approve` transaction. However, Bob sees that he's about to be downgraded and quickly submits a
`transferFrom` for the original 100 tokens he was approved for. If this transaction gets mined before Alice's
second `approve`, Bob will be able to spend 150 of Alice's tokens.

## Mitigations

- For the ERC20 bug, insist that Alice only be able to `approve` Bob when he is approved for 0 tokens.
- Keep in mind that all transactions may be front-run

## Examples
- [Race condition](RaceCondition.sol) outlined in the first bullet point above

# Re-entrancy
A state variable is changed after a contract uses `call.value`. The attacker uses
[a fallback function](ReentrancyExploit.sol#L26-L33)—which is automatically executed after
Ether is transferred from the targeted contract—to execute the vulnerable function again, *before* the
state variable is changed.

## Attack Scenarios
- A contract that holds a map of account balances allows users to call a `withdraw` function. However,
`withdraw` calls `send` which transfers control to the calling contract, but doesn't decrease their
balance until after `send` has finished executing. The attacker can then repeatedly withdraw money
that they do not have.

## Mitigations

- Avoid use of `call.value`
- Update all bookkeeping state variables _before_ transferring execution to an external contract.

## Examples

- The [DAO](http://hackingdistributed.com/2016/06/18/analysis-of-the-dao-exploit/) hack
- The [SpankChain](https://medium.com/spankchain/we-got-spanked-what-we-know-so-far-d5ed3a0f38fe) hack

# Unchecked External Call

Certain Solidity operations known as "external calls", require the developer to manually ensure that the operation succeeded. This is in contrast to operations which throw an exception on failure. If an external call fails, but is not checked, the contract will continue execution as if the call succeeded. This will likely result in buggy and potentially exploitable behavior from the contract.

## Attack

- A contract uses an unchecked `address.send()` external call to transfer Ether.
- If it transfers Ether to an attacker contract, the attacker contract can reliably cause the external call to fail, for example, with a fallback function which intentionally runs out of gas.
- The consequences of this external call failing will be contract specific.
	- In the case of the King of the Ether contract, this resulted in accidental loss of Ether for some contract users, due to refunds not being sent.

## Mitigation

- Manually perform validation when making external calls
- Use `address.transfer()`

## Example

- [King of the Ether](https://www.kingoftheether.com/postmortem.html) (line numbers:
	[100](KotET_source_code/KingOfTheEtherThrone.sol#L100),
	[107](KotET_source_code/KingOfTheEtherThrone.sol#L107),
	[120](KotET_source_code/KingOfTheEtherThrone.sol#L120),
	[161](KotET_source_code/KingOfTheEtherThrone.sol#L161))

## References

- http://solidity.readthedocs.io/en/develop/security-considerations.html
- http://solidity.readthedocs.io/en/develop/types.html#members-of-addresses
- https://github.com/ConsenSys/smart-contract-best-practices#handle-errors-in-external-calls
- https://vessenes.com/ethereum-griefing-wallets-send-w-throw-considered-harmful/

# Unprotected function
Missing (or incorrectly used) modifier on a function allows an attacker to use sensitive functionality in the contract.

## Attack Scenario

A contract with a `changeOwner` function does not label it as `private` and therefore
allows anyone to become the contract owner.

## Mitigations

Always specify a modifier for functions.

## Examples
- An `onlyOwner` modifier is [defined but not used](Unprotected.sol), allowing anyone to become the `owner`
- April 2016: [Rubixi allows anyone to become owner](https://etherscan.io/address/0xe82719202e5965Cf5D9B6673B7503a3b92DE20be#code)
- July 2017: [Parity Wallet](https://blog.zeppelin.solutions/on-the-parity-wallet-multisig-hack-405a8c12e8f7). For code, see [initWallet](WalletLibrary_source_code/WalletLibrary.sol)
- BitGo Wallet v2 allows anyone to call tryInsertSequenceId. If you try close to MAXINT, no further transactions would be allowed. [Fix: make tryInsertSequenceId private.](https://github.com/BitGo/eth-multisig-v2/commit/8042188f08c879e06f097ae55c140e0aa7baaff8#diff-b498cc6fd64f83803c260abd8de0a8f5)
- Feb 2020: [Nexus Mutual's Oraclize callback was unprotected—allowing anyone to call it.](https://medium.com/nexus-mutual/responsible-vulnerability-disclosure-ece3fe3bcefa) Oraclize triggers a rebalance to occur via Uniswap.

# Variable Shadowing
Variable shadowing occurs when a variable declared within a certain scope (decision block, method, or inner class)
has the same name as a variable declared in an outer scope.

## Attack
This depends a lot on the code of the contract itself. For instance, in the [this example](inherited_state.sol), variable shadowing prevents the owner of contract `C` from performing self destruct

## Mitigation
The solidity compiler has [some checks](https://github.com/ethereum/solidity/issues/973) to emit warnings when 
it detects this kind of issue, but [it has known examples](https://github.com/ethereum/solidity/issues/2563) where 
it fails.

# Wrong Constructor Name

A function intended to be a constructor is named incorrectly, which causes it to end up in the runtime bytecode instead of being a constructor.

## Attack
Anyone can call the function that was supposed to be the constructor.
As a result anyone can change the state variables initialized in this function.

## Mitigations

- Use `constructor` instead of a named constructor

## Examples
- [Rubixi](Rubixi_source_code/Rubixi.sol) uses `DynamicPyramid` instead of `Rubixi` as a constructor
- An [incorrectly named constructor](incorrect_constructor.sol)

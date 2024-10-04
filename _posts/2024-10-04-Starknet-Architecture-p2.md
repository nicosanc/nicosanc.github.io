---
title: "Blockchain: Starknet Architecture Pt. 2"
---

### Architecture Continued

Types of Transactions:
Declare
- Registers new code on SN
Invoke + Universal Deployer
- Executes "write" functions
Deploy_Account 
- Deploys an account contract (smart wallet)
All TXs that modify the global state require gas fees
- Read only TXs do not require gas fees since state isn't being changed

Counterfactual Deployment
Create Smart Wallet
- Deploy account contract (AC)
- Deploy AC -> Pay gas fees -> smart wallet

How do you deploy an AC if you don't have any AC to pay gas fees
- Calculate the address off-chain
- Send funds to the would-be address
- Deploy the AC using the deploy_account TX
- Sequencer deducts gas fees from address

Recap: Imagine you install braavos and make a new wallet. Truthfully, all that braavos does in the background is find an address for your wallet, which is defined before your wallet is deployed. You can then send funds to that address, so that when you send your deploy account TX it just takes some of the money you funded earlier in that same address

When you pay for gas fees when sending TXs from your smart wallet to SN, it takes part of those funds to pay Ethereum for the verification transaction to finalize it on L1
- Spikes can happen where the ETH fee might grow during transaction process

### Summary
Types:
- declare
- invoke
- deploy_account
Lifecycle:
- Received -> when mempool sees the transaction
- Accepted on L2 -> when it gets compiled in cairovm
- Accepted on L1 -> When it is verified on ethereum
- Rejected/Reverted 
	- Can be rejected if the signature is invalid (L2 specific)
	- Reverted if you run out of funds or some other extraneous error
- Counterfactual Deployment -> find an address, send funds, THEN deploy account so you can pay for gas

### Liveness problem
State N -> State N + 1
- To make this transaction to change global state, you first need the current state
	- You get current state from L2 full nodes (juno, papyrus, pathfinder), or the Sequencer
	- But what if the sequencer stopped responding and all L2 full nodes stopped responding?
		- This means you can't know the current state, and thus you can't transition to a new state -> SN gets stuck! Remember it's a state machine

Recreating L2 state from L1
- So what do we do to prevent this?
	- Send a **State Difference** to an Ethereum blob, that shows the diff between N and N+1
	- Think of it as a bucket of unstructured data (like a data lake), the blob doesn't care what the data is, it just holds it
	- The collection of states is deleted after a month

### Reducing Data Availability costs
EIP-4844
- Sending DA data to L1 as blobs
- Different Type of L1 TX
- Temporary storage 1 month
- Cheaper than call data
- Subcent TXs
Volition
- Option to store da data off-chain per TX
- off-chain da is cheaper than blobs
- boost to fully on chain games
- implementation date tbd
### Blockchain Writer
In the SHARP module, the prover uses the CairoVM compilation trace, then sends the output to a Blockchain Writer, that includes an ETH EOA to make the transaction with ethereum

### The Starknet OS
Why is the state diff an output of the Prover?
Takes in 4 inputs:
- Current state
- new blocks
- class hashes
- compiled class hashes
Output:
- state diff
- class hashes
- compiled class hashes

### Data Availability (DA) Modes
Rollup
- L2s that use ethereum for DA
- Better liveness but more expensive
- Starknet is a rollup
Validium
- L2s that don't use ethereum for DA
- Worse liveness but cheaper
Volition 
- L2s that allow users to choose between using Ethereum for DA or an off-chain data availability solution
- Flexible balance between liveness and cost depending on the user's needs


### Summary
DA
- DA prevents SN from getting stuck (liveness)
- SN sends state diffs, class hashes, and compiled class hashes as DA
- These are outputs of the StarknetOS program
- DA data is posted to ethereum using blobs
- Starknet is the cheapest L2

### Finality
Recap:
- ethereum ~6 minutes
- SN ~2 hrs
- L2 ~3 sec
L2
- Takes 3 seconds
- Trust starkware
- Used by most apps
L1
- 2 hrs
- trustless 
- used by bridges

### Mempool
There's no fee market on SN yet
- TXs are executed in FIFO 
- You can't pay more to jump the queue
- SN mempool: FIFO queue + basic TX validation
  
### The pending block
- TX gets added to the pending block
- L2 finality takes 3 seconds
- TX gets executed on accepted_on_L2
- Same process repeats until the block has enough capacity to finalize block
- It actually takes 3 minutes for the block to close off its doors to other TXs and finally be created
	- Then 2 hrs to be finalized in SN after L1 validation

### What's a full node
A full node answers questions about SN state
Users don't directly interact with the sequencer, only full nodes do
equivalent to an execution client on ethereum
Starknet has 3 diff Full Nodes (clients)
- Juno: Nethermind, Go
- Pathfinder: EQlabs, rust
- Papyrus: Starkware, rust
Synced by a smart contract in L1 called SN core, which takes blocks one at a time with block #, root hash, and block hash.

### Recursive Proving
Execution vs Proving
- O(nlogn) for the prover to generate a validity proof
- O(n) for the Sequencer to execute a transaction
- O(logn) for verification

What if we create proofs in sequence with a single prover?
- Smaller jobs with less TXs
	- Faster L1 finality
	- Higher TX cost
- large jobs with lots of TXs
	- Slower L1 finality
	- Lower TX cost

Creating proofs in parallel with multiple provers
- Package many provers proofs, then prove a bundle of bundles, recursively
- Fast L1 finality since its bundles of bundles of proofs for efficiency
- Low TX cost cause many are done at once

### Summary
- Sharp serves multiple protocols
- Sharp has multiple provers
- Starks allows for parallelization
- Combined using recursion
- More operations on a single L1 proof
- Reduces L2 TX cost
- Reduces time to L1 finality 
- Reduces Prover specs


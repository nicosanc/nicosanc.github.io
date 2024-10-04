### Macro comparison to Ethereum

| FEATURE                  | SOLANA                                         | ETHEREUM                                          |
| ------------------------ | ---------------------------------------------- | ------------------------------------------------- |
| Consensus<br>Mechanism   | Proof of History (PoH)<br>Proof of Stake (PoS) | Proof of Stake                                    |
| Transaction Speed        | ~65,000 TPS (theoretical)                      | ~ 30 TPS (ETH 1.0)<br>~ 100,000 TPS (ETH 2.0)     |
| Block Time               | ~ 400 ms                                       | ~12 s                                             |
| Scalability              | High, due to parallel processing (SeaLevel)    | Limited by network congestion (L2s boost this)    |
| Programming Language     | Rust, C, C++                                   | Solidity, Vyper                                   |
| Transaction Fees         | Very low, fractions of a cent                  | Variable, often higher depending on network usage |
| Smart Contract Execution | Optimized with SeaLevel for parallel execution | EVM (Ethereum Virtual Machine)                    |
| Node Requirements        | High hardware Requirements for validators      | Lower Hardware requirements for validators        |

-----------------------------------------------------------

##### Unique aspects of solana
- PoH: cryptographic clock that orders events, allowing high throughput
- SeaLevel: Solana's parallel smart contract runtime allowing simultaneous transactions
- Low Fees: Consistently low TX fees due to efficient processing

##### Unique aspects of ethereum
- EVM Compatibility: Broad support and compatibility with existing dApps
- Strong Ecosystem: Large dev community and extensive infrastructure
- L2 solutions: Scaling solutions like Rollups to improve TX speed and reduce costs

##### Block Generation
Key components:
- PoH
	- **Cryptographic Clock**: PoH creates a historical record that proves that an event has occurred at a specific moment in time. The sequence of events is recorded before consensus
	- **Verification**: Validators create and validate blocks based on their stake, ensuring network security and integrity 
- PoS:
	- **Validator Selection**: Validators are chosen based on the amount of SOL they have staked
	- **Block Generation**: Validators create and validate blocks based on their stake, ensuring network security and integrity
- Block Generation Process
	- **Leader Schedule**: Solana uses a leader schedule, where a validator (leader) is chosen to create a block for a short period of time (~ 400 ms)
	- **Parallel Processing**: TXs are processed in parallel using SeaLevel, Solana's runtime for smart contracts, allowing multiple TXs to be executed

##### SeaLevel
- Parallel TX Execution
	- **Non-overlapping TXs**: can process TXs in parallel if they don't modify the same state
	- **Read and Write Locks**: Ensures that TXs that do access the same state do so without conflicts. TXs reading the same data can run in parallel, while those writing to the same data are serialized (done in sequence)
- Stateless Execution
	- **Isolated Program Execution**: Smart Contracts on Solana are stateless. They receive all necessary data as input and produce a result without maintaining internal state across executions
		- This means they only use the input data to compute TXs and don't worry about data from previous runs
	- **Account model**: State is stored in accounts, which can be read and modified by TXs. This model decouples contract execution from state management, facilitating parallelism
- Execution Scheduling
	- **Optimized Scheduling**: schedules TXs to maximize parallelism. It analyzes dependencies between transactions and groups non-conflicting transactions to run concurrently
- Hardware Utilization
	- **GPU and Multi-core Processing**: leverages the capabilities of modern hardware, including GPUs and multi-core processors, to execute multiple transactions at the same time. Harnesses the full computational power of validator nodes

##### Account Data Structure
- Wallet Accounts
	- **Purpose**: Hold tokens
	- **Function**: Manage user balances and sign TXs
- Program Accounts:
	- **Purpose**: Store executable bytecode 
	- **Function**: execute smart contract logic
- Data Accounts:
	- Store state data used by programs
	- Maintain state across TXs
- Account Characteristics
	- **Ownership**: Who owns the account
	- **Rent**: The balance that must be maintained to keep the account alive
	- **Data Storage**: Can store arbitrary data for use by programs

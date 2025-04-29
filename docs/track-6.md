# Track 6: Consensus Bridge

## Overview

Following the Merge, Ethereum's architecture was split into two layers: the Execution Layer (EL) and the Consensus Layer (CL). This track explores the critical interface between these layers: the Engine API. We'll examine how EL clients like Reth communicate with CL clients to coordinate block production, validation, and finalization. The hands-on component involves building a minimal CL stub that can drive Reth's block processing functions, providing direct experience with this crucial interaction point.

## Learning Objectives

By the end of this track, you will be able to:
- Explain the post-Merge architecture and interaction between EL and CL clients
- Understand the Engine API specification and its key methods
- Describe the fork-choice rule and its role in determining the canonical chain
- Build a minimal CL stub that can drive block production in Reth
- Test the interaction between your CL stub and Reth using the Engine API

## Core Concepts

### Post-Merge Architecture

The Merge fundamentally changed Ethereum's architectural model by separating consensus from execution.

#### Two-Layer Design

Ethereum's post-Merge architecture consists of:

1. **Consensus Layer (CL)**:
   - Implements Proof of Stake consensus
   - Manages validator operations
   - Determines the canonical chain (fork choice)
   - Coordinates block production
   - Examples: Lighthouse, Prysm, Nimbus, Teku

2. **Execution Layer (EL)**:
   - Processes transactions
   - Maintains state (accounts, balances, contract code)
   - Executes EVM code
   - Serves JSON-RPC API requests
   - Examples: Geth, Nethermind, Erigon, Reth

#### Separation of Concerns

This separation provides several benefits:
- Modular development (consensus and execution evolve independently)
- Shared security across execution clients
- Cleaner upgrade paths
- Specialized optimization

#### Communication Model

The two layers communicate through a local IPC/HTTP connection:
- CL client drives the EL client
- Both run on the same machine in typical setups
- Communication happens via a standardized JSON-RPC interface (Engine API)
- The EL client no longer determines the canonical chain independently

### The Engine API

The Engine API is the standardized interface between EL and CL clients, defined in the [execution-apis](https://github.com/ethereum/execution-apis) repository.

#### Key Methods

1. **engine_newPayloadV***: 
   - Called when the CL receives a new block
   - Asks the EL to validate and execute the block
   - EL returns validation status and state root
   - Latest version (post-Cancun): `engine_newPayloadV3`

2. **engine_forkchoiceUpdatedV***: 
   - Updates the EL about the current head of the chain
   - Can include a request to build a new block
   - Latest version (post-Cancun): `engine_forkchoiceUpdatedV3`

3. **engine_getPayloadV***: 
   - Retrieves a previously requested execution payload
   - Used when a validator is selected to propose a block
   - Latest version (post-Cancun): `engine_getPayloadV3`

4. **engine_exchangeTransitionConfigurationV1**:
   - Ensures EL and CL have compatible configurations
   - Verifies chain ID, fork configuration, and terminal blocks

#### Versioning

The Engine API methods are versioned to accommodate protocol upgrades:
- **V1**: Original Merge-compatible version
- **V2**: Shanghai/Capella upgrade (added withdrawal support)
- **V3**: Cancun/Deneb upgrade (added blob transaction support)

#### Authentication

Engine API methods use an authenticated endpoint:
- JWT (JSON Web Token) authentication
- Shared secret between EL and CL
- Required for security (prevents malicious payload injection)

### Block Production Flow

The interaction between EL and CL during block production involves several steps:

1. **Validator Selection**: 
   - CL selects a validator for the next slot based on the beacon chain
   - Selected validator's CL prepares to create a block

2. **Payload Preparation**:
   - CL calls `engine_forkchoiceUpdatedV*` with `payloadAttributes` to request new block
   - EL begins assembling transactions into an execution payload
   - EL returns a `payloadId` to identify this building process

3. **Payload Retrieval**:
   - CL calls `engine_getPayloadV*` with the `payloadId`
   - EL returns the completed execution payload (block)

4. **Beacon Block Creation**:
   - CL wraps the execution payload in a beacon block
   - CL signs and broadcasts the complete beacon block

5. **Block Propagation**:
   - Other nodes receive the beacon block via CL networking
   - Their CL clients extract the execution payload

6. **Block Validation**:
   - Receiving nodes' CL clients call `engine_newPayloadV*` on their EL
   - EL validates and executes the block
   - EL returns validation status to CL

7. **Fork Choice Update**:
   - If valid, CL calls `engine_forkchoiceUpdatedV*` to update the chain head
   - EL updates its view of the canonical chain

### Fork Choice Rule

The fork choice rule determines which chain is considered canonical when multiple valid chains exist.

#### LMD-GHOST

Ethereum's Proof of Stake uses a fork choice rule called LMD-GHOST (Latest Message Driven Greedy Heaviest Observed Subtree):

- **Latest Message Driven**: Only considers the most recent attestation from each validator
- **Greedy Heaviest Observed Subtree**: Selects the chain with the most attestations by weight

#### Fork Choice Factors

Several factors influence the fork choice:
- **Attestations**: Validator votes for blocks
- **Justification**: When more than 2/3 of validators attest to a checkpoint
- **Finalization**: When consecutive epochs are justified
- **Slashing Conditions**: Validators violating protocol rules

#### EL Client Perspective

From the EL client's perspective, fork choice is simplified:
- EL receives fork choice updates from CL
- EL maintains a chain of valid blocks
- EL updates its head based on CL directives
- EL does not independently determine the canonical chain

### Reth's Engine API Implementation

Reth implements the Engine API as specified in the execution-apis repository.

#### Key Components

1. **Engine API Module**: 
   - Implements the JSON-RPC methods
   - Handles authentication
   - Processes payload validation
   - Manages fork choice updates

2. **Payload Builder**:
   - Assembles transactions into blocks
   - Handles blob transactions (post-Cancun)
   - Computes state transitions
   - Manages block sealing

3. **Chain Reorganization**:
   - Handles chain reorganizations directed by the CL
   - Updates canonical head
   - Manages transaction pool revalidation

## Exercise: Building a Minimal CL Stub

### Objective

Create a minimal Consensus Layer (CL) stub that can drive Reth's block processing functions via the Engine API.

### Prerequisites

- Completed Tracks 0-5
- Understanding of HTTP/JSON-RPC
- Familiarity with JWT authentication
- Knowledge of beacon chain concepts (blocks, slots)

### Tools and Libraries

- Reth codebase
- Rust HTTP client libraries (reqwest, hyper)
- JSON serialization (serde)
- JWT implementation (jsonwebtoken)

### Tasks

1. **Set Up the Project Structure:**

   Create a new Rust crate for your CL stub:
   ```bash
   # In the course repository
   cd modules
   cargo new cl-stub --bin
   cd cl-stub
   ```

   Add necessary dependencies to Cargo.toml:
   ```toml
   [dependencies]
   reqwest = { version = "0.11", features = ["json"] }
   tokio = { version = "1", features = ["full"] }
   serde = { version = "1.0", features = ["derive"] }
   serde_json = "1.0"
   jsonwebtoken = "8.1"
   clap = { version = "4.0", features = ["derive"] }
   ethereum_types = "0.14"
   hex = "0.4"
   rand = "0.8"
   ```

2. **Define Engine API Data Structures:**

   Create Rust structures representing Engine API objects:
   ```rust
   use ethereum_types::{H256, U256, Address, Bloom};
   use serde::{Serialize, Deserialize};
   
   #[derive(Debug, Serialize, Deserialize)]
   struct ExecutionPayload {
       parent_hash: H256,
       fee_recipient: Address,
       state_root: H256,
       receipts_root: H256,
       logs_bloom: Bloom,
       prev_randao: H256,
       block_number: u64,
       gas_limit: u64,
       gas_used: u64,
       timestamp: u64,
       extra_data: Vec<u8>,
       base_fee_per_gas: U256,
       block_hash: H256,
       transactions: Vec<Vec<u8>>,
       // Additional fields for Cancun
       withdrawals: Option<Vec<Withdrawal>>,
       blob_gas_used: Option<u64>,
       excess_blob_gas: Option<u64>,
   }
   
   #[derive(Debug, Serialize, Deserialize)]
   struct PayloadAttributes {
       timestamp: u64,
       prev_randao: H256,
       suggested_fee_recipient: Address,
       withdrawals: Option<Vec<Withdrawal>>,
       parent_beacon_block_root: Option<H256>,
   }
   
   #[derive(Debug, Serialize, Deserialize)]
   struct ForkchoiceState {
       head_block_hash: H256,
       safe_block_hash: H256,
       finalized_block_hash: H256,
   }
   
   #[derive(Debug, Serialize, Deserialize)]
   struct Withdrawal {
       index: u64,
       validator_index: u64,
       address: Address,
       amount: u64,
   }
   ```

3. **Implement JWT Authentication:**

   Create functions to generate and use JWT tokens:
   ```rust
   use jsonwebtoken::{encode, Header, EncodingKey, Algorithm};
   use std::time::{SystemTime, UNIX_EPOCH};
   use std::fs;
   
   fn generate_jwt(jwt_secret_path: &str) -> Result<String, Box<dyn std::error::Error>> {
       let secret = fs::read(jwt_secret_path)?;
       let now = SystemTime::now().duration_since(UNIX_EPOCH)?.as_secs();
       
       let claims = serde_json::json!({
           "iat": now,
           "exp": now + 60, // 1 minute expiry
       });
       
       let token = encode(
           &Header::new(Algorithm::HS256),
           &claims,
           &EncodingKey::from_secret(&secret)
       )?;
       
       Ok(token)
   }
   ```

4. **Implement Engine API Client:**

   Create a client that can interact with Reth's Engine API:
   ```rust
   struct EngineApiClient {
       http_client: reqwest::Client,
       endpoint: String,
       jwt_secret_path: String,
   }
   
   impl EngineApiClient {
       fn new(endpoint: String, jwt_secret_path: String) -> Self {
           let http_client = reqwest::Client::new();
           Self { http_client, endpoint, jwt_secret_path }
       }
       
       async fn fork_choice_updated(
           &self,
           fork_choice_state: ForkchoiceState,
           payload_attributes: Option<PayloadAttributes>,
       ) -> Result<serde_json::Value, Box<dyn std::error::Error>> {
           let token = generate_jwt(&self.jwt_secret_path)?;
           
           let response = self.http_client
               .post(&self.endpoint)
               .header("Authorization", format!("Bearer {}", token))
               .json(&serde_json::json!({
                   "jsonrpc": "2.0",
                   "method": "engine_forkchoiceUpdatedV3",
                   "params": [fork_choice_state, payload_attributes],
                   "id": 1
               }))
               .send()
               .await?
               .json::<serde_json::Value>()
               .await?;
           
           Ok(response)
       }
       
       async fn get_payload(
           &self,
           payload_id: String,
       ) -> Result<serde_json::Value, Box<dyn std::error::Error>> {
           let token = generate_jwt(&self.jwt_secret_path)?;
           
           let response = self.http_client
               .post(&self.endpoint)
               .header("Authorization", format!("Bearer {}", token))
               .json(&serde_json::json!({
                   "jsonrpc": "2.0",
                   "method": "engine_getPayloadV3",
                   "params": [payload_id],
                   "id": 1
               }))
               .send()
               .await?
               .json::<serde_json::Value>()
               .await?;
           
           Ok(response)
       }
       
       async fn new_payload(
           &self,
           payload: ExecutionPayload,
       ) -> Result<serde_json::Value, Box<dyn std::error::Error>> {
           let token = generate_jwt(&self.jwt_secret_path)?;
           
           let response = self.http_client
               .post(&self.endpoint)
               .header("Authorization", format!("Bearer {}", token))
               .json(&serde_json::json!({
                   "jsonrpc": "2.0",
                   "method": "engine_newPayloadV3",
                   "params": [payload],
                   "id": 1
               }))
               .send()
               .await?
               .json::<serde_json::Value>()
               .await?;
           
           Ok(response)
       }
   }
   ```

5. **Create Block Production Logic:**

   Implement a simple block production loop:
   ```rust
   use tokio::time::{sleep, Duration};
   
   async fn produce_blocks(
       client: &EngineApiClient,
       starting_block: u64,
       block_time_seconds: u64,
       fee_recipient: Address,
   ) -> Result<(), Box<dyn std::error::Error>> {
       let mut current_block = starting_block;
       let mut last_block_hash = H256::zero(); // You'd need to set this to a valid block hash
       
       // In a real implementation, you'd get this from the beacon chain
       let random_bytes = rand::random::<[u8; 32]>();
       let prev_randao = H256::from_slice(&random_bytes);
       
       println!("Starting block production from block {}", current_block);
       
       loop {
           // 1. Update fork choice to current head
           let fork_choice_state = ForkchoiceState {
               head_block_hash: last_block_hash,
               safe_block_hash: last_block_hash,
               finalized_block_hash: last_block_hash,
           };
           
           // 2. Request to build a new block
           let timestamp = SystemTime::now()
               .duration_since(UNIX_EPOCH)?
               .as_secs();
           
           let payload_attributes = PayloadAttributes {
               timestamp,
               prev_randao,
               suggested_fee_recipient: fee_recipient,
               withdrawals: Some(vec![]), // Empty for simplicity
               parent_beacon_block_root: None,
           };
           
           let fcli_response = client
               .fork_choice_updated(fork_choice_state, Some(payload_attributes))
               .await?;
           
           // Extract payloadId from response
           let payload_id = match fcli_response["result"]["payloadId"].as_str() {
               Some(id) => id.to_string(),
               None => {
                   eprintln!("Failed to get payloadId: {:?}", fcli_response);
                   continue;
               }
           };
           
           // 3. Wait a short time for transactions to be included
           sleep(Duration::from_secs(2)).await;
           
           // 4. Get the built payload
           let payload_response = client.get_payload(payload_id).await?;
           let payload = match payload_response["result"]["executionPayload"].as_object() {
               Some(p) => p,
               None => {
                   eprintln!("Failed to get payload: {:?}", payload_response);
                   continue;
               }
           };
           
           // Deserialize the payload (in a real implementation, properly parse this)
           let payload_hash = match payload["blockHash"].as_str() {
               Some(h) => H256::from_slice(&hex::decode(&h[2..])?),
               None => {
                   eprintln!("Failed to get block hash");
                   continue;
               }
           };
           
           println!("Built block {} with hash {}", current_block, hex::encode(payload_hash));
           
           // 5. Submit the payload as a new block
           let new_payload_response = client.new_payload(serde_json::from_value(
               payload_response["result"]["executionPayload"].clone()
           )?).await?;
           
           println!("New payload status: {:?}", new_payload_response["result"]["status"]);
           
           // 6. Update fork choice to the new block
           let new_fork_choice_state = ForkchoiceState {
               head_block_hash: payload_hash,
               safe_block_hash: payload_hash,
               finalized_block_hash: last_block_hash, // Not finalizing yet for simplicity
           };
           
           let final_fcli_response = client
               .fork_choice_updated(new_fork_choice_state, None)
               .await?;
           
           println!("Fork choice updated: {:?}", final_fcli_response["result"]["status"]);
           
           // Update state for next block
           last_block_hash = payload_hash;
           current_block += 1;
           
           // Wait for the next block time
           sleep(Duration::from_secs(block_time_seconds)).await;
       }
   }
   ```

6. **Create Main Function and CLI:**

   Set up the command-line interface:
   ```rust
   use clap::Parser;
   
   #[derive(Parser)]
   #[command(name = "cl-stub")]
   #[command(about = "A minimal Consensus Layer stub for driving Reth")]
   struct Cli {
       #[arg(long, default_value = "http://localhost:8551")]
       engine_api_url: String,
       
       #[arg(long, default_value = "jwt-secret.hex")]
       jwt_secret_path: String,
       
       #[arg(long, default_value = "12")]
       block_time: u64,
       
       #[arg(long)]
       starting_block: Option<u64>,
       
       #[arg(long, default_value = "0x0000000000000000000000000000000000000000")]
       fee_recipient: String,
   }
   
   #[tokio::main]
   async fn main() -> Result<(), Box<dyn std::error::Error>> {
       let args = Cli::parse();
       
       let fee_recipient = Address::from_slice(
           &hex::decode(args.fee_recipient.trim_start_matches("0x"))?
       );
       
       let client = EngineApiClient::new(
           args.engine_api_url,
           args.jwt_secret_path,
       );
       
       // In a real implementation, you'd query for the current block
       let starting_block = args.starting_block.unwrap_or(1);
       
       produce_blocks(
           &client,
           starting_block,
           args.block_time,
           fee_recipient,
       ).await?;
       
       Ok(())
   }
   ```

7. **Generate JWT Secret and Run Reth:**

   Generate a JWT secret and start Reth with Engine API enabled:
   ```bash
   # Generate JWT secret
   openssl rand -hex 32 > jwt-secret.hex
   
   # Run Reth with Engine API enabled
   reth node --dev --engine-api --engine-jwt-secret jwt-secret.hex
   ```

8. **Run Your CL Stub:**

   In a separate terminal, build and run your CL stub:
   ```bash
   cd modules/cl-stub
   cargo run -- --jwt-secret-path ../../jwt-secret.hex
   ```

9. **Observe and Analyze:**

   Monitor the output of both Reth and your CL stub:
   - Watch block production in your CL stub
   - Check logs in Reth for Engine API interactions
   - Use `reth debug` commands to inspect the chain state

10. **Enhance Your CL Stub (Optional):**

    Add more advanced features:
    - Transaction simulation for more realistic blocks
    - Proper error handling and retries
    - Support for chain reorganizations
    - Metrics collection on block production

### Expected Outcome

After completing this exercise, you should have:
- A functional CL stub that can drive Reth's block production
- Understanding of the Engine API and its methods
- Experience with the block production and validation flow
- Insight into how EL and CL clients interact post-Merge

### Verification

Your solution should:
- Successfully produce a series of blocks in Reth
- Properly handle fork choice updates
- Maintain consistent chain state
- Follow the correct Engine API calling sequence

### Troubleshooting

Common challenges include:
- **JWT Authentication Issues:** Ensure the secret is correctly generated and used
- **API Version Mismatch:** Confirm you're using the version supported by your Reth build
- **Data Serialization Errors:** Check object formats match the API specification
- **Timing Issues:** Ensure proper sequencing of API calls for block production

## Conclusion

You've now explored the critical interface between Execution Layer and Consensus Layer clients: the Engine API. By building a minimal CL stub, you've gained practical experience with the post-Merge architecture and the mechanisms that coordinate consensus and execution in modern Ethereum.

In the next track, we'll delve into recent protocol upgrades, focusing on blob transactions introduced in Cancun/Deneb and the ongoing work on the EVM Object Format (EOF).

## Further Reading

- [Engine API Specification](https://github.com/ethereum/execution-apis/tree/main/src/engine)
- [The Merge Architecture](https://ethereum.org/en/developers/docs/apis/backend/#engine-api)
- [Reth Engine API Documentation](https://paradigmxyz.github.io/reth/engine/overview.html)
- [Beacon Chain Specification](https://github.com/ethereum/consensus-specs)
- [Fork Choice Specification](https://github.com/ethereum/consensus-specs/blob/dev/specs/phase0/fork-choice.md)
- [JWT Authentication for Engine API](https://github.com/ethereum/execution-apis/blob/main/src/engine/authentication.md) 
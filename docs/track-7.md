# Track 7: Modern Upgrades

## Overview

Ethereum continuously evolves through protocol upgrades that enhance its functionality, security, and scalability. This track explores recent and upcoming protocol changes, with specific focus on the Cancun/Deneb upgrade that introduced blob transactions (EIP-4844) and the in-development EVM Object Format (EOF). You'll gain a deep understanding of how these upgrades are implemented in client software and hands-on experience by implementing blob transaction decoding in Reth.

## Learning Objectives

By the end of this track, you will be able to:
- Understand the motivation and technical details of EIP-4844 (Proto-Danksharding)
- Explain the structure and purpose of the EVM Object Format (EOF)
- Describe the process of implementing protocol upgrades in Ethereum clients
- Implement and test blob transaction decoding in Rust
- Navigate and modify complex protocol-related code in Reth

## Core Concepts

### EIP-4844: Blob Transactions (Proto-Danksharding)

EIP-4844, also known as Proto-Danksharding, is a significant upgrade that introduced a new transaction type designed to reduce costs for Layer 2 rollups.

#### Motivation and Goals

Layer 2 rollups publish transaction data to Ethereum L1 as calldata, which is expensive and competes with regular transactions. EIP-4844 addresses this by:
- Creating a dedicated transaction type for data blobs
- Implementing a separate fee market for blob data
- Establishing a new data availability mechanism
- Preparing for full Danksharding in the future

#### Blob Transaction Structure

Blob transactions (Type 3) extend the EIP-2718 transaction format with blob-specific fields:

- **max_fee_per_blob_gas**: Maximum price per blob gas unit
- **blob_versioned_hashes**: List of KZG commitment versioned hashes
- **blob_data**: The blobs (only included in the P2P network, not in blocks)

The transaction format follows:
```
0x03 || rlp([chain_id, nonce, max_priority_fee_per_gas, max_fee_per_gas, gas_limit, to, value, data, access_list, max_fee_per_blob_gas, blob_versioned_hashes, y_parity, r, s])
```

#### Blob Gas and Fees

EIP-4844 introduces a separate fee mechanism for blob data:
- **Blob Gas**: A new resource metric for blob data
- **Blob Gas Limit**: Target of 2 blobs per block
- **Blob Gas Price**: Adjusts based on blob usage
- **Max Blob Constraint**: Maximum of 6 blobs per block (768 KB)

#### KZG Commitments

Blob transactions use KZG (Kate-Zaverucho-Goldberg) polynomial commitments to efficiently verify blobs:
- **Commitment**: A cryptographic digest that commits to the blob
- **Proof**: A verification mechanism with constant size
- **Versioned Hash**: `0x01 | kzg_commitment[1:32]`

#### New Precompile: `POINT_EVALUATION`

Address 0x0A, costs 50,000 gas
- Input: Versioned hash, z coordinate, y coordinate, commitment proof
- Output: 1 if valid, 0 if invalid
- Purpose: Verify blob data availability

#### Data Lifecycle

Blob data has a different lifecycle than regular transaction data:
1. Validator proposes block with blob transactions
2. Blob data transmitted over P2P network
3. Blob data available for ~1-2 weeks
4. Blob data pruned, commitments remain in the blockchain

### EVM Object Format (EOF)

The EVM Object Format (EOF) is a proposal to restructure EVM bytecode for better validation, optimization, and future extensibility.

#### Current Limitations

Current EVM bytecode has several limitations:
- No container format or versioning
- No static validation of jumps
- No explicit code/data separation
- Runtime validation overhead
- Difficulty implementing subroutines

#### EOF Design

EOF introduces a structured container format:

```
MAGIC (EF00)
VERSION (01)
SECTION_HEADER_1
SECTION_1
...
SECTION_HEADER_N
SECTION_N
```

Key sections include:
- **Code Section**: Contains executable bytecode
- **Data Section**: Contains immutable data (constants)
- **Jumptable Section**: Pre-validated jump destinations

#### Key Components

1. **EIP-3540: EOF v1 (Container Format)**
   - Introduces container format with code/data separation
   - Magic prefix `0xEF00` indicates EOF format

2. **EIP-3670: Code Validation**
   - Performs static analysis of code sections
   - Validates all instructions are valid
   - Ensures JUMPDEST instructions are valid destinations

3. **EIP-4200: Static Relative Jumps**
   - Adds RJUMP and RJUMPI instructions
   - Jumps with static offsets known at deploy time
   - Enables better optimization and security

4. **EIP-4750: Functions**
   - Adds CALLF (call function) and RETF (return from function)
   - Provides proper function call semantics
   - Creates a call stack for nested function calls

#### Benefits

EOF provides several advantages:
- **Validation at Deploy Time**: Catching errors earlier
- **More Efficient Execution**: Less runtime validation needed
- **Code Semantics**: Better tooling and analysis
- **Future Extensibility**: Versioning for new features
- **Gas Savings**: Through optimization opportunities

#### Implementation Status

EOF is still in development but is expected in a future hard fork:
- Specifications are finalized for most components
- Client implementations are in progress
- Several EIPs bundled for a complete implementation
- Backward compatible with existing contracts

### Client Implementation of Protocol Upgrades

Implementing protocol upgrades like EIP-4844 and EOF requires careful coordination across client teams.

#### Upgrade Process

1. **Specification Review**
   - EIP authors and core developers discuss and refine the proposal
   - Technical details are specified precisely

2. **Implementation Planning**
   - Client teams determine architecture changes
   - Identify code changes needed across modules

3. **Feature Branches**
   - Development in feature branches with feature flags
   - Coordination between client teams for interoperability

4. **Testing Phase**
   - Unit and integration tests
   - Devnets for cross-client testing
   - Public testnets (e.g., Sepolia, Holesky)

5. **Mainnet Preparation**
   - Shadow forks of mainnet
   - Final spec freeze
   - Mainnet deployment

#### Implementation Patterns

Different clients may use different approaches, but common patterns include:

1. **Fork Configuration**
   - Chain specification with activation blocks/timestamps
   - Fork-specific parameters (e.g., blob gas limits)

2. **Feature Flags**
   - Compile-time or runtime feature toggles
   - Allows testing pre-activation

3. **Versioned Code Paths**
   - Logic that changes based on fork activation
   - Pre-fork and post-fork variants

4. **Extended Data Structures**
   - Protocol changes often require extending core data types
   - Backward compatible extensions

#### RLP Encoding for New Transaction Types

For new transaction types like EIP-4844 blobs, serialization is crucial:

- **Type Prefix**: First byte indicates transaction type
- **RLP Body**: Type-specific fields encoded with RLP
- **Transaction Signature**: Typically includes y_parity, r, s components

### Reth's Implementation of EIP-4844

Reth has implemented EIP-4844 in accordance with the protocol upgrade specifications.

#### Key Components

1. **Transaction Types**
   - Transaction type enums include TYPE_EIP4844 (0x03)
   - Struct definitions for blob transactions
   - Serialization/deserialization implementations

2. **Blob Transaction Processing**
   - Validation rules specific to blob transactions
   - Blob fee calculation logic
   - Blob gas accounting

3. **KZG Cryptography**
   - Integration with trusted setup
   - Commitment verification
   - Proof generation/validation

4. **Transaction Pool Handling**
   - Special handling for blob transactions
   - Blob-specific prioritization
   - Blob count limits

5. **Network Layer Changes**
   - Extended protocols for blob propagation
   - Blob availability maintenance
   - Separate blob gossip

## Exercise: Implementing Blob Transaction Decoding

### Objective

Implement the decoding logic for EIP-4844 blob transactions in Reth, enabling the client to properly parse and validate Type 3 transactions.

### Prerequisites

- Completed Tracks 0-6
- Understanding of RLP encoding/decoding
- Familiarity with transaction formats in Ethereum
- Knowledge of EIP-4844 specification

### Tools and Libraries

- Reth codebase
- Rust RLP libraries (e.g., `alloy-rlp`, `ethereum-types`)
- Testing framework (e.g., Rust's built-in tests, proptest)

### Tasks

1. **Fork and Clone the Repository:**

   ```bash
   git clone https://github.com/yourbuddyconner/evm-zero-to-hero.git
   cd evm-zero-to-hero/modules/reth-patches
   ```

2. **Understand the Existing Transaction Types:**

   Examine Reth's transaction type implementations:
   ```bash
   find . -name "*transaction*" -type f | xargs grep -l "enum.*Transaction"
   ```

   Key files to examine:
   - Transaction type definitions
   - Transaction trait implementations
   - Existing transaction decoding logic

3. **Study the EIP-4844 Specification:**

   Review EIP-4844 to understand the exact blob transaction format:
   ```
   0x03 || rlp([
       chain_id,
       nonce,
       max_priority_fee_per_gas,
       max_fee_per_gas,
       gas_limit,
       to,
       value,
       data,
       access_list,
       max_fee_per_blob_gas,
       blob_versioned_hashes,
       y_parity,
       r,
       s
   ])
   ```

4. **Define Data Structures:**

   Define or extend the necessary structs:

   ```rust
   /// A blob transaction according to EIP-4844
   #[derive(Debug, Clone, PartialEq, Eq)]
   pub struct BlobTransaction {
       /// Common fields for all transaction types
       pub common: TxCommon,
       /// Access list
       pub access_list: AccessList,
       /// Maximum fee per blob gas
       pub max_fee_per_blob_gas: U256,
       /// KZG commitment versioned hashes
       pub blob_versioned_hashes: Vec<H256>,
       /// Transaction signature
       pub signature: Signature,
   }
   ```

5. **Implement RLP Decoding:**

   Create a method to decode the RLP-encoded transaction:

   ```rust
   impl Decodable for BlobTransaction {
       fn decode(buf: &mut &[u8]) -> Result<Self, DecodeError> {
           // Read list length prefix
           let list_len = alloy_rlp::Header::decode(buf)?.payload_length;
           let list_start = buf.len();
           
           // Decode each field in order
           let chain_id = Decodable::decode(buf)?;
           let nonce = Decodable::decode(buf)?;
           let max_priority_fee_per_gas = Decodable::decode(buf)?;
           let max_fee_per_gas = Decodable::decode(buf)?;
           let gas_limit = Decodable::decode(buf)?;
           let to = TransactionKind::decode(buf)?;
           let value = Decodable::decode(buf)?;
           let data = Decodable::decode(buf)?;
           let access_list = Decodable::decode(buf)?;
           
           // Blob transaction specific fields
           let max_fee_per_blob_gas = Decodable::decode(buf)?;
           let blob_versioned_hashes = Decodable::decode(buf)?;
           
           // Signature fields
           let y_parity = Decodable::decode(buf)?;
           let r = Decodable::decode(buf)?;
           let s = Decodable::decode(buf)?;
           
           // Ensure we've consumed exactly the list length
           if list_start - buf.len() != list_len {
               return Err(DecodeError::ListLengthMismatch);
           }
           
           // Create the blob transaction
           Ok(BlobTransaction {
               common: TxCommon {
                   chain_id,
                   nonce,
                   max_priority_fee_per_gas,
                   max_fee_per_gas,
                   gas_limit,
                   to,
                   value,
                   data,
               },
               access_list,
               max_fee_per_blob_gas,
               blob_versioned_hashes,
               signature: Signature {
                   y_parity,
                   r,
                   s,
               },
           })
       }
   }
   ```

6. **Extend Transaction Type Enum:**

   Update the main Transaction enum to include the new blob transaction type:

   ```rust
   /// Transaction variants supported in Ethereum
   #[derive(Debug, Clone, PartialEq, Eq)]
   pub enum Transaction {
       /// Legacy transaction type
       Legacy(LegacyTransaction),
       /// EIP-2930 transaction
       Eip2930(Eip2930Transaction),
       /// EIP-1559 transaction
       Eip1559(Eip1559Transaction),
       /// EIP-4844 blob transaction
       Eip4844(BlobTransaction),
   }
   ```

7. **Implement Transaction Decoding:**

   Extend the transaction decoding logic to handle blob transactions:

   ```rust
   impl Decodable for Transaction {
       fn decode(buf: &mut &[u8]) -> Result<Self, DecodeError> {
           if buf.is_empty() {
               return Err(DecodeError::InputTooShort);
           }
           
           // Check transaction type byte
           match buf[0] {
               0x01 => {
                   // Advance past the type byte
                   *buf = &buf[1..];
                   Ok(Transaction::Eip2930(Decodable::decode(buf)?))
               }
               0x02 => {
                   // Advance past the type byte
                   *buf = &buf[1..];
                   Ok(Transaction::Eip1559(Decodable::decode(buf)?))
               }
               0x03 => {
                   // Advance past the type byte
                   *buf = &buf[1..];
                   Ok(Transaction::Eip4844(Decodable::decode(buf)?))
               }
               _ => {
                   // No type byte means legacy transaction
                   Ok(Transaction::Legacy(Decodable::decode(buf)?))
               }
           }
       }
   }
   ```

8. **Implement Validation Rules:**

   Add validation logic specific to blob transactions:

   ```rust
   impl BlobTransaction {
       /// Validate the blob transaction
       pub fn validate(&self) -> Result<(), ValidationError> {
           // Call common validation
           self.common.validate()?;
           
           // Validate signature
           self.signature.validate()?;
           
           // EIP-4844 specific validations
           
           // Check blob versioned hashes
           if self.blob_versioned_hashes.is_empty() {
               return Err(ValidationError::NoBlobs);
           }
           
           if self.blob_versioned_hashes.len() > MAX_BLOBS_PER_BLOCK {
               return Err(ValidationError::TooManyBlobs);
           }
           
           // Validate each versioned hash has the correct version byte (0x01)
           for hash in &self.blob_versioned_hashes {
               if hash.0[0] != 0x01 {
                   return Err(ValidationError::InvalidBlobVersionedHash);
               }
           }
           
           Ok(())
       }
   }
   ```

9. **Write Comprehensive Tests:**

   Create test cases for blob transaction decoding:

   ```rust
   #[cfg(test)]
   mod tests {
       use super::*;
       
       #[test]
       fn test_decode_valid_blob_transaction() {
           // Example encoded blob transaction
           let encoded_tx = hex::decode("03f8910180808080808080c0850123456789a0010000000000000000000000000000000000000000000000000000000000000001808080").unwrap();
           
           let mut buf = &encoded_tx[..];
           let result = Transaction::decode(&mut buf);
           
           assert!(result.is_ok());
           if let Ok(Transaction::Eip4844(tx)) = result {
               assert_eq!(tx.max_fee_per_blob_gas, U256::from(0x123456789u64));
               assert_eq!(tx.blob_versioned_hashes.len(), 1);
               assert_eq!(tx.blob_versioned_hashes[0].0[0], 0x01);
           } else {
               panic!("Expected blob transaction");
           }
       }
       
       #[test]
       fn test_decode_invalid_blob_transaction() {
           // Invalid transaction (missing fields)
           let encoded_tx = hex::decode("03c0").unwrap();
           
           let mut buf = &encoded_tx[..];
           let result = Transaction::decode(&mut buf);
           
           assert!(result.is_err());
       }
       
       #[test]
       fn test_validation_rules() {
           // Create a valid blob transaction
           let tx = BlobTransaction {
               // ... initialize with valid values
           };
           
           assert!(tx.validate().is_ok());
           
           // Test with too many blobs
           let mut invalid_tx = tx.clone();
           invalid_tx.blob_versioned_hashes = vec![H256::zero(); MAX_BLOBS_PER_BLOCK + 1];
           assert_eq!(invalid_tx.validate(), Err(ValidationError::TooManyBlobs));
           
           // Test with invalid version byte
           let mut invalid_tx = tx.clone();
           let mut wrong_version_hash = H256::zero();
           wrong_version_hash.0[0] = 0x02; // Wrong version byte
           invalid_tx.blob_versioned_hashes = vec![wrong_version_hash];
           assert_eq!(invalid_tx.validate(), Err(ValidationError::InvalidBlobVersionedHash));
       }
   }
   ```

10. **Integration with Transaction Pool:**

    Ensure blob transactions are properly handled by the transaction pool:

    ```rust
    impl TransactionPool {
        // Add a function to check blob-specific constraints
        fn validate_blob_transaction(
            &self, 
            tx: &BlobTransaction,
            base_fee: U256,
            blob_base_fee: U256,
        ) -> Result<(), PoolError> {
            // Check max_fee_per_blob_gas >= blob_base_fee
            if tx.max_fee_per_blob_gas < blob_base_fee {
                return Err(PoolError::BlobFeeTooLow);
            }
            
            // Check if adding this transaction would exceed blob limits
            let blob_count = tx.blob_versioned_hashes.len();
            if self.pending_blob_count + blob_count > self.max_blob_count {
                return Err(PoolError::ExceedsBlobLimit);
            }
            
            // Run normal transaction validation
            self.validate_transaction(&tx.into_transaction(), base_fee)?;
            
            Ok(())
        }
    }
    ```

11. **Test End-to-End:**

    Create an integration test that decodes, validates, and processes a blob transaction:

    ```rust
    #[test]
    fn test_blob_transaction_end_to_end() {
        // Create a test environment
        let env = TestEnvironment::new();
        
        // Craft a blob transaction
        let tx_bytes = create_test_blob_transaction();
        
        // Decode the transaction
        let tx = decode_transaction(&tx_bytes).expect("Failed to decode transaction");
        
        // Validate and add to pool
        env.tx_pool.add_transaction(tx).expect("Failed to add transaction");
        
        // Check it was included in a block
        let block = env.mine_block();
        assert!(block.transactions.iter().any(|t| t.hash() == tx.hash()));
    }
    ```

### Expected Outcome

After completing this exercise, you should have:
- A fully functional blob transaction decoder
- Validation logic for EIP-4844 transactions
- Integration with Reth's transaction pool
- Comprehensive tests for all components

Your implementation should correctly decode and validate blob transactions according to the EIP-4844 specification, enabling Reth to process this new transaction type.

### Verification

Your solution should:
- Pass all unit tests for blob transaction decoding
- Correctly validate transactions according to EIP-4844 rules
- Handle blob transactions in the transaction pool
- Integrate with the existing transaction processing pipeline

### Troubleshooting

Common challenges include:
- **RLP Decoding Errors**: Carefully check field order and types
- **Validation Logic Issues**: Review EIP-4844 for exact constraints
- **Integration Problems**: Ensure all components handle the new transaction type
- **Test Case Coverage**: Create tests for edge cases and error conditions

## Conclusion

You've now explored modern Ethereum protocol upgrades, focusing on EIP-4844 blob transactions and the upcoming EVM Object Format. By implementing blob transaction decoding in Reth, you've gained hands-on experience with the practical aspects of protocol upgrades and their client implementations. This knowledge is invaluable for core Ethereum developers who need to integrate new features while maintaining compatibility and security.

In the next track, we'll shift our focus to performance optimization, exploring profiling, flamegraphs, and implementing instrumentation in Reth.

## Further Reading

- [EIP-4844: Shard Blob Transactions](https://eips.ethereum.org/EIPS/eip-4844)
- [EIP-3540: EVM Object Format v1](https://eips.ethereum.org/EIPS/eip-3540)
- [EIP-3670: EOF - Code Validation](https://eips.ethereum.org/EIPS/eip-3670)
- [EIP-4200: EOF - Static Relative Jumps](https://eips.ethereum.org/EIPS/eip-4200)
- [EIP-4750: EOF - Functions](https://eips.ethereum.org/EIPS/eip-4750)
- [KZG Polynomial Commitments](https://dankradfeist.de/ethereum/2020/06/16/kate-polynomial-commitments.html)
- [Proto-Danksharding FAQ](https://ethereum.org/en/roadmap/danksharding/)
- [RLP Encoding Specification](https://ethereum.org/en/developers/docs/data-structures-and-encoding/rlp/)
- [Reth Blob Transaction Implementation](https://github.com/paradigmxyz/reth/blob/main/crates/primitives/src/transaction/mod.rs) 
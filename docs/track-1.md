# Track 1: Ethereum Fundamentals

## Overview

This track explores the foundational data structures and concepts that power Ethereum. We'll dive deep into Ethereum's account model, the Merkle Patricia Trie (MPT) that underpins state storage, and the economic mechanisms that govern computation on the platform. By understanding these fundamentals, you'll build a solid foundation for the more advanced topics in later tracks.

## Learning Objectives

By the end of this track, you will be able to:
- Distinguish between Externally Owned Accounts (EOAs) and Contract Accounts
- Understand how Ethereum represents and verifies its state through Merkle Patricia Tries
- Explain the purpose and mechanics of the gas system
- Implement and verify Merkle Proofs in Rust

## Core Concepts

### Ethereum Account Model

Ethereum uses an account-based model rather than Bitcoin's UTXO model. This means that the global state consists of a mapping between account addresses and account states.

#### Account Types

There are two types of accounts in Ethereum:

1. **Externally Owned Accounts (EOAs):**
   - Controlled by private keys
   - Cannot contain code
   - Can initiate transactions
   - Identified by a 20-byte address derived from a public key

2. **Contract Accounts:**
   - Controlled by their code
   - Can contain code (smart contracts)
   - Can only perform actions when triggered by an EOA transaction
   - Address derived from creator address and nonce

#### Account State

Both account types share the same state structure with four components:

- **Nonce:** Counter to ensure transactions are processed only once
  - For EOAs: Number of transactions sent from the address
  - For contracts: Number of contracts created by this account
- **Balance:** Amount of Ether (in wei) owned by the account
- **Storage Root:** Hash of the root node of the account's storage trie
- **Code Hash:** Hash of the account's contract code (empty for EOAs)

### State Representation

Ethereum needs an efficient way to store and verify its state. The solution is the Merkle Patricia Trie (MPT).

#### Merkle Patricia Tries (MPTs)

The MPT is a combination of:
- **Merkle Tree:** A tree where each non-leaf node is the hash of its children, enabling efficient verification
- **Patricia Trie:** A trie optimized for storing key-value pairs with shared prefixes

Ethereum uses four main MPTs:
1. **State Trie:** Maps addresses to account data
2. **Storage Trie:** One per contract, maps contract state variables
3. **Transaction Trie:** One per block, stores transactions
4. **Receipts Trie:** One per block, stores transaction receipts

#### MPT Node Types

MPTs consist of three node types:

1. **Leaf Nodes:** Contain key remainder and value
2. **Extension Nodes:** Contain shared prefix path and a reference to another node
3. **Branch Nodes:** 17-element array with references to up to 16 child nodes (one per hex digit) and an optional value

#### RLP Encoding

All nodes in Ethereum's MPTs are serialized using Recursive Length Prefix (RLP) encoding before being stored or hashed. RLP is a space-efficient encoding method for arbitrary nested arrays of binary data.

Basic RLP encoding rules:
- Single bytes in the range [0x00, 0x7f] are encoded as themselves
- Strings 0-55 bytes long: [0x80 + length] followed by the string
- Strings longer than 55 bytes: [0xb7 + length-of-length] followed by the length, followed by the string
- Lists with total payload 0-55 bytes: [0xc0 + length] followed by the concatenation of the RLP encodings of the items
- Lists with total payload > 55 bytes: [0xf7 + length-of-length] followed by the length, followed by the concatenation of the RLP encodings of the items

#### Keccak-256 Hashing

Ethereum uses Keccak-256 (not the standardized SHA3-256) for cryptographic hashing. All references to nodes in the MPT structure are actually references to the Keccak-256 hash of the RLP-encoded node.

### Gas Mechanism

Ethereum's gas system serves multiple critical functions:
- Prevents denial of service attacks by putting a cost on computation
- Provides economic incentive for validators
- Allocates scarce computation resources fairly

#### Gas Concepts

- **Gas:** Unit of computation measurement in Ethereum
- **Gas Price:** Price in ether per unit of gas (set by transaction sender)
- **Gas Limit:** Maximum gas a transaction or block can consume
- **Base Fee:** Algorithmically determined minimum gas price (post EIP-1559)
- **Priority Fee (Tip):** Additional fee to incentivize inclusion (post EIP-1559)

#### Gas Calculation

Each operation in the EVM has a fixed gas cost:
- Simple operations (ADD, SUB): 3-5 gas
- Storage operations (SSTORE): 5,000+ gas (varies by EIP-2200 conditions)
- Contract creation: 32,000 gas
- Transaction base cost: 21,000 gas

Additional gas costs apply for:
- Transaction data (4 gas per zero byte, 16 gas per non-zero byte)
- Memory expansion
- Complex cryptographic operations

#### Gas Refunds

In some cases, gas can be refunded:
- Releasing storage (setting a non-zero value to zero)
- Self-destructing a contract

However, gas refunds are limited to 1/5th of total gas used (post-London fork).

### Merkle Proofs

Merkle proofs allow verification of specific state data without requiring the entire state. They are used in:
- Light clients that don't store the full state
- Layer 2 solutions that need to prove state transitions
- Cross-chain bridges that need to verify Ethereum state

#### Proof Components

A Merkle proof typically consists of:
1. The key-value pair being proven
2. A collection of "sibling" hashes needed to reconstruct the path from the leaf to the root
3. The expected root hash

#### Verification Process

To verify a Merkle proof:
1. Hash the target leaf node
2. Combine with siblings and hash upward, following the path
3. Compare the resulting root hash with the expected root hash
4. If they match, the proof is valid

## Exercise: MPT Proofs with `mpt-play`

### Objective

Build and verify Merkle Patricia Trie proofs using Rust, gaining hands-on experience with the data structure that powers Ethereum's state.

### Prerequisites

- Completed Track 0 setup
- Basic understanding of Rust syntax and data structures
- Conceptual understanding of hash functions and trees

### Tools and Libraries

The `mpt-play` module provides:
- Core MPT implementation
- Functions for constructing tries
- Utilities for generating and verifying proofs
- The `simple_proof` example to demonstrate the workflow

### Tasks

1. **Explore the MPT Implementation:**

   Navigate to the `mpt-play` directory and examine its structure:
   ```bash
   cd modules/mpt-play
   ls -la src/
   ```

   Key files to review:
   - `lib.rs`: Core module definitions
   - `node.rs`: MPT node type implementations
   - `trie.rs`: Trie construction and traversal logic
   - `proof.rs`: Proof generation and verification

2. **Run the Simple Proof Example:**

   ```bash
   cargo run --example simple_proof
   ```
   
   This example:
   - Creates a simple MPT
   - Inserts several key-value pairs
   - Generates a proof for one of the keys
   - Verifies the proof against the trie's root hash

3. **Modify the Example:**

   Create a copy of the simple proof example and modify it to:
   
   ```bash
   cp examples/simple_proof.rs examples/custom_proof.rs
   ```
   
   Edit `examples/custom_proof.rs` to:
   - Insert at least 5 different key-value pairs
   - Generate proofs for multiple keys
   - Verify each proof
   - Try to verify a proof with an incorrect root hash (should fail)
   - Try to verify a proof for a non-existent key

4. **Understand the Proof Generation:**

   Examine the proof generation code in the `proof.rs` file. Notice how it:
   - Traverses the path from root to leaf
   - Collects sibling nodes along the path
   - Encodes the necessary components for verification

5. **Understand the Verification Logic:**

   Study the proof verification function. Pay attention to:
   - How it reconstructs the path to the root
   - How it computes intermediate hashes
   - The comparison with the expected root hash

6. **Challenge: Implement Batch Proof:**

   Extend the `mpt-play` library with a function to generate and verify proofs for multiple keys in a single operation. This might involve:
   - Modifying the `proof.rs` file to add a new function
   - Creating an efficient algorithm that avoids redundant traversals
   - Adding a new example to demonstrate batch proofs

### Expected Outcome

After completing this exercise, you should:
- Understand how MPTs store and organize data
- Know how to construct a trie and insert key-value pairs
- Be able to generate cryptographic proofs for trie contents
- Understand the relationship between MPTs and Ethereum's state

### Verification

Your solution should:
- Successfully compile and run without errors
- Correctly generate and verify proofs for existing keys
- Correctly handle verification failures for invalid proofs
- If you implemented the batch proof challenge, it should efficiently handle multiple proofs

### Troubleshooting

Common issues include:
- **Hash Mismatches:** Often caused by incorrect RLP encoding or node structure
- **Path Traversal Errors:** Check that key encoding is consistent
- **Memory Issues:** Very large tries might require optimizing memory usage

## Conclusion

You've now mastered the foundational concepts of Ethereum's state representation. Understanding accounts, MPTs, and the gas mechanism provides the groundwork for exploring more complex aspects of Ethereum's architecture in future tracks.

In the next track, we'll explore the Ethereum Improvement Proposal (EIP) process and dive into several critical EIPs that have shaped Ethereum's evolution.

## Further Reading

- [Ethereum.org - Accounts](https://ethereum.org/en/developers/docs/accounts/)
- [Ethereum.org - Merkle Patricia Tries](https://ethereum.org/en/developers/docs/data-structures-and-encoding/patricia-merkle-trie/)
- [Ethereum.org - Gas and Fees](https://ethereum.org/en/developers/docs/gas/)
- [Vitalik's Blog - Merkling in Ethereum](https://blog.ethereum.org/2015/11/15/merkling-in-ethereum)
- [RLP Encoding Specification](https://ethereum.org/en/developers/docs/data-structures-and-encoding/rlp/)
- [Yellow Paper - Appendix D (MPT) and Appendix G (Fee Schedule)](https://ethereum.github.io/yellowpaper/paper.pdf) 
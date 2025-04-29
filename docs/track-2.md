# Track 2: EIP Bootcamp

## Overview

Ethereum Improvement Proposals (EIPs) are the official mechanism for proposing, discussing, and implementing changes to the Ethereum protocol. In this track, we'll explore the EIP process, understand the role of Meta EIPs in coordinating network upgrades, and study several pivotal EIPs that have reshaped Ethereum's economics and capabilities. We'll also build a tool to analyze and visualize differences between EIP versions.

## Learning Objectives

By the end of this track, you will be able to:
- Understand the EIP process and various EIP types
- Identify and understand Meta EIPs that coordinate network upgrades
- Analyze critical EIPs like EIP-1559 (fee market), EIP-4844 (blob transactions), and EVM Object Format (EOF)
- Build and use a tool to parse, search, and visualize changes in EIPs

## Core Concepts

### The EIP Process

EIPs are the formal documents that detail proposed changes to Ethereum. Understanding the process is essential for contributing effectively to protocol development.

#### EIP Types

According to EIP-1, there are three main types of EIPs:

1. **Standards Track EIPs:**
   - **Core:** Changes to consensus rules, requiring network upgrade
   - **Networking:** Improvements to p2p protocol, devp2p, discv4
   - **Interface:** Changes to client API/RPC specifications
   - **ERC (Ethereum Request for Comments):** Application-level standards/conventions

2. **Meta EIPs:**
   - Process proposals
   - Community information
   - Fork/upgrade coordination

3. **Informational EIPs:**
   - General Ethereum design issues
   - Guidelines or information for the community
   - Do not propose new features

#### EIP Lifecycle

An EIP goes through several statuses:

1. **Idea:** Initial discussion on forums, Ethereum Magicians
2. **Draft:** Formal proposal created, follows EIP-1 template
3. **Review:** Open for broader community feedback
4. **Last Call:** Final review period, typically 14 days
5. **Final:** Accepted standard, no further changes allowed
6. **Stagnant/Withdrawn/Rejected:** For inactive, removed, or declined proposals

#### EIP Roles

- **EIP Author:** Person(s) writing the EIP
- **EIP Champion:** Advocate who follows the EIP through the process
- **EIP Editor:** Maintains the EIP repository, enforces formatting standards
- **Core Developers:** Evaluate technical aspects of Core EIPs
- **Community:** Provides feedback and use cases

### Network Upgrades and Meta EIPs

Network upgrades (hard forks) are coordinated through Meta EIPs, which specify which Core EIPs will be included in a particular upgrade.

#### Hard Fork Coordination

Traditionally, Ethereum uses a "difficulty bomb" to encourage upgrades by making mining progressively harder. Post-Merge, this has been replaced with a "time bomb" that serves a similar purpose.

Recent major upgrades include:
- **London (August 2021):** Implemented EIP-1559
- **Merge (September 2022):** Shifted from Proof of Work to Proof of Stake
- **Shanghai/Capella (April 2023):** Enabled staking withdrawals
- **Cancun/Deneb (March 2024):** Implemented EIP-4844 (blob transactions)

Each upgrade is coordinated through a Meta EIP that lists all included changes.

#### Upgrade Testing Process

Before activation on mainnet, upgrades follow a testing process:
1. **Specification:** EIPs are finalized and specification written
2. **Implementation:** Client teams implement changes
3. **Testing:** Unit tests, integration tests, fuzzing
4. **Devnet:** Private development networks
5. **Testnet:** Public test networks (Sepolia, Holesky)
6. **Mainnet Shadow Fork:** Testing on fork of mainnet
7. **Mainnet:** Full deployment

### Critical EIPs

#### EIP-1559: Fee Market Change

Implemented in the London hard fork, EIP-1559 transformed Ethereum's transaction fee mechanism:

**Key Changes:**
- **Base Fee:** A network-determined minimum fee that is burned
- **Block Size:** Flexible block size with a target gas limit
- **Fee Mechanism:** 
  - Base fee + priority fee (tip) = total fee
  - Base fee adjusts based on network congestion
  
**Impact:**
- More predictable transaction fees
- Better user experience (less guesswork on fees)
- Deflationary pressure through fee burning

#### EIP-4844: Blob Transactions (Proto-Danksharding)

Implemented in the Cancun/Deneb upgrade, this EIP introduced a new transaction type designed to support layer 2 rollups.

**Key Components:**
- **Blob Transactions:** New transaction type (EIP-2718 TransactionType=3)
- **Blobs:** Large pieces of data (up to 128 KB each)
- **Data Availability:** Ensures data is temporarily available to the network
- **KZG Commitments:** Cryptographic commitments to blob data
- **Separate Fee Market:** Different pricing model for blob data vs. execution

**Benefits:**
- Reduces rollup costs by ~10x
- Improves data availability
- Prepares for full Danksharding in the future

#### EVM Object Format (EOF)

A series of EIPs (3540, 3670, 4200, 4750, etc.) that aim to restructure EVM code for better validation, performance, and extensibility.

**Key Features:**
- **Code Validation:** Better validation at deploy time
- **Container Format:** Organized sections for code and data
- **Versioning:** Allow EVM instruction sets to evolve
- **Jumptable:** Explicit jump destinations for better analysis
- **Subroutines:** Proper function calling semantics

**Status:** In active development; planned for a future upgrade.

### EIP Analysis and Tools

Understanding EIPs requires careful reading and ability to track changes as proposals evolve. Tools can help with this process.

#### EIP Documentation Standards

EIPs follow strict formatting requirements:
- Must be in Markdown format
- Requires specific sections (Abstract, Specification, Rationale, etc.)
- Should include test cases when applicable
- Needs backward compatibility assessment

#### EIP Versioning

EIPs evolve through drafts as feedback is incorporated. Tracking changes between versions helps understand the evolution of a proposal.

## Exercise: Building `eip-grep` - A Colorized EIP Diff Tool

### Objective

Create a command-line tool to parse, search, and visualize changes between different versions of EIPs.

### Prerequisites

- Completed Track 0 and Track 1
- Familiarity with Rust command-line application development
- Basic understanding of Git and diffing algorithms

### Tools and Libraries

- The `eip-grep` module scaffold
- Git repositories
- Rust libraries like `clap`, `colored`, `regex`

### Tasks

1. **Explore the EIP Repository:**

   Clone the official EIP repository to understand its structure:
   ```bash
   git clone https://github.com/ethereum/EIPs.git
   cd EIPs
   ls -la EIPS/
   ```

   Note how EIPs are organized with a consistent format.

2. **Set Up the Project:**

   Navigate to the `eip-grep` directory in our course repository:
   ```bash
   cd modules/eip-grep
   ```

   Examine the skeleton code provided. This will include:
   - Command-line argument parsing
   - Basic file loading capabilities
   - Stub functions for EIP parsing and diffing

3. **Implement EIP Parsing:**

   Complete the EIP parsing functionality:
   - Parse EIP metadata from the frontmatter
   - Extract sections (Abstract, Motivation, Specification, etc.)
   - Identify code blocks and special formatting

4. **Implement Section Search:**

   Add functionality to search for specific sections within an EIP:
   ```
   eip-grep --eip 1559 --section Specification
   ```

   This should display only the "Specification" section of EIP-1559.

5. **Implement EIP Comparison:**

   Add functionality to compare two versions of the same EIP:
   ```
   eip-grep --eip 4844 --compare --version1 <commit1> --version2 <commit2>
   ```

   The output should display a colorized diff of the changes between versions.

6. **Add Colorization:**

   Enhance the output with syntax highlighting:
   - Section headings in bold blue
   - Added content in green
   - Removed content in red
   - Technical terms/values in yellow

7. **Additional Features (Optional):**

   Extend the tool with additional functionality:
   - Filter by EIP status
   - Search for terms across multiple EIPs
   - Generate summaries of changes between versions
   - Visualize dependencies between EIPs

8. **Write Tests:**

   Create comprehensive tests for your implementation:
   - Unit tests for parsing functions
   - Integration tests for command-line functionality
   - Ensure edge cases are handled gracefully

### Expected Outcome

After completing this exercise, you should have:
- A functional `eip-grep` tool that can parse and display EIPs
- The ability to search for specific sections within EIPs
- A colorized diff view for comparing EIP versions
- A deeper understanding of EIP structure and versioning

### Verification

Your solution should:
- Handle various EIP formats correctly
- Produce readable, colorized output
- Run efficiently, even on large EIPs
- Include appropriate error handling for missing files or invalid inputs

### Troubleshooting

Common issues include:
- **Markdown Parsing Complexity:** EIPs may contain nested structures that are challenging to parse
- **Git Integration Challenges:** Retrieving specific versions may require careful Git commands
- **Terminal Compatibility:** Colorization may work differently across various terminals

## Conclusion

You've now gained a comprehensive understanding of the EIP process, the role of Meta EIPs in coordinating upgrades, and the details of several pivotal EIPs. Your `eip-grep` tool will serve as a valuable resource for analyzing future EIPs and tracking their evolution.

In the next track, we'll dive into the Ethereum Virtual Machine (EVM), exploring its instruction set, gas costs, and execution model, and we'll implement a custom opcode in a Rust EVM implementation.

## Further Reading

- [EIP-1: EIP Purpose and Guidelines](https://eips.ethereum.org/EIPS/eip-1)
- [EIP-1559: Fee Market Change](https://eips.ethereum.org/EIPS/eip-1559)
- [EIP-4844: Shard Blob Transactions](https://eips.ethereum.org/EIPS/eip-4844)
- [EIP-3540: EOF - EVM Object Format v1](https://eips.ethereum.org/EIPS/eip-3540)
- [Ethereum Magicians Forum](https://ethereum-magicians.org/)
- [Tim Beiko's Ethereum Upgrades Overview](https://tim.mirror.xyz/CHQtTJb1NDxCK41JpULL-zAJe7YOtw-m4UDw6KDju6c)
- [EIP Editor Handbook](https://eips.ethereum.org/EIPS/eip-5069) 
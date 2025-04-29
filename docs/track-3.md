# Track 3: Opcode + VM

## Overview

The Ethereum Virtual Machine (EVM) is the computational engine that powers Ethereum. This track dives deep into the EVM's architecture, its instruction set (opcodes), gas mechanics, and precompiled contracts. The hands-on component involves modifying `revm`, a Rust implementation of the EVM, to add a custom opcode—providing practical experience with the internals of this crucial component of Ethereum.

## Learning Objectives

By the end of this track, you will be able to:
- Explain the EVM's stack-based architecture and execution model
- Identify and understand the purpose of key EVM opcodes
- Describe how gas costs are calculated for EVM operations
- Navigate and modify the codebase of a production-grade EVM implementation
- Implement a custom opcode in Rust and test its functionality

## Core Concepts

### EVM Architecture

The EVM is a quasi-Turing-complete virtual machine designed for deterministic execution of smart contracts.

#### Stack Machine

The EVM is a stack-based machine, where most operations take their inputs from and push their outputs to a stack:

- **Stack:** Last-in, first-out (LIFO) data structure
- **Stack elements:** 256-bit (32-byte) words
- **Stack limit:** 1024 elements
- **Operations:** Push values to stack, pop values from stack, perform calculations, store results

#### Memory Model

The EVM's memory model consists of:

1. **Stack:** Temporary workspace for calculations (as described above)
2. **Memory:** Volatile byte-addressable linear space that resets between transactions
3. **Storage:** Persistent key-value store that persists between transactions
4. **Calldata:** Read-only area containing transaction input data

#### Program Counter

The EVM executes code sequentially using a program counter (PC):
- Starts at position 0
- Increments by 1 or the opcode's width
- Can jump to arbitrary positions via JUMP and JUMPI opcodes

#### Execution Context

Each execution has a context containing:
- Account address being executed
- Transaction origin (original sender)
- Gas price and available gas
- Call value (ETH amount)
- Block information (number, timestamp, etc.)

### EVM Opcodes

Opcodes are the instructions that the EVM executes. Each opcode is a single byte (0x00-0xFF), though many values are currently unused.

#### Opcode Categories

1. **Stack Operations:**
   - **PUSH1-PUSH32** (0x60-0x7F): Push 1-32 bytes onto stack
   - **POP** (0x50): Remove top item from stack
   - **DUP1-DUP16** (0x80-0x8F): Duplicate stack item
   - **SWAP1-SWAP16** (0x90-0x9F): Swap stack items

2. **Arithmetic Operations:**
   - **ADD** (0x01), **SUB** (0x03), **MUL** (0x02), **DIV** (0x04)
   - **SDIV** (0x05), **MOD** (0x06), **SMOD** (0x07)
   - **ADDMOD** (0x08), **MULMOD** (0x09)
   - **EXP** (0x0A), **SIGNEXTEND** (0x0B)

3. **Comparison & Bitwise Logic:**
   - **LT** (0x10), **GT** (0x11), **SLT** (0x12), **SGT** (0x13), **EQ** (0x14)
   - **AND** (0x16), **OR** (0x17), **XOR** (0x18), **NOT** (0x19)
   - **BYTE** (0x1A), **SHL** (0x1B), **SHR** (0x1C), **SAR** (0x1D)

4. **Environmental Information:**
   - **ADDRESS** (0x30), **BALANCE** (0x31), **ORIGIN** (0x32)
   - **CALLER** (0x33), **CALLVALUE** (0x34), **CALLDATALOAD** (0x35)
   - **CALLDATASIZE** (0x36), **CALLDATACOPY** (0x37)
   - **CODESIZE** (0x38), **CODECOPY** (0x39)
   - **GASPRICE** (0x3A), **EXTCODESIZE** (0x3B), **EXTCODECOPY** (0x3C)
   - **RETURNDATASIZE** (0x3D), **RETURNDATACOPY** (0x3E)
   - **EXTCODEHASH** (0x3F), **BLOCKHASH** (0x40)
   - **COINBASE** (0x41), **TIMESTAMP** (0x42), **NUMBER** (0x43)
   - **DIFFICULTY/PREVRANDAO** (0x44), **GASLIMIT** (0x45), **CHAINID** (0x46), **SELFBALANCE** (0x47), **BASEFEE** (0x48)

5. **Memory Operations:**
   - **MLOAD** (0x51): Load word from memory
   - **MSTORE** (0x52): Save word to memory
   - **MSTORE8** (0x53): Save byte to memory
   - **MSIZE** (0x59): Get size of active memory

6. **Storage Operations:**
   - **SLOAD** (0x54): Load word from storage
   - **SSTORE** (0x55): Save word to storage

7. **Control Flow:**
   - **JUMP** (0x56): Unconditional jump
   - **JUMPI** (0x57): Conditional jump
   - **JUMPDEST** (0x5B): Valid destination for jumps
   - **PC** (0x58): Program counter
   - **STOP** (0x00): Halt execution

8. **Contract Operations:**
   - **CREATE** (0xF0), **CREATE2** (0xF5): Create new contract
   - **CALL** (0xF1), **CALLCODE** (0xF2), **DELEGATECALL** (0xF4), **STATICCALL** (0xFA): Call other contracts
   - **RETURN** (0xF3): Return data from execution
   - **REVERT** (0xFD): Revert and return data
   - **INVALID** (0xFE): Invalid instruction
   - **SELFDESTRUCT** (0xFF): Destroy contract

9. **Logging Operations:**
   - **LOG0-LOG4** (0xA0-0xA4): Emit an event with 0-4 topics

### Gas Costs

Gas is the metering mechanism that prevents resource abuse and compensates validators for computation.

#### Gas Pricing Model

- Each operation has a fixed or dynamic gas cost
- Total transaction gas = sum of all operation costs
- Transaction must include enough gas to cover execution
- Unused gas is refunded, used gas is paid for

#### Gas Cost Factors

Several factors determine an operation's gas cost:

1. **Computation intensity:** More CPU-intensive operations cost more gas
2. **Storage impact:** Operations affecting state storage are expensive
3. **Memory expansion:** Growing memory costs additional gas
4. **I/O operations:** Reading/writing from state is costlier than stack operations

#### Example Gas Costs

- **Basic operations (ADD, SUB):** 3 gas
- **Memory operations (MLOAD, MSTORE):** 3 gas + memory expansion cost
- **Storage operations:**
  - **SLOAD:** 2,100 gas (cold access), 100 gas (warm access)
  - **SSTORE:** 20,000+ gas (depending on previous value)
- **Contract calls (CALL, CREATE):** 700+ gas + memory costs
- **Contract creation (CREATE):** 32,000 gas + code cost (200 gas per byte)
- **Transaction base cost:** 21,000 gas

#### Gas Refunds

Historically, Ethereum provided gas refunds for clearing storage (setting non-zero to zero) and contract self-destruction. However, recent changes have significantly reduced or eliminated refunds.

Current rules (post-London fork):
- No refund for SELFDESTRUCT
- Limited refund for storage clearing, capped at 1/5th of total gas used

### Precompiled Contracts

Precompiled contracts are special contracts at predefined addresses that implement cryptographic operations too complex or gas-intensive to implement efficiently in EVM bytecode.

#### Core Precompiles (Addresses 1-9)

1. **ECRECOVER (0x01):** Recovers the signing address from a signature
2. **SHA256 (0x02):** Computes SHA-256 hash
3. **RIPEMD160 (0x03):** Computes RIPEMD-160 hash
4. **IDENTITY (0x04):** Data copy/identity function
5. **MODEXP (0x05):** Modular exponentiation
6. **ECADD (0x06):** Elliptic curve addition on alt_bn128 curve
7. **ECMUL (0x07):** Elliptic curve multiplication on alt_bn128 curve
8. **ECPAIRING (0x08):** Elliptic curve pairing check on alt_bn128 curve
9. **BLAKE2F (0x09):** BLAKE2 compression function

#### Post-Cancun Precompiles

10. **POINT_EVALUATION (0x0A):** KZG point evaluation verifier for EIP-4844

#### Accessing Precompiles

Precompiled contracts are called using the same CALL opcode used for regular contracts:
```
PUSH n        # push gas amount
PUSH 0        # push value (usually 0)
PUSH inputOffset  # push memory offset of input
PUSH inputSize    # push size of input
PUSH 0        # push memory offset for output
PUSH outputSize   # push size of output area
PUSH precompileAddress  # push address (1-9)
CALL          # call the precompile
```

### EVM Implementations

Multiple EVM implementations exist, each with different focuses:

1. **Geth (Go):** Reference implementation in the Go client
2. **Nethermind (C#):** .NET implementation focused on performance
3. **revm (Rust):** Modular, dependency-free Rust implementation focused on performance
4. **evmone (C++):** C++ implementation focused on speed and modularity
5. **ethereumjs-vm (JavaScript):** JavaScript implementation for testing and educational purposes

**revm** is particularly notable for its:
- Performance-focused design
- Rust memory safety benefits
- Modular architecture
- Use in the Reth client and tools like Foundry

## Exercise: Adding a Custom Opcode to revm

### Objective

Implement a simple custom opcode in revm to gain hands-on experience with EVM internals and Rust implementation patterns.

### Prerequisites

- Completed Tracks 0-2
- Solid understanding of Rust programming
- Basic knowledge of EVM execution model

### Tools and Libraries

- revm GitHub repository
- Cargo and Rust toolchain
- Foundry (optional, for additional testing)

### Tasks

1. **Fork and Clone revm:**

   First, fork the revm repository on GitHub, then clone your fork locally:
   ```bash
   git clone https://github.com/<your-username>/revm.git
   cd revm
   ```

2. **Explore the Codebase:**

   Take time to understand revm's architecture:
   ```bash
   ls -la
   ```

   Key directories and files:
   - `crates/interpreter/src/instructions` - Opcode implementations
   - `crates/interpreter/src/interpreter.rs` - Main execution loop
   - `crates/interpreter/src/opcode.rs` - Opcode definitions

3. **Choose an Unused Opcode:**

   Select an unimplemented opcode value for your custom instruction. Good candidates include:
   - `0xF6` - Unused between DELEGATECALL and STATICCALL
   - `0x0C-0x0F` - Unused in the arithmetic range
   - `0x5A` - Unused between PC and JUMPDEST

4. **Define Your Opcode:**

   For this exercise, we'll implement a simple "ADDSQ" opcode that:
   - Pops two values from the stack
   - Adds them together
   - Squares the result (multiplies by itself)
   - Pushes the final value back onto the stack

5. **Add the Opcode to the Enumerations:**

   Modify `crates/interpreter/src/opcode.rs` to add your new opcode:

   ```rust
   // Add to the Opcode enum
   pub enum Opcode {
       // ... existing opcodes ...
       ADDSQ = 0x0C, // Our new opcode at position 0x0C
       // ... more existing opcodes ...
   }

   // Update the opcode name mapping
   impl Opcode {
       pub const fn as_str(&self) -> &'static str {
           match self {
               // ... existing mappings ...
               Self::ADDSQ => "ADDSQ",
               // ... more existing mappings ...
           }
       }
   }
   ```

6. **Implement the Opcode Logic:**

   Create a function for your opcode in an appropriate file in `crates/interpreter/src/instructions/`:

   ```rust
   pub fn addsq<DB: Database>(interpreter: &mut Interpreter<DB>) -> Result<(), StatusCode> {
       // Pop two values from stack
       pop_top!(interpreter, a, b);
       
       // Add them
       let sum = a.overflowing_add(b.0)?;
       
       // Square the result (multiply by itself)
       let result = sum.overflowing_mul(sum)?;
       
       // Push result back to stack
       push_new!(interpreter, result);
       
       Ok(())
   }
   ```

7. **Add to Instruction Table:**

   Update the instruction table in `crates/interpreter/src/interpreter.rs` to include your opcode:

   ```rust
   // In the function that initializes the instruction table
   instruction_table[0x0C as usize] = Some(InstructionTables {
       gas: gas::constants::VERYLOW, // Use a standard gas cost
       instruction: instructions::addsq,
   });
   ```

8. **Update Gas Costs:**

   Define the gas cost for your opcode in `crates/interpreter/src/gas.rs` if needed, or use an existing constant.

9. **Write Tests:**

   Create unit tests for your opcode in an appropriate test file:

   ```rust
   #[test]
   fn test_addsq() {
       // Setup a test EVM instance
       let mut evm = /* initialize test EVM */;
       
       // Create bytecode with your opcode
       let bytecode = vec![
           0x60, 0x03, // PUSH1 3
           0x60, 0x04, // PUSH1 4
           0x0C,       // ADDSQ (our new opcode)
           0x00        // STOP
       ];
       
       // Execute
       let result = /* run the EVM with bytecode */;
       
       // Check stack has one item with value (3+4)² = 49
       assert_eq!(result.stack.pop().unwrap(), U256::from(49));
   }
   ```

10. **Compile and Run Tests:**

    ```bash
    cargo test -p revm-interpreter
    ```

11. **Create an Integration Example:**

    Create a simple example program that uses your new opcode:

    ```rust
    fn main() {
        // Setup a test EVM instance
        let mut evm = /* initialize EVM */;
        
        // Bytecode with your custom opcode
        let bytecode = vec![
            0x60, 0x05, // PUSH1 5
            0x60, 0x07, // PUSH1 7
            0x0C,       // ADDSQ
            0x00        // STOP
        ];
        
        // Execute and print results
        let result = /* run the EVM with bytecode */;
        println!("Result: {}", result.stack.pop().unwrap());
        // Should print "Result: 144" ((5+7)² = 144)
    }
    ```

### Expected Outcome

After completing this exercise, you should have:
- A modified version of revm that includes your custom ADDSQ opcode
- Unit tests that verify the opcode works correctly
- An example program demonstrating the opcode in action
- A deeper understanding of EVM architecture and instruction execution

### Verification

Your solution should:
- Compile without errors
- Pass all tests, including your new opcode tests
- Correctly handle edge cases like overflow
- Follow revm's existing code style and patterns

### Troubleshooting

Common issues include:
- **Compilation Errors:** Make sure your new code follows Rust patterns used in revm
- **Stack Manipulation Errors:** Check that your opcode correctly handles stack items
- **Gas Calculation Issues:** Ensure proper gas accounting for your operation
- **Overflow Handling:** Make sure arithmetic operations properly handle potential overflows

## Conclusion

You've now gained an in-depth understanding of the EVM's architecture, instruction set, and gas mechanics. By implementing a custom opcode in revm, you've experienced first-hand how the EVM executes instructions and how client implementations represent this process in code.

In the next track, we'll explore the Reth client architecture, focusing on its execution pipeline and storage mechanisms, and trace a transaction's journey from JSON-RPC to state commitment.

## Further Reading

- [Ethereum.org - Ethereum Virtual Machine](https://ethereum.org/en/developers/docs/evm/)
- [EVM Codes - Interactive Reference](https://www.evm.codes/)
- [Ethereum Yellow Paper - Appendix H (Virtual Machine Specification)](https://ethereum.github.io/yellowpaper/paper.pdf)
- [Ethereum Yellow Paper - Appendix G (Fee Schedule)](https://ethereum.github.io/yellowpaper/paper.pdf)
- [revm Documentation](https://docs.rs/revm)
- [Gas Optimization Techniques](https://hackmd.io/@roli/r1WM-CHs3)
- [Precompiled Contracts in Ethereum](https://medium.com/@rbkhmrcr/precompiles-in-ethereum-e5d0dde3c2d7) 
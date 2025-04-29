# EVM Zero → Hero Course Materials

This directory contains the textbook-style materials for the EVM Zero → Hero course, which aims to help you master Ethereum core development using Rust.

## Course Tracks

| Track | Title | Description | Exercise |
|-------|-------|-------------|----------|
| [Track 0](track-0.md) | Orientation | Introduction to Ethereum's architecture and setting up the development environment | Environment setup and verification |
| [Track 1](track-1.md) | Ethereum Fundamentals | Accounts, MPT, gas, and the core data structures | MPT proof generation and verification with `mpt-play` |
| [Track 2](track-2.md) | EIP Bootcamp | Understanding the EIP process and analyzing protocol changes | Building `eip-grep` for colorized EIP diffing |
| [Track 3](track-3.md) | Opcode + VM | Deep dive into the EVM, its instruction set, and gas mechanics | Adding a custom opcode to `revm` |
| [Track 4](track-4.md) | Reth tour I | Understanding Reth's architecture, storage, and execution pipeline | Tracing a transaction end-to-end |
| [Track 5](track-5.md) | Reth tour II | Exploring devp2p networking and sync strategies | Patching peer-scoring logic |
| [Track 6](track-6.md) | Consensus bridge | The Engine API and interaction between EL and CL clients | Building a minimal CL stub |
| [Track 7](track-7.md) | Modern upgrades | Implementation of recent protocol changes (EIP-4844, EOF) | Implementing blob transaction decoding |
| [Track 8](track-8.md) | Performance | Profiling, flamegraphs, and optimization techniques | Adding flamegraph instrumentation to Reth |
| [Track 9](track-9.md) | Security testing | Invariant testing and fuzzing for Ethereum clients | Building a differential fuzzer |
| [Track 10](track-10.md) | Governance | The EIP process, release flow, and community contribution | Drafting a Meta-EIP |

## Additional Resources

- [Syllabus](syllabus.md) - Detailed course syllabus with learning objectives and resources
- [EIP Cheat Sheets](eip-cheatsheets/) - Quick reference guides for important EIPs
- [Fork History](fork-history.md) - Timeline of Ethereum network upgrades
- [Glossary](glossary.md) - Definitions of key terms used throughout the course

## How to Use These Materials

1. Start with Track 0 and work through the tracks sequentially.
2. Read the conceptual material first, then attempt the accompanying exercise.
3. Refer to the "Further Reading" section at the end of each track for deeper exploration.
4. Use the course repository's `modules/` directory for the exercise code.

## Contributing

If you find errors or have suggestions for improvements:

1. Fork the repository
2. Make your changes
3. Submit a pull request with the `docs:` prefix in the title

Please see the main [CONTRIBUTING.md](../CONTRIBUTING.md) for detailed guidelines. 
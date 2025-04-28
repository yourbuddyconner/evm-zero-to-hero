# EVM Zero → Hero 🏆

**Master Ethereum core development in a single Rust-powered sprint (≈ 22 h of study & hacking).**

This repo is the open-source curriculum, code scaffolding, and challenge set for the **"EVM Zero → Hero"** course.  
If you can write idiomatic Rust, you can ship a consensus-safe patch to an Ethereum client by the end.

---

## 🎯  What you'll achieve

| ✅ Skill | Outcome |
|---------|---------|
| **Spec literacy** | Read any EIP or the Yellow Paper without your eyes glazing over. |
| **Code traversal** | Trace a transaction through a modern Rust client (Reth) from JSON-RPC to DB commit. |
| **Protocol hacking** | Prototype new opcodes, blob-tx logic, or pruning strategies in Rust. |
| **DevOps confidence** | Spin up private EL + CL nodes, instrument them, and benchmark forks. |
| **Community chops** | Draft an EIP, survive feedback, and open production-grade PRs. |

---

## 📦  Repo layout

```
.
├── bootstrap/          # OS-agnostic environment scripts
├── modules/            # Per-track Rust crates & labs
│   ├── mpt-play/       # Track 1 proof toy
│   ├── eip-grep/       # Track 2 CLI helper
│   ├── reth-patches/   # Track 4-8 guided forks
│   └── fuzz-differ/    # Track 9 differential fuzzer
├── capstone/           # Blank scaffold for your final project
├── docs/               # Slides, hand-outs, hard-fork "cheat sheets"
└── README.md
```

---

## 🗺️  Course roadmap

| # | Track | Key themes | Lab (all Rust) |
|---|-------|------------|----------------|
| 0 | Orientation | EL vs CL, spec locations | Bootstrap script & sanity check |
| 1 | Ethereum fundamentals | Accounts, MPT, gas | `mpt-play` – craft & verify proofs |
| 2 | **EIP Bootcamp** | Process, fork metas, 1559/4844/EOF | `eip-grep` – colour-diff any EIP |
| 3 | Opcode + VM | Stack machine, gas, precompiles | Add a toy opcode in `revm` |
| 4 | **Reth tour I** | Storage, execution pipeline | Trace a tx end-to-end |
| 5 | Reth tour II | devp2p, Snap-sync | Patch peer-scoring logic & test |
| 6 | Consensus bridge | Engine API, fork-choice | Minimal CL stub driving Reth |
| 7 | Modern upgrades | Cancun blobs, EOF, Verkle prep | Implement blob-tx decoding |
| 8 | Performance | Flamegraphs, pruning | `--flame` flag for Reth |
| 9 | Security testing | Invariants, fuzzing | Differential fuzzer vs evmone |
| 10 | Governance | EIP etiquette, release flow | Draft a meta-EIP |

Full descriptions live in [`docs/syllabus.md`](docs/syllabus.md).  

---

## ⚡ Quick-start

> Tested on macOS 13 & Ubuntu 22.04.

```bash
git clone https://github.com/<you>/evm-zero-to-hero.git
cd evm-zero-to-hero

# 1. Install toolchain & deps (~60 s)
./bootstrap/setup.sh

# 2. Run first sanity test
cargo run -p mpt-play --example simple_proof
```

If everything builds, you're ready for Track 1.

---

## 👩‍💻  Contributing

Pull requests, issue reports, and new labs are welcome!

1. Fork → branch → code.
2. `cargo test --all` must be green and `rustfmt + clippy` clean.
3. Open PR with the `track:` prefix (`track:5 peer-scoring tweak`).

See [`CONTRIBUTING.md`](CONTRIBUTING.md) for full guidelines.

---

## 🗒️  FAQ

| Q | A |
|---|---|
| **Do I need prior Solidity/contract experience?** | No. This course works *below* the contract layer. |
| **Can I use another client instead of Reth?** | All labs are Rust-first, but concepts transfer 1-to-1. You can port patches to Nethermind, Erigon, etc. |

---

## 📜  License

MIT + Apache 2.0 dual-license, same as Rust.

---

### Acknowledgements

Huge thanks to the Reth and Foundry teams, AllCoreDevs, and every EIP author we stand on.  
Inspired by [@portport255 @ ZKSync](https://x.com/portport255/status/1916908567795535926).

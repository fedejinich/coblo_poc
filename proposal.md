# PoC Proposal: Evaluation of the COBLO Block Format Proposed in RSKIP-62

## Introduction

Rootstock faces an inefficiency in block propagation: transactions must be executed sequentially because each one modifies the global state. This creates bottlenecks, increases block propagation time, and raises the probability of orphaned-blocks negatively impacting network security and fairness.

RSKIP-62 proposes a new compressed block format called COBLO (Compressed Block using State Trie Update Batch). It allows nodes to start building and mining the next block before fully executing the previous one. This is achieved by including a summary of state changes along with the block header and a compressed list of transaction identifiers.

This PoC aims to evaluate the technical feasibility and practical benefits of adopting COBLO in a simulated network environment.

---

## Problem

- Transaction execution depends on the state produced by previous transactions.
- Propagating full blocks requires transmitting all transactions and executing them sequentially, which is costly in terms of bandwidth and latency.
- Mining the next block must wait until the parent block is fully executed, introducing delays.

---

## Proposed Solution: COBLO

The COBLO format introduces:

1. A compressed list of transaction identifiers.
2. A summary of state changes caused by those transactions.
3. The ability for miners to use the `changeBatch` to start mining the next block before the current block is fully executed.

---

## PoC Description

### Objective

Demonstrate that it is possible to:

- Propagate blocks faster using COBLO.
- Start mining the next block earlier.
- Preserve state consistency and security.
- Reduce the orphan block rate.

### Architecture

Three types of nodes are simulated:

- Miner A: produces blocks using the COBLO format.
- Full Node B: receives COBLO, validates its header, and reconstructs the state from `compressedTransactionIds` and `changeBatch`.
- Miner C: uses the `changeBatch` to begin mining the next block without waiting to execute all transactions from the parent block.

### Components

- COBLO block

```
CobloBlock
├── CobloBlockHeader
│   ├── BlockHeader
│   ├── UnclesHeader[]
    │   └── UncleHeader
│   └── BitcoinSpvProof
├── CompressedTransactionId[]
│   └── CompressedTransactionId
└── Change[]
    └── Change
        ├── TransactionIndex
        └── TransactionChange[]
            └── TransactionChange
                ├── Account
                ├── InternalChange[]
                │   └── InternalChange
                │       ├── InternalKey
                │       │   ├── Nonce
                │       │   ├── Balance
                │       │   ├── Code
                │       │   └── other
                │       └── Value
                └── StorageChange[]
                    └── StorageChange
                        ├── StorageKey
                        └── Value
```

The entire tree structure should be serialized for the later use on block propagation.

- COBLO block propagation
  - Add a new `NetworkMessage`
  - Block connection
  - PoW validation
  - Check and fetch for missing `compressed_transaction_ids`
  - (HARD?) Recontrsuct transaction trie from compressed ids
  - Missing transactions fetch

- COBLO minning mechanism/client

- `StorageKey` compression algorithm

- Setup a testing network

---

## Evaluation and Metrics

### What will be evaluated

- Block propagation time: Time from block emission to full reception, with and without COBLO.
- Mining start time: Whether mining can begin before the full execution of the parent block.
- Orphan block rate: Impact of COBLO on reducing the number of discarded blocks.
- COBLO compression ratio: Comparison between traditional block size and COBLO block size.
- Local mempool resolution rate: Effectiveness of `compressedTransactionIds` in reconstructing transactions from local state.
- ID collision resolution time: Overhead introduced when compressed transaction IDs collide.
- State validation error rate: Frequency at which final state derived from `changeBatch` does not match expected state.

---

## Security Considerations

- Intentional collisions in `compressedTransactionIds`.
- Inconsistencies between `changeBatch` and actual transaction effects.
- Partial state validation vulnerabilities.
- LEAVE_THIS_TO_DISCUSS_WITH_SERGIO

---

## Conclusion

This PoC aims to validate whether the COBLO format proposed in RSKIP-62 can be securely and efficiently implemented to enhance block propagation and reduce block production delays. The outcome of this PoC can lead to reduce difficulty as proposed by RSKIP-72

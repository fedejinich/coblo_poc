# PoC Proposal: Evaluation of the COBLO Format Proposed in RSKIP-62

## Introduction

Stateful blockchains like Rootstock face a fundamental limitation in block propagation efficiency: transactions must be executed sequentially because each one modifies the global state. This creates bottlenecks, increases block propagation time, and raises the probability of orphaned-blocks negatively impacting network security and fairness.

RSKIP-62 proposes a new compressed block format called **COBLO (Compressed Block using State Trie Update Batch)**. It allows nodes to start building and mining the next block before fully executing the previous one. This is achieved by including a summary of state changes (`changeBatch`) along with the block header and a compressed list of transaction identifiers.

This PoC aims to evaluate the technical feasibility and practical benefits of adopting COBLO in a simulated network environment.

---

## Problem

- Transaction execution depends on the state produced by previous transactions.
- Propagating full blocks requires transmitting all transactions and executing them sequentially, which is costly in terms of bandwidth and latency.
- Mining the next block must wait until the parent block is fully executed, introducing delays.

---

## Proposed Solution: COBLO

The COBLO format introduces:

1. A compressed list of transaction identifiers (`compressedTransactionIds`).
2. A summary of state changes (`changeBatch`) caused by those transactions.
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

- **Miner A**: produces blocks using the COBLO format.
- **Relay B**: receives COBLO, validates its header, and reconstructs the state from `compressedTransactionIds` and `changeBatch`.
- **Miner C**: uses the `changeBatch` to begin mining the next block without waiting to execute all transactions from the parent block.

### Simulated Components

- `blockHeader`: standard block header.
- `compressedTransactionIds`: truncated transaction hashes (e.g., first 10 bytes).
- `changeBatch`: list of key-value pairs representing state trie updates.
- `mempool`: local transaction pool for each node.
- Simulated network with configurable latency.

---

## Evaluation and Metrics

### What will be evaluated

1. **Block propagation time (with and without COBLO)**
   - Time from block emission to full reception.

2. **Time to start mining the child block**
   - Whether mining can begin before the full execution of the parent block.

3. **Compression ratio achieved by COBLO**
   - Comparison between traditional block size and COBLO block size.

4. **Percentage of transactions resolved via local mempool**
   - Effectiveness of `compressedTransactionIds` in reconstructing transactions.

5. **Time to resolve ID collisions**
   - Measure overhead when compressed transaction IDs collide.

6. **State validation error rate**
   - Measure if the final state derived from `changeBatch` matches expectations.

7. **Orphan block rate**
   - Assess whether COBLO helps reduce the number of discarded blocks.

---

## Security Considerations

- Intentional collisions in `compressedTransactionIds`.
- Inconsistencies between `changeBatch` and actual transaction effects.
- Partial state validation vulnerabilities.

---

## Conclusion

This PoC aims to validate whether the COBLO format proposed in RSKIP-62 can be securely and efficiently implemented to enhance block propagation and reduce block production delays. The outcome of this PoC may inform decisions about adopting the protocol in real-world networks such as RSK or other EVM-compatible chains.

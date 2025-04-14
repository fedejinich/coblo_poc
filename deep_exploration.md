# A Deep Exploration of the 'Old' COBLO Block Proposal

In this document/article we will explore the insights of the RSKIP62 which introduces a new block type (COBLO) to improve block propagation times.

## Critical Path During Block Propagation

BRIEF EXPLANATION OF THE CRITICAL PATH OF A BLOCK PROPAGATION

## Two Proposals To Rule Them All

Two proposals:

- even/odd sharding (destructive as fuck)
- headers-first propagation (what we will explore care)

## A Twisted Headers-First Propagation (RSKIP62)

This version allows minners to start minning a child block before verifying the full parent block.

How we do this? Including in the header block a batch of state modifications and forwarding the header BEFORE the full block.

## A Compressed New Block Format (COBLO)

RSKIP62 introduces a new compressed block format `COBLO` defined as:

- `CobloBlock`
  - `CobloBlockHeader block_header`
    - `BlockHeader`
    - `UnclesHeaders` (a list?)
    - `BitcoinSpvProof`
  - `CompressedTransactionId[] compressed_transaction_ids`
    - `CompressedTransactionId`: 10-byte prefix of a 256-bit `TransactionId`
  - `Change[] change_batch`:  list of changes a transaction performs on the state trie
    - `Change`: a tuple of `(TransactionIndex, TransactionChange[])`
      - `TransactionIndex`: index of the transaction in the block that is being described.
      - `TransactionChange[]`: a list of transaction changes.
        - `TransactionChange`: a triplet of `(Account, InternalChange[], StorageChange[])`
          - `Account`: address of the account (or contract) that this transaction is changing (or being created).
          - `InternalChange[]`: list of internal changes.
            - `InternalChange`: a tuple of `(InternalKey, Value)`
              - `InternalKey`: a change type
                - `Nonce`: number 0
                - `Balance`: number 1
                - `Code`: number 2
                - other: number > 2
              - `Value`
          - `StorageChange[]`: a list of storage changes.
            - `StorageChange`: a tuple of `(StorageKey, Value)`
              - `StorageKey`: address of a cell in contract storage.
              - `Value`: value to be stored in that cell.

### Special Cases

#### StorageChange Empty Value

When a `StorgeChange` has an empty value it means to remove that contract storage cell.

#### CPU and Storage Bounded Transactions

If a transaction index is not present in the `changeBatch` list, then that transaction must be executed in order to detect the changes.

This allows the miner to compress the block by characterizing transactions in `CPU-bound` and `Storage-bound`.

`CPU-bound` transactions are transactions that consume a higher amount of gas in computation than in modifying the storage.

`Storage-bound` transactions are the opposite.

The exact threshold is chosen by the miner. The miner will add to the `changeBatch` all CPU-bound transactions.

## Compressing Storage Keys

`StorageKeys` are subject to the key-compression scheme defined as a variant of `run-length` encoding.

When first byte is zero we read normal data and we continue from the second byte.

When first byte is non-zero then the three most significant bits are special and they represent different compression options:

- `000`: ...
- `001`: ...
- `010`: ...
- `011`: ...
- `100`: ...
- `otherwise`: reserved for future use.

## A New Network Message

In this proposal we introduce a new `NetworkMessage` == `COBLO_BLOCK` used to propagate COBLOs.

## COBLOs Propagation

...

### Special Cases

Block headers should be discarded if any of the `StorageKey[]` contains a `StorageKey` that doesn't match to the compression options described in the above section.

If the node that receives the compressed header does not have a copy of a transaction specified by a `CompressedTransactionId`, it doesn’t need to immediately ask for it to a peer. It can forward the compressed block.

## How to Introduce This New Block Type?

IN THIS SECTION WE DESCRIBE HOW THE NEW BLOCK TYPE IS INTRODUCED AND HOW WILL THE ROOTSTOCK PROTOCOL CHANGE.

## Alternatives

POTENTIAL ALTERNATIVE DESIGNS

### Without StorageKey Compression

This removes complexity and prevents over-engnieering leaving space for potential improvements. The downside of this alternative is that the COBLO block sizes will be bigger and this will affect block propagation, making the entire proposal useless.

### ???

SPACE FOR EXTRA ALTERNATIVE

## Components to build a PoC

- COBLO block
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
- Benchmakring

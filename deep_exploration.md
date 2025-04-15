# A Deep Exploration of the 'Old' COBLO Block Proposal

In this document/article we will explore the insights of the RSKIP62 which introduces a new block type (COBLO) to improve block propagation times.

The goal of this document is explore and write a potenital article/s.

## Critical Path During Block Propagation

BRIEF EXPLANATION OF THE CRITICAL PATH OF A BLOCK PROPAGATION

## Two Proposals To Rule Them All

Two proposals:

- even/odd sharding (destructive as fuck)
- headers-first propagation (what we are exploring)

## A Twisted Headers-First Propagation (RSKIP62)

This version allows minners to start minning a child block before verifying the full parent block.

How we do this? Including in the header block a batch of state modifications and forwarding the header BEFORE the full block.

## A Compressed New Block Format (COBLO)

RSKIP62 introduces a new compressed block format `COBLO` defined as:

- `CobloBlock`
  - `CobloBlockHeader block_header`
    - `BlockHeader`
    - `UnclesHeaders[]`
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

`StorageKeys` are subject to the key-compression scheme defined as a variant of run length encoding (RLE), a very simple compression technique where we replace a given symbol with a count of how many times that symbol is repeated plus the symbol itself.

```
PPPPPAAATOOO -> 5P3A1T3O
```

RLE is effective for compressing homogenious data but is less effective when the data size is small (CHECK THIS) and when it has fewer variations, and at some point it can increase the data size because it takes 2 bytes to represent a single symbol.

```
FEDDE -> 1F1E2D1E
```

### The algorithm

For those reasons, in this RSKIP we propose the compression algorithm defined as following (EXPLAIN MORE THIS PART IT NEEDS MORE REASONS):

- When first byte is zero we read normal data and we continue from the second byte.
- When first byte is non-zero then the three most significant bits are special and they represent different compression options:
  - `000`: blob matches `(dummyByte, valueData)` and the result is `valueData`
  - `001`: blob matches `(dataLength, dataOffset)` and the value is taken from the `data` field of the transaction. Each element corresponds to a 16-bit unsigned word (4 bytes)
    - (MAYBE THIS SHOULD BE SOMWHERE ELSE) If the provided values are invalid, then the block header is invalid.
    - This type of compression is useful to specify that a certain contract will store the code that is specified by a contract creation transaction.
  - `010`: the blob matches `(dummyByte, account, storageAddress, valueOffset, valueLength)` and the value is extracted from the state of the contract.
    - Rootstock allows storage values to be larger than 32 bytes, this compression method allows to extract any number of bytes at any offset.
    - This can be used when a contract creates another contract (`CREATE3`)
  - `011`: blob matches `(dummyByte)` and the value extracted is the receiver's address of the transaction.
  - `100`: blob matches `(dataByte)` and the extracted data is `dataByte` (without the most significant bit)
    - Allows encoding values from 0 to 31.
  - `otherwise`: reserved for future use.

ADD_EXAMPLES_FOR_EACH_CASE

## A New Network Message

In this proposal we introduce a new `NetworkMessage` == `COBLO_BLOCK` used to propagate COBLOs.

## COBLOs Propagation

...

### Special Cases

Block headers should be discarded if any of the `StorageKey[]` contains a `StorageKey` that doesn't match to the compression options described in the above section.

If the node that receives the compressed header does not have a copy of a transaction specified by a `CompressedTransactionId`, it doesn’t need to immediately ask for it to a peer. It can forward the compressed block.

## How to Introduce This New Block Type?

IN THIS SECTION WE DESCRIBE HOW THE NEW BLOCK TYPE IS INTRODUCED AND HOW WILL THE ROOTSTOCK PROTOCOL CHANGE.

- is there a any block type on the block header?
- how would the node identify the new block? with the NetworkMessage?

## Alternatives

POTENTIAL ALTERNATIVE DESIGNS

### Without StorageKey Compression

This removes complexity and prevents over engineering leaving space for potential improvements. The downside of this alternative is that the COBLO block sizes will be bigger and this will affect block propagation, making the entire proposal useless.

### ???

SPACE FOR EXTRA ALTERNATIVE

## Components to build a PoC

- `StorageKey` compression algorithm
- COBLO block
  - structure
  - serialization
- COBLO block propagation
  - Add a new `NetworkMessage`
  - Block connection
  - PoW validation
  - Check and fetch for missing `compressed_transaction_ids`
  - (HARD?) Recontrsuct transaction trie from compressed ids
  - Missing transactions fetch
- COBLO minning mechanism/client
- Setup a testing network

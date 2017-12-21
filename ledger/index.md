# 账本

## /common/ledger_interface.go

```go
package ledger
```

```go
// Ledger captures the methods that are common across the 'PeerLedger', 'OrdererLedger', and 'ValidatedLedger'
type Ledger interface {
    // GetBlockchainInfo returns basic info about blockchain
    GetBlockchainInfo() (*common.BlockchainInfo, error)
    // GetBlockByNumber returns block at a given height
    // blockNumber of  math.MaxUint64 will return last block
    GetBlockByNumber(blockNumber uint64) (*common.Block, error)
    // GetBlocksIterator returns an iterator that starts from `startBlockNumber`(inclusive).
    // The iterator is a blocking iterator i.e., it blocks till the next block gets available in the ledger
    // ResultsIterator contains type BlockHolder
    GetBlocksIterator(startBlockNumber uint64) (ResultsIterator, error)
    // Close closes the ledger
    Close()
    // Commit adds a new block
    Commit(block *common.Block) error
}
```

账本接口

## /core/ledger/ledger_interface.go

```go
package ledger
```

```go
// PeerLedger differs from the OrdererLedger in that PeerLedger locally maintain a bitmask
// that tells apart valid transactions from invalid ones
type PeerLedger interface {
    commonledger.Ledger
    // GetTransactionByID retrieves a transaction by id
    GetTransactionByID(txID string) (*peer.ProcessedTransaction, error)
    // GetBlockByHash returns a block given it's hash
    GetBlockByHash(blockHash []byte) (*common.Block, error)
    // GetBlockByTxID returns a block which contains a transaction
    GetBlockByTxID(txID string) (*common.Block, error)
    // GetTxValidationCodeByTxID returns reason code of transaction validation
    GetTxValidationCodeByTxID(txID string) (peer.TxValidationCode, error)
    // NewTxSimulator gives handle to a transaction simulator.
    // A client can obtain more than one 'TxSimulator's for parallel execution.
    // Any snapshoting/synchronization should be performed at the implementation level if required
    NewTxSimulator() (TxSimulator, error)
    // NewQueryExecutor gives handle to a query executor.
    // A client can obtain more than one 'QueryExecutor's for parallel execution.
    // Any synchronization should be performed at the implementation level if required
    NewQueryExecutor() (QueryExecutor, error)
    // NewHistoryQueryExecutor gives handle to a history query executor.
    // A client can obtain more than one 'HistoryQueryExecutor's for parallel execution.
    // Any synchronization should be performed at the implementation level if required
    NewHistoryQueryExecutor() (HistoryQueryExecutor, error)
    //Prune prunes the blocks/transactions that satisfy the given policy
    Prune(policy commonledger.PrunePolicy) error
}
```

peer 账本接口

## /core/ledger/kvledger/kv_ledger.go

```go
package kvledger
```

```go
// KVLedger provides an implementation of `ledger.PeerLedger`.
// This implementation provides a key-value based data model
type kvLedger struct {
    ledgerID   string
    blockStore blkstorage.BlockStore
    txtmgmt    txmgr.TxMgr
    historyDB  historydb.HistoryDB
}
```

实现了`ledger.PeerLedger`接口的 key-value 账本

```go
// NewKVLedger constructs new `KVLedger`
func newKVLedger(ledgerID string, blockStore blkstorage.BlockStore,
    versionedDB statedb.VersionedDB, historyDB historydb.HistoryDB) (*kvLedger, error)
```

根据 state 数据库以及区块存储结构新建 key-value 账本

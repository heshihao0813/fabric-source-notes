# 区块存储

## /common/ledger/blkstorage/blockstorage.go

```go
package blkstorage
```

```go
// BlockStore - an interface for persisting and retrieving blocks
// An implementation of this interface is expected to take an argument
// of type `IndexConfig` which configures the block store on what items should be indexed
type BlockStore interface {
    AddBlock(block *common.Block) error
    GetBlockchainInfo() (*common.BlockchainInfo, error)
    RetrieveBlocks(startNum uint64) (ledger.ResultsIterator, error)
    RetrieveBlockByHash(blockHash []byte) (*common.Block, error)
    RetrieveBlockByNumber(blockNum uint64) (*common.Block, error) // blockNum of  math.MaxUint64 will return last block
    RetrieveTxByID(txID string) (*common.Envelope, error)
    RetrieveTxByBlockNumTranNum(blockNum uint64, tranNum uint64) (*common.Envelope, error)
    RetrieveBlockByTxID(txID string) (*common.Block, error)
    RetrieveTxValidationCodeByTxID(txID string) (peer.TxValidationCode, error)
    Shutdown()
}
```

区块存储接口

## /common/ledger/fsblkstorage/fs_blockstore.go

```go
package fsblkstorage
```

基于文件的区块存储实现

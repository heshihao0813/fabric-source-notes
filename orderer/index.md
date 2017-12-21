# 排序

## /orderer/multichain/chainsupport.go

```go
package multichain
```

```go
// Consenter defines the backing ordering mechanism
type Consenter interface {
    // HandleChain should create and return a reference to a Chain for the given set of resources
    // It will only be invoked for a given chain once per process.  In general, errors will be treated
    // as irrecoverable and cause system shutdown.  See the description of Chain for more details
    // The second argument to HandleChain is a pointer to the metadata stored on the `ORDERER` slot of
    // the last block committed to the ledger of this Chain.  For a new chain, this metadata will be
    // nil, as this field is not set on the genesis block
    HandleChain(support ConsenterSupport, metadata *cb.Metadata) (Chain, error)
}

// ConsenterSupport provides the resources available to a Consenter implementation
type ConsenterSupport interface {
    crypto.LocalSigner
    BlockCutter() blockcutter.Receiver
    SharedConfig() config.Orderer
    CreateNextBlock(messages []*cb.Envelope) *cb.Block
    WriteBlock(block *cb.Block, committers []filter.Committer, encodedMetadataValue []byte) *cb.Block
    ChainID() string // ChainID returns the chain ID this specific consenter instance is associated with
    Height() uint64  // Returns the number of blocks on the chain this specific consenter instance is associated with
}
```

一致性相关接口

# 状态数据库

## /core/ledger/kvledger/txmgnt/statedb/statedb.go

```go
package statedb
```

```go
// VersionedDB lists methods that a db is supposed to implement
type VersionedDB interface {
    // GetState gets the value for given namespace and key. For a chaincode, the namespace corresponds to the chaincodeId
    GetState(namespace string, key string) (*VersionedValue, error)
    // GetStateMultipleKeys gets the values for multiple keys in a single call
    GetStateMultipleKeys(namespace string, keys []string) ([]*VersionedValue, error)
    // GetStateRangeScanIterator returns an iterator that contains all the key-values between given key ranges.
    // startKey is inclusive
    // endKey is exclusive
    // The returned ResultsIterator contains results of type *VersionedKV
    GetStateRangeScanIterator(namespace string, startKey string, endKey string) (ResultsIterator, error)
    // ExecuteQuery executes the given query and returns an iterator that contains results of type *VersionedKV.
    ExecuteQuery(namespace, query string) (ResultsIterator, error)
    // ApplyUpdates applies the batch to the underlying db.
    // height is the height of the highest transaction in the Batch that
    // a state db implementation is expected to ues as a save point
    ApplyUpdates(batch *UpdateBatch, height *version.Height) error
    // GetLatestSavePoint returns the height of the highest transaction upto which
    // the state db is consistent
    GetLatestSavePoint() (*version.Height, error)
    // ValidateKey tests whether the key is supported by the db implementation.
    // For instance, leveldb supports any bytes for the key while the couchdb supports only valid utf-8 string
    ValidateKey(key string) error
    // Open opens the db
    Open() error
    // Close closes the db
    Close()
}
```

数据库接口

## /core/ledger/kvledger/txmgnt/statedb/stateleveldb/stateleveldb.go

```go
package stateleveldb
```

LevelDB 实现

## /core/ledger/kvledger/txmgnt/statedb/statecouchdb/statecouchdb.go

```go
package statecouchdb
```

CouchDB 实现

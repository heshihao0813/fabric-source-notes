# chaincode

chaincode 应用生命周期

## /core/shim/interfaces.go

```go
package shim
```

```go
// Chaincode 接口必须被所有 chaincode 实现
// Fabric 调用这些方法执行交易
type Chaincode interface {
    // Init 在 chaincode 容器创建后执行 Instantiate 交易时被调用
    // 初始化 chaincode 内部数据
    Init(stub ChaincodeStubInterface) pb.Response

    // Invoke 在收到更新或查询账本的交易提案时被调用
    // 交易被 commit 之前不会更新账本
    Invoke(stub ChaincodeStubInterface) pb.Response
}
```

```go
// ChaincodeStubInterface 用于在部署后的 chaincode 应用中读写账本
type ChaincodeStubInterface interface {
    // GetArgs 以 byte slice 数组形式返回调用 chaincode Init 和 Invoke 方法的参数
    GetArgs() [][]byte

    // GetStringArgs 以 string 数组形式返回调用 chaincode Init 和 Invoke 方法的参数
    // 保证客户端传 string 的情况下才调用此方法
    GetStringArgs() []string

    // GetFunctionAndParameters 第一个返回是方法名
    // 第二个返回是 string 数组形式的调用参数
    // 保证客户端传 string 的情况下才调用此方法
    GetFunctionAndParameters() (string, []string)

    // GetArgsSlice 以 byte 数组形式返回调用 chaincode Init 和 Invoke 方法的参数
    GetArgsSlice() ([]byte, error)

    // GetTxID 返回交易提案的 tx_id
    // 参见 protos/common/common.proto 中的 ChannelHeader
    GetTxID() string

    // InvokeChaincode 以交易上下文从本地调用指定 chaincode 的 Invoke 方法
    // 这种调用不会生成新的交易消息
    // 如果被调用 chaincode 属于同一 channel，交易的读写集被相应扩充
    // 如果被调用 chaincode 属于不同 channel，调用 chaincode 能收到的返回只有 Response 对象
    // 被调用 chaincode 中 PutState 方法无法写入账本
    // 也就是说，被调用 chaincode 不会对交易的读写集造成影响
    // 调用不同 channcel 的 chaincode 只有查询作用，而不参与之后 commit 阶段的状态校验
    // 如果 channel 为空，默认为调用者的 channel
    InvokeChaincode(chaincodeName string, args [][]byte, channel string) pb.Response

    // GetState returns the value of the specified `key` from the
    // ledger. Note that GetState doesn't read data from the writeset, which
    // has not been committed to the ledger. In other words, GetState doesn't
    // consider data modified by PutState that has not been committed.
    // If the key does not exist in the state database, (nil, nil) is returned.
    GetState(key string) ([]byte, error)

    // PutState puts the specified `key` and `value` into the transaction's
    // writeset as a data-write proposal. PutState doesn't effect the ledger
    // until the transaction is validated and successfully committed.
    // Simple keys must not be an empty string and must not start with null
    // character (0x00), in order to avoid range query collisions with
    // composite keys, which internally get prefixed with 0x00 as composite
    // key namespace.
    PutState(key string, value []byte) error

    // DelState records the specified `key` to be deleted in the writeset of
    // the transaction proposal. The `key` and its value will be deleted from
    // the ledger when the transaction is validated and successfully committed.
    DelState(key string) error

    // GetStateByRange returns a range iterator over a set of keys in the
    // ledger. The iterator can be used to iterate over all keys
    // between the startKey (inclusive) and endKey (exclusive).
    // The keys are returned by the iterator in lexical order. Note
    // that startKey and endKey can be empty string, which implies unbounded range
    // query on start or end.
    // Call Close() on the returned StateQueryIteratorInterface object when done.
    // The query is re-executed during validation phase to ensure result set
    // has not changed since transaction endorsement (phantom reads detected).
    GetStateByRange(startKey, endKey string) (StateQueryIteratorInterface, error)

    // GetStateByPartialCompositeKey queries the state in the ledger based on
    // a given partial composite key. This function returns an iterator
    // which can be used to iterate over all composite keys whose prefix matches
    // the given partial composite key. The `objectType` and attributes are
    // expected to have only valid utf8 strings and should not contain
    // U+0000 (nil byte) and U+10FFFF (biggest and unallocated code point).
    // See related functions SplitCompositeKey and CreateCompositeKey.
    // Call Close() on the returned StateQueryIteratorInterface object when done.
    // The query is re-executed during validation phase to ensure result set
    // has not changed since transaction endorsement (phantom reads detected).
    GetStateByPartialCompositeKey(objectType string, keys []string) (StateQueryIteratorInterface, error)

    // CreateCompositeKey combines the given `attributes` to form a composite
    // key. The objectType and attributes are expected to have only valid utf8
    // strings and should not contain U+0000 (nil byte) and U+10FFFF
    // (biggest and unallocated code point).
    // The resulting composite key can be used as the key in PutState().
    CreateCompositeKey(objectType string, attributes []string) (string, error)

    // SplitCompositeKey splits the specified key into attributes on which the
    // composite key was formed. Composite keys found during range queries
    // or partial composite key queries can therefore be split into their
    // composite parts.
    SplitCompositeKey(compositeKey string) (string, []string, error)

    // GetQueryResult performs a "rich" query against a state database. It is
    // only supported for state databases that support rich query,
    // e.g.CouchDB. The query string is in the native syntax
    // of the underlying state database. An iterator is returned
    // which can be used to iterate (next) over the query result set.
    // The query is NOT re-executed during validation phase, phantom reads are
    // not detected. That is, other committed transactions may have added,
    // updated, or removed keys that impact the result set, and this would not
    // be detected at validation/commit time.  Applications susceptible to this
    // should therefore not use GetQueryResult as part of transactions that update
    // ledger, and should limit use to read-only chaincode operations.
    GetQueryResult(query string) (StateQueryIteratorInterface, error)

    // GetHistoryForKey returns a history of key values across time.
    // For each historic key update, the historic value and associated
    // transaction id and timestamp are returned. The timestamp is the
    // timestamp provided by the client in the proposal header.
    // GetHistoryForKey requires peer configuration
    // core.ledger.history.enableHistoryDatabase to be true.
    // The query is NOT re-executed during validation phase, phantom reads are
    // not detected. That is, other committed transactions may have updated
    // the key concurrently, impacting the result set, and this would not be
    // detected at validation/commit time. Applications susceptible to this
    // should therefore not use GetHistoryForKey as part of transactions that
    // update ledger, and should limit use to read-only chaincode operations.
    GetHistoryForKey(key string) (HistoryQueryIteratorInterface, error)

    // GetCreator returns `SignatureHeader.Creator` (e.g. an identity)
    // of the `SignedProposal`. This is the identity of the agent (or user)
    // submitting the transaction.
    GetCreator() ([]byte, error)

    // GetTransient returns the `ChaincodeProposalPayload.Transient` field.
    // It is a map that contains data (e.g. cryptographic material)
    // that might be used to implement some form of application-level
    // confidentiality. The contents of this field, as prescribed by
    // `ChaincodeProposalPayload`, are supposed to always
    // be omitted from the transaction and excluded from the ledger.
    GetTransient() (map[string][]byte, error)

    // GetBinding returns the transaction binding
    GetBinding() ([]byte, error)

    // GetSignedProposal returns the SignedProposal object, which contains all
    // data elements part of a transaction proposal.
    GetSignedProposal() (*pb.SignedProposal, error)

    // GetTxTimestamp returns the timestamp when the transaction was created. This
    // is taken from the transaction ChannelHeader, therefore it will indicate the
    // client's timestamp, and will have the same value across all endorsers.
    GetTxTimestamp() (*timestamp.Timestamp, error)

    // SetEvent allows the chaincode to propose an event on the transaction
    // proposal. If the transaction is validated and successfully committed,
    // the event will be delivered to the current event listeners.
    SetEvent(name string, payload []byte) error
}
```

作为参数传给应用 chaincode 回调方法的接口

## /core/shim/chaincode.go

```go
package shim
```

```go
// ChaincodeStub is an object passed to chaincode for shim side handling of
// APIs.
type ChaincodeStub struct {
    TxID           string
    chaincodeEvent *pb.ChaincodeEvent
    args           [][]byte
    handler        *Handler
    signedProposal *pb.SignedProposal
    proposal       *pb.Proposal

    // Additional fields extracted from the signedProposal
    creator   []byte
    transient map[string][]byte
    binding   []byte
}
```

`ChaincodeStubInterface`接口的实现

```go
func (stub *ChaincodeStub) init(handler *Handler, txid string, input *pb.ChaincodeInput, signedProposal *pb.SignedProposal) error
```

初始化 ChaincodeStub 实例

```go
func Start(cc Chaincode) error
```

启动 chaincode

```go
func userChaincodeStreamGetter(name string) (PeerChaincodeStream, error)
```

建立与 peer 的连接 stream

```go
func chatWithPeer(chaincodename string, stream PeerChaincodeStream, cc Chaincode) error
```

启动 stream 通信 goroutine

通过`PeerChaincodeStream.Recv`接口接收消息送到`msgAvail`channel
通过`Handler.handleMessage`处理后监听`Handler.nextState`channel 并返回结果

## /core/shim/handler.go

```go
package shim
```

```go
// Handler handler implementation for shim side of chaincode.
type Handler struct {
    sync.RWMutex
    //shim to peer grpc serializer. User only in serialSend
    serialLock sync.Mutex
    To         string
    ChatStream PeerChaincodeStream
    FSM        *fsm.FSM
    cc         Chaincode
    // Multiple queries (and one transaction) with different txids can be executing in parallel for this chaincode
    // responseChannel is the channel on which responses are communicated by the shim to the chaincodeStub.
    responseChannel map[string]chan pb.ChaincodeMessage
    nextState       chan *nextStateInfo
}
```

处理栈，内含一个 FSM 状态机，管理状态迁移

```go
func (handler *Handler) handleMessage(msg *pb.ChaincodeMessage) error
```

根据`msg.Type`调用 FSM 状态机注册的回调处理

```go
func (handler *Handler) beforeInit(e *fsm.Event)
func (handler *Handler) beforeTransaction(e *fsm.Event)
```

注册在 FSM 状态机上的回调，其中会调用真正的处理函数

```go
func (handler *Handler) handlerInit(msg *pb.ChaincodeMessage)
func (handler *Handler) handleTransaction(msg *pb.ChaincodeMessage)
```

反序列化 protobuf 格式的`pb.ChaincodeMessage.Payload`

调用`Chaincode.Init`/`Chaincode.Invoke`接口并构建返回 protobuf

将返回状态通过`Handler.triggerNextState`函数塞入`Handler.nextState`channel 供`charWithPeer`中启动的 goroutine 做后续处理

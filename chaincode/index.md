# chaincode

chaincode 应用生命周期

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

作为参数传给应用 chaincode 回调的结构，实现了`ChaincodeStubInterface`接口

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

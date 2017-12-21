# 通信

## /core/comm/connection.go

```go
package comm
```

```go
func NewClientConnectionWithAddress(peerAddress string, block bool, tslEnabled bool, creds credentials.TransportCredentials) (*grpc.ClientConn, error)
```

根据地址返回 grpc 连接

## /protos

protobuf 通信结构

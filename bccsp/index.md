# 加密

blockchain cryptographic service provider

加密服务

## /bccsp/bccsp.go

```go
package bccsp
```

加密服务接口

## /bccsp/factory/factory.go

```go
package factory
```

```go
type BCCSPFactory interface {
    // Name returns the name of this factory
    Name() string

    // Get returns an instance of BCCSP using opts.
    Get(opts *FactoryOpts) (bccsp.BCCSP, error)
}
```

获取 BCCSP 实例的工厂接口

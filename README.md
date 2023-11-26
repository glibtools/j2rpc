# j2rpc

go jsonrpc framework

## install

```bash
go get -u github.com/glibtools/j2rpc
```

## usage

```go
package main

import (
    "log"
    "net/http"
    "time"

    "github.com/glibtools/j2rpc"
)

func main() {
    rpc := j2rpc.NewServer(
        j2rpc.WithCallerAfterReadBody(func(bytes []byte) ([]byte, error) {
            log.Printf("read body: %s\n", string(bytes))
            return bytes, nil
        }),
        j2rpc.WithCallerBeforeWrite(func(bytes []byte) ([]byte, error) {
            log.Printf("write body: %s\n", string(bytes))
            return bytes, nil
        }),
    )
    rpc.Use("", loggerHandler())
    rpc.Use("", j2rpc.Recover())
    rpc.RegisterTypeBus(new(apiBus))
    mux := http.NewServeMux()
    mux.Handle("/rpc", rpc)
    log.Fatalln(http.ListenAndServe(":8080", mux))
}

type api struct{}

func (*api) Add(a, b int) int {
    return a + b
}

func (*api) Client(c j2rpc.Context) interface{} {
    return map[string]interface{}{
        "ip":     c.Request().RemoteAddr,
        "header": c.Request().Header,
    }
}

func (*api) Empty() {}

func (*api) Ping() string {
    return "pong"
}

type apiBus struct {
    API *api `j2rpc:"open"`
}

func loggerHandler() j2rpc.Handler {
    return func(ctx j2rpc.Context) {
        t1 := time.Now()
        ctx.Next()
        log.Printf("time cost: %s\n", time.Since(t1))
    }
}

```
[[Packages in Go]]
Every package can have its own tests.

```
order/

    order.go

    service.go

    order_test.go
```

Run:

```
go test ./...
```

Go automatically finds and runs tests for every package in the module.

This package-centric testing model encourages modular, independently testable code.

---
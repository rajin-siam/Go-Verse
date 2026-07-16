[[Packages in Go]]

Unlike some languages that compile file by file, Go compiles **package by package**.

Suppose you have

```
order/
    order.go
    service.go
    repository.go
```

Go first compiles these three files into a single package.

Only after that does another package import it.

This package-based compilation enables build caching and contributes to Go's fast incremental builds.
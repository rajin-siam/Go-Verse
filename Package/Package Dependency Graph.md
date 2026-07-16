[[Packages in Go]]
Every Go project forms a **Directed Acyclic Graph (DAG)**.

For example:

```
main
 ├── api
 │     ├── order
 │     ├── payment
 │     └── auth
 │
 ├── config
 └── logger
```

Notice every dependency points downward.

There are no loops.

Go compiles packages following this graph.

Understanding dependency graphs helps explain why Go builds projects so quickly.
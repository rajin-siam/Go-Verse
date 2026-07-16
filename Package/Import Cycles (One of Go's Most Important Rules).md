[[Packages in Go]]

Go **does not allow circular dependencies** between packages.

Imagine this structure:

```
user
```

```go
import "payment"
```

and

```
payment
```

```go
import "user"
```

Now the compiler gets stuck.

```
user
 └── payment
      └── user
           └── payment
```

Who should be compiled first?

Instead of trying to solve this, Go simply forbids it.

```
import cycle not allowed
```

Learning how to design packages to avoid import cycles is one of the biggest architecture skills in Go.

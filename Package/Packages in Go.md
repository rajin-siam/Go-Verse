# Understanding Packages in Go

Packages are one of the most important concepts in Go. They are the primary way Go organizes code, encourages modular design, and prevents naming conflicts. If you understand packages well, you'll find it much easier to read large Go projects, understand the standard library, and design your own applications.

---

# What is a Package?

A **package** is simply a collection of Go source files that belong together and provide related functionality.

Think of a package as a folder that groups code with a common purpose.

For example:

```
math/
    sqrt.go
    pow.go
    trig.go
```

Every file begins with:

```go
package math
```

All these files become part of the same package.

Instead of putting thousands of functions into one huge file, Go allows you to divide them into logical packages.

Go code is organized into packages, which are similar to libraries or modules in other languages. A package consists of one or more .go source files in a single directory that define what the package does.


---

# Why Packages Exist

Imagine writing an application without packages.

Everything would exist in one giant file.

```
CreateUser()
DeleteUser()
CalculateSalary()
ConnectDatabase()
SendEmail()
EncryptPassword()
GenerateInvoice()
```

After a few months, your project could contain thousands of functions, making it nearly impossible to navigate.

Packages solve this by grouping related functionality.

---

# Package Declaration

Every Go file starts with a package declaration.

```go
package main
```

or

```go
package math
```

This tells the compiler which package the file belongs to.

For example:

```go
package user

func CreateUser() {}

func DeleteUser() {}
```

Another file:

```go
package user

func UpdateUser() {}
```

Although these are separate files, Go treats them as one package.

---

# All Files in a Folder Form One Package

Suppose you have:

```
calculator/

    add.go
    subtract.go
    multiply.go
```

**add.go**

```go
package calculator

func Add(a, b int) int {
    return a + b
}
```

**subtract.go**

```go
package calculator

func Subtract(a, b int) int {
    return a - b
}
```

The compiler combines these files into one package.

Inside the package, every file can access the exported and unexported identifiers of the package (subject to normal visibility rules). It almost feels as if all the files were merged into one before compilation.

---

# Packages Create Namespaces

One important job of packages is preventing naming collisions.

Imagine two different packages.

```
user
```

```go
func Save() {}
```

and

```
product
```

```go
func Save() {}
```

Without packages, Go wouldn't know which `Save()` you mean.

Instead, you write:

```go
user.Save()

product.Save()
```

The package name acts as a namespace.

---

# Importing Packages

To use another package, you import it.

```go
import "fmt"
```

Now you can write

```go
fmt.Println("Hello")
```

`Println` belongs to the `fmt` package.

The compiler knows exactly where to find it.

---

# Standard Library Packages

Go comes with many built-in packages.

Some common examples are:

|Package|Purpose|
|---|---|
|`fmt`|Printing|
|`os`|Operating system interaction|
|`net/http`|HTTP server and client|
|`strings`|String manipulation|
|`strconv`|String conversions|
|`math`|Mathematical functions|
|`time`|Working with dates and times|
|`sync`|Concurrency primitives|
|`context`|Request cancellation and deadlines|
|`encoding/json`|JSON encoding and decoding|

---
# The `main` Package

Every executable Go program starts from the `main` package.

```go
package main

func main() {
    fmt.Println("Hello")
}
```

Only the package named `main` can produce an executable program.

Other packages are libraries.

---

# Package Name vs Module Name

Beginners often confuse packages with modules.

Suppose your project is

```
github.com/siam/coffeeshop
```

This is the **module**.

Inside it you have

```
github.com/siam/coffeeshop/internal/order

github.com/siam/coffeeshop/internal/payment

github.com/siam/coffeeshop/internal/menu
```

Each directory is its own package.

```
order
payment
menu
```

A module is a collection of packages.

---

# Package Import Path

Suppose your project is

```
coffeeshop/

    go.mod

    internal/

        payment/

            payment.go
```

`go.mod`

```go
module github.com/siam/coffeeshop
```

`payment.go`

```go
package payment

func Pay() {}
```

Another package imports it using the full import path:

```go
import "github.com/siam/coffeeshop/internal/payment"
```

Then:

```go
payment.Pay()
```

The import path tells Go where the package lives, while the package name is how you refer to it in code.

---



---

# Package Initialization

A package can contain an `init()` function.

```go
package config

func init() {
    fmt.Println("Loading configuration...")
}
```

When the package is imported, Go automatically runs `init()` once before any other code in that package is used.

Example:

```go
import "myapp/config"
```

Output:

```
Loading configuration...
```

Then `main()` starts.

If multiple packages are imported, Go initializes them in dependency order before calling `main()`.

---

# Import Aliases(Shortcut for package name)

Sometimes two packages have the same name.

```go
import (
    stdjson "encoding/json"
    myjson "github.com/company/json"
)
```

Now:

```go
stdjson.Marshal(v)

myjson.Encode(v)
```

Aliases can also shorten long package names.


---

# Package Design Philosophy

A good package should have a single, clear responsibility.

Instead of one large package that handles users, orders, payments, emails, logging, and notifications, it's better to separate concerns:

```
user/
order/
payment/
email/
inventory/
notification/
```

Each package focuses on one area of the application. This makes the code easier to understand, test, and maintain.

---


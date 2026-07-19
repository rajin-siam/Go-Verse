This is an excellent question because it takes us from **Go language** into **compiler** and **runtime** internals.

The short answer is:

> **String constants are not allocated like normal variables.** They are stored in the program's read-only data section, and Go string values simply point to that memory.

Let's go deeper.

---

# Step 1: What is a Go String?

A Go string is **not the characters themselves**.

A string is a small structure (header) that contains:

- a pointer to the first byte
    
- the length
    

Conceptually:

```go
type string struct {
    Data *byte
    Len  int
}
```

> This is a simplified view. The actual runtime uses an internal representation, but the idea is the same.

So when you write:

```go
var s = "hello"
```

Memory looks like:

```
String Header

+-----------------+
| Pointer ------+ |
| Length = 5     |
+-----------------+
                 |
                 ▼
        Read-only Memory

+-----------------------+
| h | e | l | l | o |
+-----------------------+
```

The header is stored with the variable.

The characters are stored elsewhere.

---

# Step 2: What About String Constants?

Suppose

```go
const StatusPending = "pending"
```

Many beginners imagine:

```
Stack

StatusPending

↓

"pending"
```

This is **not** what happens.

The compiler sees

```go
const StatusPending = "pending"
```

and places the bytes

```
pending
```

into the executable's **read-only data section** (often called `.rodata`).

Conceptually:

```
Executable

.text
.data
.rodata
```

Inside `.rodata`

```
Address

0x5000

↓

p e n d i n g
```

Since it's read-only, your program cannot modify it.

---

# Step 3: What Happens When You Use the Constant?

Suppose

```go
const StatusPending = "pending"

status := StatusPending
```

The compiler effectively turns this into:

```
status

↓

String Header

↓

Pointer

↓

"pending"
```

Both point to the **same bytes**.

```
             +----------------+
status ----> | ptr | len = 7 |
             +----------------+
                    |
                    ▼

        Read-only Memory

+---------------------------+
| p | e | n | d | i | n | g |
+---------------------------+
```

No copy of `"pending"` is made.

---

# Step 4: Multiple Variables

```go
const A = "pending"

x := A
y := A
z := A
```

Does Go create

```
pending

pending

pending
```

No.

Memory:

```
x

↓

+-------+
| ptr---+
+-------+
        |
y       |
↓       |
+-------+
| ptr---+
+-------+
        |
z       |
↓       |
+-------+
| ptr---+
+-------+
        |
        ▼

Read-only Memory

pending
```

One string.

Many pointers.

---

# Step 5: Why Is This Safe?

Strings are immutable.

You cannot write

```go
StatusPending[0] = 'P'
```

Compiler error.

Because strings never change, every variable can safely point to the same bytes.

This saves memory.

---

# Step 6: What Happens With Variables?

```go
var s = "pending"
```

Even though it's a variable,

```
pending
```

still lives in read-only memory.

Only the **string header** is stored with the variable.

```
Variable

+----------------+
| Pointer ------+
| Length = 7    |
+----------------+
                |
                ▼

Read-only Memory

pending
```

---

# Step 7: Concatenation

Example

```go
const A = "Hello"

const B = "World"

const C = A + " " + B
```

The compiler performs **constant folding**.

Instead of generating code to concatenate at runtime, it computes the result during compilation.

The executable contains:

```
Hello World
```

There is no runtime concatenation.

---

# Step 8: Runtime Strings

Now compare:

```go
first := "Hello"

second := "World"

third := first + second
```

Here the compiler cannot replace the operation with a single constant because it depends on runtime values (even if they currently hold literals).

At runtime:

1. Allocate memory for the new string.
    
2. Copy `"Hello"`.
    
3. Copy `"World"`.
    

Result:

```
Heap

HelloWorld
```

Unlike constants, this produces a new string value.

> **Note:** The Go compiler can optimize some concatenations of known literals, but in the general case, concatenating variables creates a new string.

---

# Step 9: Why String Enums Are Efficient

Suppose

```go
const (
	StatusPending OrderStatus = "pending"
	StatusReady   OrderStatus = "ready"
)
```

Every order

```go
order.Status = StatusPending
```

does **not** duplicate the bytes `"pending"`.

It stores only a string header pointing to the shared read-only bytes.

```
Order

↓

Header

↓

Pointer

↓

pending
```

Millions of orders can reference the same literal without storing millions of copies of `"pending"`.

---

# Compiler Optimization

The compiler is smart enough to reuse identical string literals.

```go
const A = "pending"
const B = "pending"
```

Conceptually:

```
A ----+

      |

B ----+

      |

      ▼

Read-only Memory

pending
```

One copy.

Multiple references.

---

# Memory Diagram

```
Source Code

const StatusPending = "pending"

           │
           ▼

Compiler

           │
           ▼

Executable

.text

.data

.rodata
    │
    ▼
+-----------------------------+
| p | e | n | d | i | n | g |
+-----------------------------+

           ▲
           │
     String Header
           ▲
           │
   status variable
```

---

# Key Takeaways

- A Go string is a **header** (pointer + length), not the character data itself.
    
- String literals and string constants are typically stored once in the executable's **read-only data section**.
    
- Multiple variables or constants with the same literal usually share the same underlying bytes.
    
- String constants are immutable, making this sharing safe.
    
- Runtime string operations (such as many concatenations) create new string values rather than modifying existing ones.
    
- This is one reason **string enums are memory-efficient**: each enum value usually references shared immutable string data rather than duplicating the characters.

[[Constants]]

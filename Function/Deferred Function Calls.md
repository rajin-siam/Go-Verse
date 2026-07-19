
## What is `defer`?

The `defer` keyword schedules a function call to run **just before the surrounding function returns**.

Instead of executing immediately, the function call is postponed.

Example:

```go
func main() {
	defer fmt.Println("World")

	fmt.Println("Hello")
}
```

Output

```
Hello
World
```

Even though `defer` appears first, it executes last.

---

# Why Do We Need `defer`?

Imagine opening a file.

```go
file, err := os.Open("data.txt")
if err != nil {
	return
}

// Read the file

file.Close()
```

This works.

But what if the function has multiple return statements?

```go
func Read() error {

	file, err := os.Open("data.txt")
	if err != nil {
		return err
	}

	if somethingWrong {
		file.Close()
		return errors.New("failed")
	}

	file.Close()

	return nil
}
```

You must remember to close the file everywhere.

Using `defer`:

```go
func Read() error {

	file, err := os.Open("data.txt")
	if err != nil {
		return err
	}

	defer file.Close()

	if somethingWrong {
		return errors.New("failed")
	}

	return nil
}
```

Now the file is always closed.

---

# Real World Analogy

Imagine borrowing a library book.

```
Borrow Book

↓

Read

↓

Return Book
```

Instead of remembering later,

you immediately make a note:

```
Before leaving,

↓

Return the book.
```

That's exactly what `defer` does.

---

# Basic Syntax

```go
defer functionCall()
```

Example

```go
defer fmt.Println("Done")
```

---

# When Does `defer` Execute?

It runs **when the surrounding function is about to return**.

Example

```go
func Demo() {

	defer fmt.Println("Second")

	fmt.Println("First")
}
```

Output

```
First

Second
```

---

# Multiple Deferred Calls

```go
func main() {

	defer fmt.Println("A")

	defer fmt.Println("B")

	defer fmt.Println("C")
}
```

Output

```
C

B

A
```

---

# Why?

Deferred calls use a **stack**.

```
defer A

↓

defer B

↓

defer C
```

Execution

```
Pop

↓

C

↓

B

↓

A
```

This is called **LIFO (Last In, First Out)**.

---

# Arguments Are Evaluated Immediately

This is one of the most important rules.

Example

```go
func main() {

	x := 10

	defer fmt.Println(x)

	x = 20
}
```

Output

```
10
```

Why?

When Go sees

```go
defer fmt.Println(x)
```

it immediately evaluates the argument.

Conceptually

```
x

↓

10

↓

Store Argument
```

Later,

```
Run Println(10)
```

Even though `x` became `20`.

---

# Deferred Closures Behave Differently

Example

```go
func main() {

	x := 10

	defer func() {
		fmt.Println(x)
	}()

	x = 20
}
```

Output

```
20
```

Why?

The closure captures the **variable**, not its current value.

When the deferred function finally runs,

```
x

↓

20
```

This difference is a common interview question.

---

# Defer with Return

```go
func Test() int {

	defer fmt.Println("Done")

	return 10
}
```

Execution

```
return 10

↓

Run Deferred Functions

↓

Function Returns
```

The deferred function runs **after the return value has been determined but before the function actually exits**.

---

# Named Return Values

Example

```go
func Test() (result int) {

	defer func() {
		result++
	}()

	return 10
}
```

Output

```
11
```

Why?

Execution order:

```
result = 10

↓

Run Deferred Function

↓

result++

↓

Return 11
```

This is one reason named return values and `defer` are closely related.

---

# Deferred Methods

```go
file, _ := os.Open("a.txt")

defer file.Close()
```

Very common.

Also:

```go
mutex.Lock()

defer mutex.Unlock()
```

The resource is guaranteed to be released before the function exits.

---

# Defer in Loops

Avoid this:

```go
for _, file := range files {

	f, _ := os.Open(file)

	defer f.Close()
}
```

Why?

None of the files close until the surrounding function returns.

If there are many files, you may keep them all open at once.

Better:

```go
for _, file := range files {

	func() {

		f, _ := os.Open(file)

		defer f.Close()

		// Use file

	}()
}
```

Each anonymous function has its own deferred call.

---

# Panic and Defer

When a panic occurs,

Go still runs deferred functions while unwinding the stack.

```go
func main() {

	defer fmt.Println("Cleaning")

	panic("Boom")
}
```

Output

```
Cleaning

panic: Boom
```

This makes `defer` ideal for cleanup.

---

# Recover Depends on Defer

`recover()` only works inside a deferred function.

```go
func main() {

	defer func() {

		if r := recover(); r != nil {

			fmt.Println("Recovered:", r)
		}

	}()

	panic("Boom")
}
```

Without `defer`, `recover()` cannot stop a panic.

---

# Compiler Internals

Conceptually,

```go
defer fmt.Println("Done")
```

becomes

```
Register Deferred Call

↓

Continue Execution

↓

Return

↓

Execute Deferred Calls

↓

Exit Function
```

The runtime keeps track of deferred calls and executes them in reverse order.

Modern Go compilers can optimize many simple `defer` statements by avoiding some runtime overhead, but the behavior remains the same.

---

# Memory Model

Every deferred call stores enough information to execute later.

Conceptually:

```
Deferred Stack

↓

Function

Arguments

Receiver (if any)
```

As deferred functions execute,

entries are removed from the stack.

---

# Common Uses

## Closing Files

```go
file, _ := os.Open("a.txt")

defer file.Close()
```

---

## Unlocking Mutexes

```go
mu.Lock()

defer mu.Unlock()
```

---

## Closing Database Connections

```go
tx, _ := db.Begin()

defer tx.Rollback()
```

If everything succeeds:

```go
tx.Commit()
```

If an error occurs before the commit, the deferred rollback cleans up automatically.

---

## Logging

```go
func Work() {

	defer fmt.Println("Finished")

	// Work
}
```

---

## Measuring Execution Time

```go
func Work() {

	start := time.Now()

	defer func() {
		fmt.Println(time.Since(start))
	}()

	// Work
}
```

---

# Common Mistakes

## Expecting Arguments to Update

```go
x := 10

defer fmt.Println(x)

x = 20
```

Prints

```
10
```

Arguments are evaluated immediately.

---

## Deferring Inside Large Loops

Thousands of deferred calls accumulate until the function returns.

This may consume unnecessary resources.

---

## Forgetting LIFO Order

Many beginners expect

```
A

B

C
```

Actually

```
C

B

A
```

---

## Ignoring Errors from Deferred Calls

Some deferred functions return errors.

Example:

```go
defer file.Close()
```

`Close()` may itself return an error.

Whether to check it depends on the context.

---

# Best Practices

✅ Use `defer` immediately after acquiring a resource.

---

✅ Use `defer` for cleanup logic.

---

✅ Keep deferred functions small.

---

✅ Avoid excessive use inside long-running loops.

---

✅ Understand the difference between deferred arguments and deferred closures.

---

# Interview Questions

### When does a deferred function execute?

Immediately before the surrounding function returns, including during panic unwinding.

---

### In what order are deferred functions executed?

Last In, First Out (LIFO).

---

### When are deferred function arguments evaluated?

Immediately when the `defer` statement is encountered.

---

### Why does this print `10`?

```go
x := 10

defer fmt.Println(x)

x = 20
```

Because `x` is evaluated when `defer` is executed.

---

### Why does this print `20`?

```go
x := 10

defer func() {
	fmt.Println(x)
}()

x = 20
```

Because the closure captures the variable, which is read when the deferred function executes.

---

### Why is `defer` important?

It guarantees cleanup regardless of how a function exits—normal return, early return, or panic.

---

# Related Notes

- [[Functions]]
- [[Named Return Values]]
- [[Closures]]
- [[Panic]]
- [[Recover]]
- [[Error Handling]]
- [[Memory Model]]
- [[Compiler Internals of Functions]]

---

# Summary

- `defer` schedules a function call to execute just before the surrounding function returns.
- Deferred calls execute in **Last In, First Out (LIFO)** order.
- Function arguments in a deferred call are evaluated immediately.
- Deferred closures capture variables and observe their values when they execute.
- `defer` is commonly used for cleaning up resources such as files, mutexes, and database transactions.
- Deferred functions run even if the function exits because of a panic.
- `recover()` only works when called from a deferred function.
- Modern Go compilers optimize many `defer` statements while preserving the language's semantics.
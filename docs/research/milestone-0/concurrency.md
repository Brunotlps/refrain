# Concurrency

Concurrency in programming is the design and structure of a system to handle multiple tasks that make progress over overlapping time periods, rather than strictly one after another. It is about managing and dealing with many things at once, often by rapidly switching back and forth between different tasks. 

**Concurrency vs. Parallelism**

- Concurrency is about structure: It means a program is broken into independent parts that can run during overlapping time frames. On a single processor core, the system rapidly switches between tasks (context switching) to give the illusion of doing things at once

- Parallelism is about execution: It means multiple tasks run at the exact same physical instant. This requires multi-core hardware where each core handles a distinct task simultaneously. Concurrency provides the software structure that can enable parallelism on multi-core chips

**Why We Use Concurrency?**

- Better performance: Keeps the computer busy when a task is waiting for slow actions like network requests or disk reads (I/O-bound tasks).
- Responsive apps: Stops heavy background calculations from freezing user interfaces.
- Resource sharing: Allows multiple independent users or system components to access shared resources safely

**Common Challenges**

- Race conditions: When two parts of a program try to change the same data at the exact same time, causing unexpected bugs.
- Deadlocks: When two or more tasks are stuck waiting for each other to finish, freezing the program completely.

---

**Do not communicate by sharing memory; instead, share memory by communicating.**

Go's approach to concurrent programming is based on shared variables transmitted over channels and, in fact, never actively shared by separate threads of execution.

Although Go's approach to concurrency originates in Hoare's Communicating Sequential Processes (CSP), it can also be seen as a type-safe generalization of Unix pipes.

---

## Goroutines

A goroutine has a simple model: it is a function executing concurrently with other goroutines in the same address space. It is lightweight, costing little more than the allocation of stack space. And the stacks start small, so they are cheap, and grow by allocating (and freeing) heap storage as required.

Prefix a function or method call with the go keyword to run the call in a new goroutine. When the call completes, the goroutine exits, silently. (The effect is similar to the Unix shell's & notation for running a command in the background.)

```go
go list.Sort()  // run list.Sort concurrently; don't wait for it.
```

A function literal can be handy in a goroutine invocation.

```go
func Announce(message string, delay time.Duration) {
    go func() {
        time.Sleep(delay)
        fmt.Println(message)
    }()  // Note the parentheses - must call the function.
}
```

In Go, function literals are closures: the implementation makes sure the variables referred to by the function survive as long as they are active.

These examples aren't too practical because the functions have no way of signaling completion. For that, we need channels.

---

## Channels

Go channels are the built-in pipelines that connect concurrent goroutines, allowing them to communicate and synchronize safely without explicit locks.

**Core Syntax**

Channels are strongly typed. You define them using the chan keyword and initialize them using the make() function. Data flows in the direction of the arrow operator (<-).

```go
// Create a channel that transfers integers
ch := make(chan int) 

// Send data into the channel
ch <- 42             

// Receive data from the channel
value := <-ch      
```

Channels operate in two primary modes depending on how you initialize them:

| **Feature** | **Unbuffered Channels** | **Buffered Channels** |
| --- | --- | --- |
| **Initialization** | `make(chan int)` | `make(chan int, 3)` (capacity specified) |
| **Behavior** | **Synchronous.** Blocks until both sender and receiver are ready. | **Asynchronous.** Blocks only when full (sends) or empty (receives). |
| **Use Case** | Strict synchronization and direct handoffs. | Decoupling execution speeds or batch processing. |

**Channel Lifecycle & Patterns**

1. Range Loop (Iterating over a Channel)

You can loop over a channel using range. The loop automatically pulls values out of the channel until it is closed

```go
func main() {
    ch := make(chan int, 3)
    ch <- 1
    ch <- 2
    close(ch) // Explicitly close to stop the range loop

    for val := range ch {
        fmt.Println(val)
    }
}
```

2. Checking if a Channel is Open

Receiving from a channel can return an optional second boolean variable indicating whether the channel is still open.

```go
val, open := <-ch
if !open {
    // The channel was closed and its buffer is empty
}
```

3. Directional Constraints

You can restrict a channel's access permissions in function signatures to make code safer.
Send-only channel: chan<- intReceive-only channel: <-chan int

- Send-only channel: chan<- int
- Receive-only channel: <-chan int

**Critical State Matrix**

To avoid app crashes (panic) or frozen execution (deadlock), keep these behaviors in mind:
- Nil Channel (uninitialized): Both sending and receiving block forever. Closing it triggers a panic.
- Open Channel: Standard sending/receiving behaviors apply.
- Closed Channel: Sending triggers a panic. Receiving yields the type's default "zero-value" safely without blocking. Closing it a second time triggers a panic.

**The select Statement**

When working with multiple channels, Go provides the select statement. It allows a goroutine to wait on multiple communication operations simultaneously, executing whichever one becomes ready first.

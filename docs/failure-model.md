# Failure Model

## V0 Failure Model

### Processes

- Can crash at any moment.
- Can restart later.

### Workers

- Can disappear during task execution.

### Network

- Connections can fail.
- Communication can be delayed.
- A client may not know whether an operation completed before a connection failure.

### Messages / Operations

- Application-level retries may cause the same operation to be requested more than once.

### Storage

- Initially assumed to be reliable.
- Storage failures will be introduced later.

### Machines

- Initial assumption: one machine, multiple processes.
- Multiple physical or logical nodes come later.

### Clocks

- No assumption of perfectly synchronized clocks.

### Fault Type

- Crash faults only, initially.
- No Byzantine behavior.

### Security

- Out of scope at the beginning.

## Expected Learning Progression

The expected progression is a pedagogical ladder:

```text
single-process execution
        ↓
concurrent execution
        ↓
multiple processes
        ↓
network communication
        ↓
process failures
        ↓
persistent state
        ↓
multiple nodes
        ↓
network partitions
        ↓
replication
        ↓
consensus
```

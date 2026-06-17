# Dispatch Queues in Swift (GCD)

Grand Central Dispatch (GCD) is Apple's low-level concurrency framework. Dispatch Queues are its core primitive — you submit work as closures, and GCD manages the underlying threads for you.

---

## Table of Contents

1. [Core Concepts](#core-concepts)
2. [Queue Types](#queue-types)
3. [QoS (Priority) Levels](#qos-priority-levels)
4. [Async vs Sync](#async-vs-sync)
5. [The Main Queue](#the-main-queue)
6. [Global Queues](#global-queues)
7. [Custom Queues](#custom-queues)
8. [DispatchGroup](#dispatchgroup)
9. [DispatchSemaphore](#dispatchsemaphore)
10. [Barrier Tasks](#barrier-tasks)
11. [asyncAfter](#asyncafter)
12. [Thread Safety Patterns](#thread-safety-patterns)
13. [Common Pitfalls](#common-pitfalls)
14. [Quick Reference](#quick-reference)
15. [GCD vs async/await](#gcd-vs-asyncawait)

---

## Core Concepts

Dispatch Queues work on two axes:

| Axis | Option A | Option B |
|---|---|---|
| **Execution style** | Serial | Concurrent |
| **How you dispatch** | `.async` / `.sync` | `.async` / `.sync` |

- **Serial / Concurrent**: describes the queue — how many tasks can run at once.
- **sync / async**: describes the caller — does it wait or not.
- **Serial**: tasks run one at a time, in the order they were submitted.
- **Concurrent**: tasks can overlap; they start in order but may finish in any order.
- **async**: the caller continues immediately without waiting.
- **sync**: the caller blocks until the submitted task completes.

<img width="1440" height="840" alt="image" src="https://github.com/user-attachments/assets/f88eb8b0-6aec-4cb4-a5e6-ceec1862bdb4" />

---

## Queue Types

### Main Queue

- Serial
- Always runs on the main (UI) thread
- Use for all UIKit/SwiftUI updates

```swift
DispatchQueue.main.async {
    self.label.text = "Updated"
}
```

### Global Queues

- Concurrent
- System-managed thread pools
- Six QoS levels (see below)

```swift
DispatchQueue.global(qos: .userInitiated).async {
    doBackgroundWork()
}
```

### Custom Queues

- You create and name them
- Serial by default; pass `.concurrent` attribute for concurrent behaviour
- Ideal for isolating subsystems (database, cache, networking)

```swift
let myQueue = DispatchQueue(label: "com.myapp.networking") OR let customConcurrentQueue = DispatchQueue(label: "com.concurrent", attributes: .concurrent)
let defaultConcurrentQueue = DispatchConcurrentQueue(label: "com.default.concurrent")
let defaultSerialQueue = DispatchSerialQueue(label: "com.default.serial")
```

---

## QoS (Priority) Levels

QoS (Quality of Service) tells the system how urgently to schedule the work. Higher priority = more CPU time, scheduled sooner.

| QoS | Use when... | Examples |
|---|---|---|
| `.userInteractive` | Work that directly drives the UI right now | Animation frames, gesture handling |
| `.userInitiated` | User triggered it and is waiting for a result | Tapping "Load", opening a file |
| `.default` | General work with no special urgency | Most background tasks |
| `.utility` | Long-running tasks the user is aware of | Progress bars, file import |
| `.background` | Work the user doesn't know about | Sync, prefetch, cleanup |
| `.unspecified` | Legacy; the system infers priority | Avoid in new code |

```swift
DispatchQueue.global(qos: .utility).async {
    processLargeFile()
}
```

> **Rule of thumb:** most background work should be `.userInitiated` (fast, user-triggered) or `.utility` (slower, background). Reserve `.userInteractive` for tiny, non-blocking tasks only.

---

## Async vs Sync

### `.async` — non-blocking

The caller submits the task and moves on immediately. The task runs when the queue is ready.

```swift
print("Before")
DispatchQueue.global().async {
    print("Background task")
}
print("After")  // prints before "Background task"
```

Output order: `Before` → `After` → `Background task`

### `.sync` — blocking

The caller waits until the task finishes before continuing.

```swift
print("Before")
DispatchQueue.global().sync {
    print("Sync task")
}
print("After")  // prints after "Sync task"
```

Output order: `Before` → `Sync task` → `After`

### When to use sync

Sync is useful for:
- Reading from a thread-safe resource (see Barrier pattern)
- Ensuring a setup step completes before proceeding
- Quick, non-blocking lookups

**Never** use `.sync` to dispatch onto the queue you're already on — this is a **deadlock**.

```swift
// DEADLOCK — never do this
DispatchQueue.main.sync {  // called from main thread
    doSomething()
}
```

---

## The Main Queue

The main queue is a serial queue that runs exclusively on the main thread. All UIKit and SwiftUI updates **must** happen here.

### The standard pattern

```swift
func loadAvatar(for user: User) {
    DispatchQueue.global(qos: .userInitiated).async {
        // Heavy work on background thread
        let image = downloadImage(from: user.avatarURL)

        DispatchQueue.main.async {
            // Back to main for UI update
            self.avatarImageView.image = image
        }
    }
}
```

### Rules for the main queue

- Never block it with expensive work (network, disk I/O, heavy computation)
- Never call `DispatchQueue.main.sync` from the main thread
- When in doubt about which thread you're on: `Thread.isMainThread`

```swift
assert(Thread.isMainThread, "Must be called on main thread")
```

---

## Global Queues

Global queues are shared concurrent queues provided by the system. You don't create or manage them — you just ask for one at a given QoS level.

```swift
// Get a reference
let queue = DispatchQueue.global(qos: .background)

// Or use inline
DispatchQueue.global(qos: .userInitiated).async {
    let result = fetchFromDatabase()
    DispatchQueue.main.async {
        self.display(result)
    }
}
```

You do **not** own global queues. Don't use barrier tasks on them — barriers only work on queues you created with `.concurrent`.

---

## Custom Queues

Create your own queues to organise work by subsystem and control execution order.

### Serial queue (default)

```swift
let dbQueue = DispatchQueue(label: "com.myapp.database")

dbQueue.async { write(recordA) }
dbQueue.async { write(recordB) }  // always runs after recordA
dbQueue.async { write(recordC) }  // always runs after recordB
```

Tasks are guaranteed to run in submission order. This is the simplest way to serialise access to a shared resource.

### Concurrent queue

```swift
let imageQueue = DispatchQueue(
    label: "com.myapp.imageProcessing",
    attributes: .concurrent
)

imageQueue.async { process(image1) }
imageQueue.async { process(image2) }  // may run simultaneously
imageQueue.async { process(image3) }
```

### Inheriting QoS

```swift
let highPriorityQueue = DispatchQueue(
    label: "com.myapp.urgent",
    qos: .userInteractive
)
```

### Target queue

You can assign a custom queue a target queue to set its underlying thread pool:

```swift
let myQueue = DispatchQueue(
    label: "com.myapp.worker",
    target: DispatchQueue.global(qos: .utility)
)
```

---

## DispatchGroup

`DispatchGroup` lets you track a collection of tasks and be notified when all of them complete. Ideal for fan-out/fan-in patterns (start multiple independent tasks, proceed when all done).

### Basic usage

```swift
let group = DispatchGroup()
let queue = DispatchQueue.global(qos: .userInitiated)

group.enter()
queue.async {
    let user = fetchUser()
    group.leave()
}

group.enter()
queue.async {
    let posts = fetchPosts()
    group.leave()
}

group.enter()
queue.async {
    let comments = fetchComments()
    group.leave()
}

// Notified on main queue once all three have called leave()
group.notify(queue: .main) {
    self.renderUI(user: user, posts: posts, comments: comments)
}
```

### Shorthand with `.async(group:)`

```swift
let group = DispatchGroup()
let queue = DispatchQueue.global()

queue.async(group: group) { fetchUser() }
queue.async(group: group) { fetchPosts() }

group.notify(queue: .main) {
    print("Done")
}
```

### Synchronous wait (use sparingly)

```swift
group.wait()  // blocks the current thread — don't use on main thread
group.wait(timeout: .now() + 5.0)  // with timeout
```

### Rules

- Every `enter()` must be balanced by a `leave()` — mismatching them causes either a hang or a crash.
- `notify` is non-blocking and preferred over `wait` in almost all cases.

---

## DispatchSemaphore

A semaphore limits how many tasks can proceed past a certain point simultaneously. Think of it as a ticket counter.

```swift
// Allow at most 3 concurrent downloads
let semaphore = DispatchSemaphore(value: 3)

for url in urls {
    DispatchQueue.global(qos: .utility).async {
        semaphore.wait()        // block until a slot is free
        download(url)
        semaphore.signal()      // release the slot
    }
}
```

### Binary semaphore (value: 1)

A semaphore with value `1` acts like a mutex:

```swift
let lock = DispatchSemaphore(value: 1)

lock.wait()
criticalSection()
lock.signal()
```

### Rules

- `wait()` decrements the count; blocks if count is 0.
- `signal()` increments the count; unblocks a waiting task.
- Always pair every `wait()` with a `signal()` — even in error paths.
- Avoid calling `wait()` on the main thread.

---

## Barrier Tasks

The barrier pattern solves the **readers-writers problem**: multiple readers can access a resource simultaneously, but a writer needs exclusive access.

Use a concurrent custom queue + `.barrier` flag for safe, performant read/write access.

```swift
class ThreadSafeCache {
    private var store: [String: Any] = [:]
    private let queue = DispatchQueue(
        label: "com.myapp.cache",
        attributes: .concurrent
    )

    // Many reads can happen simultaneously
    func get(_ key: String) -> Any? {
        return queue.sync {
            store[key]
        }
    }

    // A barrier write blocks all other work until it finishes
    func set(_ key: String, value: Any) {
        queue.async(flags: .barrier) {
            self.store[key] = value
        }
    }

    func remove(_ key: String) {
        queue.async(flags: .barrier) {
            self.store.removeValue(forKey: key)
        }
    }
}
```

### How it works

1. Barrier task is submitted to the concurrent queue.
2. It waits for all currently running tasks to finish.
3. It runs alone — no other task in the queue runs during this time.
4. Once done, new tasks proceed concurrently again.

### Important: barriers only work on custom concurrent queues

Do **not** use `.barrier` on global queues — it has no effect and may cause unexpected behaviour.

```swift
// WRONG — barrier on global queue does nothing useful
DispatchQueue.global().async(flags: .barrier) { ... }

// RIGHT — barrier on your own concurrent queue
myCustomConcurrentQueue.async(flags: .barrier) { ... }
```

---

## asyncAfter

Delays the execution of a task.

```swift
// Run after 2 seconds
DispatchQueue.main.asyncAfter(deadline: .now() + 2.0) {
    showToast("Saved!")
}

// Run after 500ms on background queue
DispatchQueue.global().asyncAfter(deadline: .now() + 0.5) {
    prefetchNextPage()
}
```

### DispatchTime vs DispatchWallTime

```swift
// Relative to now (pauses during sleep)
let deadline = DispatchTime.now() + 3.0

// Wall clock time (continues even if device sleeps)
let wallDeadline = DispatchWallTime.now() + 3.0
DispatchQueue.main.asyncAfter(wallDeadline: wallDeadline) { ... }
```

### Cancelling a delayed task

GCD doesn't support cancelling `asyncAfter` directly. Use a flag:

```swift
var isCancelled = false

DispatchQueue.main.asyncAfter(deadline: .now() + 2.0) {
    guard !isCancelled else { return }
    doWork()
}

isCancelled = true  // cancel before the deadline
```

For cancellable delayed work, prefer `DispatchWorkItem` or `Task` (async/await).

---

## Thread Safety Patterns

### Pattern 1: Serial queue (simplest)

Protect mutable state by funnelling all access through a single serial queue.

```swift
class Counter {
    private var value = 0
    private let queue = DispatchQueue(label: "com.myapp.counter")

    func increment() {
        queue.async { self.value += 1 }
    }

    func get(completion: @escaping (Int) -> Void) {
        queue.async { completion(self.value) }
    }
}
```

### Pattern 2: Concurrent queue + barrier (read-heavy)

Best when reads far outnumber writes.

```swift
// (See ThreadSafeCache example above)
```

### Pattern 3: DispatchWorkItem

A work item is a reusable, cancellable task wrapper.

```swift
var workItem: DispatchWorkItem?

func search(query: String) {
    workItem?.cancel()  // cancel previous search

    workItem = DispatchWorkItem {
        let results = performSearch(query)
        DispatchQueue.main.async {
            self.display(results)
        }
    }

    DispatchQueue.global(qos: .userInitiated)
        .asyncAfter(deadline: .now() + 0.3, execute: workItem!)
}
```

### Pattern 4: DispatchSpecificKey

Check which queue you're running on:

```swift
let key = DispatchSpecificKey<Bool>()
let myQueue = DispatchQueue(label: "com.myapp.myqueue")
myQueue.setSpecific(key: key, value: true)

if DispatchQueue.getSpecific(key: key) == true {
    print("We're on myQueue")
}
```

---

## Common Pitfalls

### 1. Deadlock — syncing onto the current queue

```swift
// DEADLOCK
DispatchQueue.main.sync { doSomething() }  // called from main thread

// Also a deadlock on a serial custom queue
let q = DispatchQueue(label: "my.queue")
q.sync {
    q.sync { doSomething() }  // inner sync blocks waiting for outer to finish
}
```

### 2. Updating UI from a background thread

```swift
// CRASH or visual glitch
DispatchQueue.global().async {
    self.label.text = "Done"  // WRONG — not on main thread
}

// Correct
DispatchQueue.global().async {
    let result = compute()
    DispatchQueue.main.async {
        self.label.text = result
    }
}
```

### 3. Retain cycles in async closures

```swift
// Retain cycle — self holds the queue, queue holds self
DispatchQueue.global().async {
    self.doSomething()
}

// Correct — use weak capture
DispatchQueue.global().async { [weak self] in
    self?.doSomething()
}
```

### 4. Using barrier on global queues

```swift
// Has no effect (or worse, undefined behaviour)
DispatchQueue.global().async(flags: .barrier) { writeSharedState() }

// Correct
myCustomConcurrentQueue.async(flags: .barrier) { writeSharedState() }
```

### 5. Unbalanced DispatchGroup enter/leave

```swift
group.enter()
doAsyncWork { result in
    if result.isError { return }  // MISSING group.leave() — hangs forever
    process(result)
    group.leave()
}

// Always leave in every path
group.enter()
doAsyncWork { result in
    defer { group.leave() }  // defer guarantees this runs
    guard !result.isError else { return }
    process(result)
}
```

### 6. Blocking the main thread

```swift
// Freezes the UI
let data = DispatchQueue.global().sync { fetchData() }  // blocks main

// Move the whole operation off main
DispatchQueue.global(qos: .userInitiated).async {
    let data = fetchData()
    DispatchQueue.main.async { self.render(data) }
}
```

---

## Quick Reference

```swift
// Background → main
DispatchQueue.global(qos: .userInitiated).async {
    let result = heavyWork()
    DispatchQueue.main.async { self.update(result) }
}

// Delay
DispatchQueue.main.asyncAfter(deadline: .now() + 1.5) {
    fadeIn(view)
}

// Group of concurrent tasks
let group = DispatchGroup()
let queue = DispatchQueue.global()

queue.async(group: group) { taskA() }
queue.async(group: group) { taskB() }
queue.async(group: group) { taskC() }

group.notify(queue: .main) { allDone() }

// Thread-safe reads + writes
let q = DispatchQueue(label: "com.myapp.store", attributes: .concurrent)
func read() -> Value { q.sync { store[key] } }
func write(v: Value) { q.async(flags: .barrier) { store[key] = v } }

// Limit concurrency
let sem = DispatchSemaphore(value: 4)
DispatchQueue.global().async {
    sem.wait()
    doWork()
    sem.signal()
}

// Cancellable delayed task
var item: DispatchWorkItem?
item = DispatchWorkItem { doWork() }
DispatchQueue.main.asyncAfter(deadline: .now() + 1, execute: item!)
item?.cancel()  // cancel before it fires
```

---

## GCD vs async/await

Swift 5.5 introduced structured concurrency with `async`/`await`, `Task`, and `actors`. For new code, prefer that model. GCD remains relevant for:

- Legacy codebases and Objective-C interop
- Fine-grained thread pool control (QoS tuning)
- Libraries that can't adopt Swift concurrency yet
- Understanding what `async`/`await` does under the hood

### Rough equivalents

| GCD | Swift Concurrency |
|---|---|
| `DispatchQueue.global().async { }` | `Task.detached { }` or `Task { }` |
| `DispatchQueue.main.async { }` | `await MainActor.run { }` |
| `DispatchGroup` | `async let` or `TaskGroup` |
| Serial queue | `actor` |
| Concurrent queue + barrier | `actor` |
| `DispatchSemaphore` | `AsyncSemaphore` (third-party) or `TaskGroup` |

### Serial queue → actor

```swift
// GCD serial queue pattern
class Counter {
    private var value = 0
    private let queue = DispatchQueue(label: "counter")
    func increment() { queue.async { self.value += 1 } }
}

// Swift concurrency equivalent
actor Counter {
    private var value = 0
    func increment() { value += 1 }  // actor serialises automatically
}
```

---

*Understanding GCD deeply makes async/await much easier to reason about — the same threading concepts apply, just with a cleaner syntax.*

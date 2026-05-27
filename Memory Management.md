# iOS Memory Management
### From ARC to Swift Concurrency — A Complete Study Guide

---

## Table of Contents
1. [Why Memory Management Matters](#1-why-memory-management-matters)
2. [The Old World — Manual Reference Counting (MRC)](#2-the-old-world--manual-reference-counting-mrc)
3. [ARC — Automatic Reference Counting](#3-arc--automatic-reference-counting)
4. [Strong, Weak, and Unowned References](#4-strong-weak-and-unowned-references)
5. [Retain Cycles — The Classic Enemy](#5-retain-cycles--the-classic-enemy)
6. [Closures and Capture Lists](#6-closures-and-capture-lists)
7. [Value Types vs Reference Types](#7-value-types-vs-reference-types)
8. [Swift Concurrency and Memory](#8-swift-concurrency-and-memory)
9. [Actors — Isolated Memory](#9-actors--isolated-memory)
10. [Memory Safety in Swift 5.5+](#10-memory-safety-in-swift-55)
11. [Instruments — Finding Leaks](#11-instruments--finding-leaks)
12. [Quick Reference Cheat Sheet](#12-quick-reference-cheat-sheet)

---

## 1. Why Memory Management Matters

Every object your app creates lives on the **heap** and occupies RAM. If you keep objects alive longer than needed, you get:

- 📈 **Memory leaks** — objects that are never freed, RAM keeps climbing
- 💥 **Crashes** — `EXC_BAD_ACCESS` from accessing a deallocated object
- 🐢 **Performance degradation** — system starts terminating your app (Jetsam kills)

iOS does **not** have a garbage collector (unlike Java/Android). Memory is managed via **reference counting**.

> **Key insight:** An object stays alive as long as at least one thing holds a strong reference to it. When the reference count hits 0, `deinit` is called and memory is freed.

---

## 2. The Old World — Manual Reference Counting (MRC)

Before ARC (pre-2011, Objective-C era), you had to manually track object lifetime:

```objc
// Objective-C — Manual Retain/Release
MyObject *obj = [[MyObject alloc] init]; // retain count = 1
[obj retain];                             // retain count = 2
[obj release];                            // retain count = 1
[obj release];                            // retain count = 0 → dealloc
[obj release];                            // 💥 CRASH — over-release
```

### Rules of MRC (you had to memorise these)
| Rule | Meaning |
|------|---------|
| `alloc` / `new` / `copy` | You own it — must call `release` |
| `retain` | You claim ownership |
| `release` | You give up ownership |
| `autorelease` | Release at end of the current run-loop cycle |

**This was error-prone.** One missed `release` = leak. One extra `release` = crash.

---

## 3. ARC — Automatic Reference Counting

Introduced in **Xcode 4.2 / LLVM 3.0 (2011)**, ARC has the compiler insert `retain`/`release` calls automatically at compile time. This is **not** a runtime garbage collector — it's purely a compile-time transformation.

```swift
// Swift with ARC — you write this:
class Dog {
    let name: String
    init(name: String) { self.name = name }
    deinit { print("\(name) is being deallocated") }
}

var dog1: Dog? = Dog(name: "Rex")  // RC = 1
var dog2 = dog1                    // RC = 2
dog1 = nil                         // RC = 1
dog2 = nil                         // RC = 0 → "Rex is being deallocated"
```

### How ARC works under the hood

```
Stack variable ──strong──▶ [Heap Object | RC: 2 ]
Stack variable ──strong──▶ ───────────────────────
```

When a strong reference goes out of scope, the compiler inserts a `release`. When RC reaches 0, `deinit` runs and memory is returned.

> **Important:** ARC works at **compile time**, not runtime. There is no background GC thread pausing your app.

---

## 4. Strong, Weak, and Unowned References

### `strong` (default)

```swift
class Owner {
    var pet: Dog?          // strong — Owner keeps Dog alive
}
```

Every reference is `strong` by default. It increments the retain count.

---

### `weak`

```swift
class Dog {
    weak var owner: Owner?   // weak — does NOT keep Owner alive
}
```

- Does **not** increment the retain count
- **Always optional** (`?`) — automatically set to `nil` when the object is deallocated
- Safe to use — you always check `if let owner = owner`
- Use when the referenced object can have a shorter lifetime than you

---

### `unowned`

```swift
class CreditCard {
    unowned let customer: Customer   // non-optional, no RC bump
    init(customer: Customer) { self.customer = customer }
}
```

- Does **not** increment the retain count
- **Non-optional** — assumes the object will always be alive when accessed
- If you access it after the object is deallocated → **crash** (`EXC_BAD_ACCESS`)
- Use only when you are 100% certain the referenced object outlives this one

### When to use what

| Scenario | Use |
|----------|-----|
| Normal ownership | `strong` |
| Delegate pattern | `weak` |
| Parent → child | `strong` |
| Child → parent | `weak` |
| Closure capturing `self` where self may be nil | `[weak self]` |
| Closure capturing `self` where self is guaranteed alive | `[unowned self]` |

---

## 5. Retain Cycles — The Classic Enemy

A retain cycle happens when two objects hold **strong** references to each other. Neither can reach RC = 0.

```swift
class Person {
    var apartment: Apartment?          // strong ──▶ Apartment
    deinit { print("Person deallocated") }
}

class Apartment {
    var tenant: Person?                // strong ──▶ Person (CYCLE!)
    deinit { print("Apartment deallocated") }
}

var john: Person? = Person()
var apt: Apartment? = Apartment()

john?.apartment = apt
apt?.tenant = john

john = nil   // Person RC: 1 (Apartment still holds it) — NOT freed
apt = nil    // Apartment RC: 1 (Person still holds it) — NOT freed
// ⚠️ Both deinits never called — memory leaked forever
```

### Fix: Make one side `weak`

```swift
class Apartment {
    weak var tenant: Person?    // breaks the cycle ✅
}
// Now: john = nil → Person RC = 0 → deinit → apt.tenant = nil
//      apt = nil  → Apartment RC = 0 → deinit
```

---

## 6. Closures and Capture Lists

Closures **capture** values from their surrounding context. By default, they capture objects **strongly**.

### The problem

```swift
class ViewController: UIViewController {
    var name = "Nithin"

    func setupTimer() {
        Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { timer in
            print(self.name)   // self is strongly captured 🚨
        }
        // Timer holds the closure strongly
        // Closure holds self strongly
        // ViewController can never be deallocated
    }
}
```

### Fix with capture list

```swift
// weak self — safe, self may become nil
Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [weak self] timer in
    guard let self = self else { return }
    print(self.name)   // ✅
}

// unowned self — use only when VC is guaranteed to outlive the timer
Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [unowned self] timer in
    print(self.name)
}
```

### Capture list syntax

```swift
let closure = { [weak self, weak delegate, unowned manager] in
    // ...
}
```

> **Rule of thumb:** In `@escaping` closures (callbacks, async handlers), always use `[weak self]` unless you have a very strong reason not to.

---

## 7. Value Types vs Reference Types

Swift's memory model has a crucial split:

### Value types (structs, enums, tuples)
- Stored on the **stack** (usually)
- **Copied** on assignment — no reference counting needed
- No retain cycles possible

```swift
struct Point {
    var x: Double
    var y: Double
}

var a = Point(x: 1, y: 2)
var b = a           // independent copy
b.x = 99
print(a.x)         // still 1 ✅
```

### Reference types (classes, actors)
- Stored on the **heap**
- **Shared** on assignment — ARC manages lifetime
- Retain cycles possible

```swift
class Node {
    var value: Int
    var next: Node?    // can form a cycle!
    init(_ v: Int) { value = v }
}
```

> **Swift prefers value types.** Prefer `struct` unless you need identity, inheritance, or shared mutable state.

---

## 8. Swift Concurrency and Memory

Swift 5.5 introduced `async/await`, `Task`, and `Actor`. This changes how you think about memory — especially captures in async contexts.

### The old way — completion handlers (problematic)

```swift
func loadData(completion: @escaping (Data?) -> Void) {
    URLSession.shared.dataTask(with: url) { [weak self] data, _, _ in
        guard let self = self else { return }
        self.process(data)
        completion(data)
    }.resume()
}
```

Problems: callback hell, easy to forget `[weak self]`, hard to cancel.

---

### The new way — async/await

```swift
func loadData() async throws -> Data {
    let (data, _) = try await URLSession.shared.data(from: url)
    return data
}

// Call site
Task {
    let data = try await loadData()
    await MainActor.run {
        self.updateUI(with: data)   // safe UI update on main thread
    }
}
```

### Tasks and memory

```swift
class DataManager {
    var task: Task<Void, Never>?

    func start() {
        task = Task { [weak self] in          // capture self weakly
            guard let self else { return }
            let result = await self.fetch()
            await self.handleResult(result)
        }
    }

    func cancel() {
        task?.cancel()    // always cancel tasks in deinit or viewDidDisappear
        task = nil
    }

    deinit {
        task?.cancel()    // ✅ critical — cancel detached tasks
    }
}
```

> **Key rule:** Tasks are reference-counted objects too. If a `Task` captures `self` strongly and is never cancelled, it creates a leak.

---

### Task groups

```swift
func fetchAll(ids: [Int]) async -> [Result] {
    await withTaskGroup(of: Result.self) { group in
        for id in ids {
            group.addTask {
                await fetchOne(id: id)
            }
        }
        var results: [Result] = []
        for await result in group {
            results.append(result)
        }
        return results
    }
}
// TaskGroup is automatically cancelled and cleaned up on scope exit ✅
```

---

## 9. Actors — Isolated Memory

Actors are a new reference type in Swift 5.5 that protect their mutable state from concurrent access, eliminating data races.

```swift
actor BankAccount {
    private var balance: Double = 0

    func deposit(_ amount: Double) {
        balance += amount
    }

    func withdraw(_ amount: Double) throws {
        guard balance >= amount else { throw BankError.insufficientFunds }
        balance -= amount
    }

    func getBalance() -> Double {
        return balance
    }
}
```

### Using an actor

```swift
let account = BankAccount()

// Must use await — crosses actor boundary
await account.deposit(100)
let balance = await account.getBalance()
```

- The actor **serialises** access — only one task can run inside it at a time
- Properties are **isolated** — you cannot access them from outside without `await`
- No data races, no need for locks/semaphores

### `@MainActor`

```swift
@MainActor
class ViewModel: ObservableObject {
    @Published var title = ""

    func update() {
        title = "Updated"   // always on main thread — safe for UI ✅
    }
}
```

`@MainActor` is a global actor that ensures code runs on the main thread. Apply it to ViewModels or any class that drives UI.

---

## 10. Memory Safety in Swift 5.5+

### Sendable protocol

Swift's concurrency system prevents data races at compile time using `Sendable`:

```swift
// A type is Sendable if it's safe to share across concurrency domains
struct Config: Sendable {
    let timeout: Int
    let retries: Int
}

// Non-Sendable — mutable class shared across tasks would be a race condition
class MutableState {   // NOT Sendable ⚠️
    var counter = 0
}
```

- Structs with `Sendable` properties are automatically `Sendable`
- Classes must be explicitly marked and must be thread-safe to be `Sendable`
- `@Sendable` on closures ensures they only capture `Sendable` values

### Structured Concurrency and lifetime

```swift
func process() async {
    async let result1 = fetchA()    // child task
    async let result2 = fetchB()    // child task

    let (a, b) = await (result1, result2)
    // Child tasks are automatically cancelled if this scope exits early ✅
}
```

Structured concurrency guarantees that child tasks **cannot outlive** their parent scope. This is a fundamental memory safety improvement over GCD.

---

## 11. Instruments — Finding Leaks

### Leaks Instrument

1. **Product → Profile** (`⌘ I`)
2. Choose **Leaks** template
3. Run your app, navigate through flows
4. Red bars in the timeline = leak detected
5. Click leak → see the object graph and call stack

### Allocations Instrument

- Shows all heap allocations over time
- Look for objects whose count only ever goes up (never freed)
- Use **Mark Generation** to snapshot before/after a user action

### Memory Graph Debugger (Xcode)

1. Run your app in Xcode
2. Click **Debug Memory Graph** button (grid icon in debug bar)
3. Look for purple `!` badges — those are leaked objects
4. Click a leaked object to see what's holding strong references to it

```
⚠️ Warning signs:
- ViewController count keeps growing as you push/pop
- Same object type showing 1000s of instances
- "Malloc" blocks you don't recognise
```

---

## 12. Quick Reference Cheat Sheet

### ARC Keywords

| Keyword | RC change | Optional? | Nil on dealloc? | Use case |
|---------|-----------|-----------|-----------------|----------|
| `strong` | +1 | No | No | Default ownership |
| `weak` | 0 | Yes (`?`) | Yes ✅ | Delegates, parent refs |
| `unowned` | 0 | No | Crash ⚠️ | Guaranteed-alive backrefs |

### Closure Captures

| Pattern | When to use |
|---------|-------------|
| `[weak self]` | Escaping closures, async callbacks, timers |
| `[unowned self]` | Closures where self is guaranteed alive |
| `[weak delegate]` | Multiple captures |
| No capture list | Non-escaping closures (safe by default) |

### Swift Concurrency Memory Rules

| Situation | Rule |
|-----------|------|
| Task captures self | Use `[weak self]` or ensure task is cancelled |
| UI updates in async code | `await MainActor.run { }` or `@MainActor` |
| Shared mutable state | Use `actor` |
| Cross-task data | Conform to `Sendable` |
| Child tasks | Structured concurrency — auto-cancelled with scope |

### Common Leak Patterns to Watch

```swift
// ❌ Delegate not weak
var delegate: MyDelegate?           // leak if delegate holds this object too

// ✅ Fix
weak var delegate: MyDelegate?

// ❌ Timer / NotificationCenter not removed
NotificationCenter.default.addObserver(self, ...)   // holds self forever

// ✅ Fix — remove in deinit
deinit {
    NotificationCenter.default.removeObserver(self)
}

// ❌ Escaping closure without [weak self]
fetchData { self.update() }         // self kept alive indefinitely

// ✅ Fix
fetchData { [weak self] in self?.update() }
```

---

## Summary

| Era | Mechanism | Manual work needed |
|-----|-----------|-------------------|
| Pre-2011 (ObjC) | MRC — manual retain/release | Everything |
| 2011–2021 | ARC | Breaking retain cycles manually |
| 2021+ (Swift 5.5) | ARC + Structured Concurrency + Actors | Sendable conformance, task cancellation |

> **The big picture:** ARC handles 90% of memory for you. Your job is to break retain cycles (`weak`/`unowned`), manage closure captures, cancel Tasks, and use `actor` for shared state in concurrent code. Swift Concurrency moves many of these guarantees to **compile time**, making it harder to write unsafe code.

---

*Study notes — iOS Memory Management | Swift ARC & Concurrency*

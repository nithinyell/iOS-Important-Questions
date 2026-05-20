# Swift Structured Concurrency

A complete guide to modern Swift concurrency — `async/await`, `Task`, `TaskGroup`, `Actor`, and beyond.

---

## Table of Contents

1. [The Problem with GCD Callbacks](#the-problem-with-gcd-callbacks)
2. [async / await basics](#async--await-basics)
3. [Task — bridging sync to async](#task--bridging-sync-to-async)
4. [Void and Never](#void-and-never)
5. [Sequential API calls](#sequential-api-calls)
6. [Parallel API calls — async let](#parallel-api-calls--async-let)
7. [Dynamic parallel work — TaskGroup](#dynamic-parallel-work--taskgroup)
8. [Update UI as results arrive](#update-ui-as-results-arrive)
9. [Cancellation](#cancellation)
10. [MainActor](#mainactor)
11. [Actors — thread-safe state](#actors--thread-safe-state)
12. [The full pattern](#the-full-pattern)
13. [GCD vs async/await equivalents](#gcd-vs-asyncawait-equivalents)

---

<img width="1440" height="680" alt="image" src="https://github.com/user-attachments/assets/7327a893-e646-4bfb-9000-16130cac3371" />

---

## The Problem with GCD Callbacks

Before structured concurrency, chaining async work meant deeply nested closures — often called "callback hell".

```swift
// ❌ GCD callback pyramid
func loadProfile() {
    fetchUser { user in
        fetchPosts(for: user) { posts in
            fetchComments(for: posts.first!) { comments in
                DispatchQueue.main.async {
                    self.render(user, posts, comments)
                }
            }
        }
    }
}
```

Problems with this approach:

- Error handling has to be threaded through every closure manually
- Cancellation is nearly impossible to implement cleanly
- The code reads inside-out, not top-to-bottom
- Forgetting to dispatch back to main causes crashes

---

## async / await basics

Mark a function `async` to say it can be suspended. Call it with `await` to say "wait here, but don't block the thread".

```swift
// Declare an async function
func fetchUser() async throws -> User {
    let data = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(User.self, from: data.0)
}

// Same logic as the callback pyramid — but flat
func loadProfile() async throws {
    let user     = try await fetchUser()
    let posts    = try await fetchPosts(for: user)
    let comments = try await fetchComments(for: posts.first!)
    await updateUI(user, posts, comments)
}
```

`await` suspends the function — it yields the thread so other work can run — then resumes when the result is ready. It does **not** block the thread.

---

## Converting a Completion Handler

withCheckedContinuation        // when it cannot fail
withCheckedThrowingContinuation // when it can fail

```
If you have an old-style network request:

fetchData(completion: @escaping (Result<String, Error>) -> Void) { ... }

Use code with caution.You can wrap it like this:swiftfunc fetchData() async throws -> String {
    try await withCheckedThrowingContinuation { continuation in
        // Call the original closure-based function
        fetchData { result in
            switch result {
            case .success(let data):
                // Resume and return data to the await call site
                continuation.resume(returning: data)
            case .failure(let error):
                // Resume and throw the error
                continuation.resume(throwing: error)
            }
        }
    }
}
```

```
continuation.resume(returning: value)   // success — hands back a value
continuation.resume(throwing: error)    // failure — throws at the await site
continuation.resume(with: result)       // takes a Result<T, Error> directly
```

---

## Task — bridging sync to async

You can't call `async` functions from synchronous code directly. `Task` is the bridge.

```swift
// Called from viewDidLoad, button tap, etc.
Task {
    do {
        try await loadProfile()
    } catch {
        print("Failed:", error)
    }
}
```

`Task` schedules the async work and returns immediately to the caller.

### Task priorities

```swift
Task(priority: .userInitiated) { await loadProfile() }
Task(priority: .background)    { await prefetchNextPage() }
Task(priority: .utility)       { await syncToServer() }
```

### Storing a task reference

```swift
private var loadTask: Task<Void, Never>?

loadTask = Task {
    await loadData()
}

// Cancel it later
loadTask?.cancel()
```

---

## Void and Never

These appear constantly in `Task` generics and are easy to confuse.

### Void

`Void` means the function **returns nothing**. It's literally a type alias for the empty tuple `()`.

```swift
func sayHello() -> Void { print("Hello") }
func sayHello()          { print("Hello") }  // identical — Void is implicit
```

### Never

`Never` means the function **never returns at all** — it crashes, loops forever, or exits the app. It is a real return type, not the absence of one.

```swift
func crashApp() -> Never {
    fatalError("Something went horribly wrong")
}

func runForever() -> Never {
    while true { }
}
```

### In Task generics

`Task<ResultType, ErrorType>`

```swift
Task<Void, Never>    // produces no value, cannot throw
Task<String, Never>  // produces a String, cannot throw
Task<Void, Error>    // produces no value, can throw
Task<String, Error>  // produces a String, can throw
```

In practice Swift infers these:

```swift
let t1 = Task { "hello" }            // inferred: Task<String, Never>
let t2 = Task { try await fetch() }  // inferred: Task<String, any Error>
let t3 = Task { doWork() }           // inferred: Task<Void, Never>
```

### The mental model

| | Came back | Didn't come back |
|---|---|---|
| Brought a value | `String`, `Int`, etc. | — |
| Brought nothing | `Void` | `Never` |

`Void` = *came back, brought nothing.*
`Never` = *didn't come back at all.*

---

## Sequential API calls

Use plain `try await` when each call depends on the result of the previous one.

```swift
func loadDashboard() async throws {
    let user    = try await api.fetchUser()           // ① wait for user
    let account = try await api.fetchAccount(user)    // ② needs user
    let feed    = try await api.fetchFeed(account)    // ③ needs account

    await MainActor.run {
        self.displayUser(user)
        self.displayAccount(account)
        self.displayFeed(feed)
    }
}
```

Each `await` suspends until the result arrives. Total time = sum of all call durations.

---

## Parallel API calls — async let

Use `async let` when calls are **independent** and can run at the same time.

```swift
func loadDashboard() async throws {
    // All three fire immediately — nothing waits yet
    async let user     = api.fetchUser()
    async let posts    = api.fetchPosts()
    async let comments = api.fetchComments()

    // Now we collect — they've been running in parallel
    let (u, p, c) = try await (user, posts, comments)

    await MainActor.run {
        self.render(u, p, c)
    }
}
```

### Timing comparison

<img width="1440" height="480" alt="image" src="https://github.com/user-attachments/assets/2f9fde0b-085f-4ce3-a27e-208c93d8c52c" />

```
Sequential:   [--user--][--posts--][--comments--]   = 900ms
Parallel:     [--user--]
              [--posts--]
              [--comments--]                         = 300ms
```

If **any** task throws, the others are automatically cancelled. You don't have to manage cleanup.

### Mixing sequential and parallel

```swift
func load() async throws {
    let user = try await api.fetchUser()         // must come first

    async let posts    = api.fetchPosts(user)    // these two
    async let settings = api.fetchSettings(user) // fire together

    let (p, s) = try await (posts, settings)
    render(user, p, s)
}
```

---

## Dynamic parallel work — TaskGroup

Use `TaskGroup` when the number of tasks isn't known at compile time.

```swift
func loadAvatars(for users: [User]) async throws -> [UIImage] {
    try await withThrowingTaskGroup(of: (Int, UIImage).self) { group in

        for (index, user) in users.enumerated() {
            group.addTask {
                let image = try await downloadImage(url: user.avatarURL)
                return (index, image)
            }
        }

        // Collect results — they arrive in completion order, not submission order
        var images = [UIImage?](repeating: nil, count: users.count)
        for try await (index, image) in group {
            images[index] = image
        }

        return images.compactMap { $0 }
    }
}
```

Results arrive as tasks finish — not in the order they were added. Use an index to reassemble in the original order.

### Non-throwing group

```swift
await withTaskGroup(of: SomeType.self) { group in
    group.addTask { ... }
}
```

### Limiting concurrency

If you have 100 URLs but don't want 100 simultaneous requests:

```swift
func downloadAll(urls: [URL]) async throws -> [Data] {
    try await withThrowingTaskGroup(of: Data.self) { group in
        var results: [Data] = []
        var index = 0

        // Seed with first 5
        for url in urls.prefix(5) {
            group.addTask { try await fetch(url) }
            index += 1
        }

        // As each finishes, add the next one
        for try await result in group {
            results.append(result)
            if index < urls.count {
                group.addTask { try await fetch(urls[index]) }
                index += 1
            }
        }

        return results
    }
}
```

---

## Update UI as results arrive

### Show sections as they load

```swift
@MainActor
func loadFeedProgressively() async {
    // Section 1 — show immediately when ready
    let featured = try? await api.fetchFeatured()
    featuredSection.display(featured)

    // Section 2 and 3 — fire in parallel
    async let trending     = api.fetchTrending()
    async let recommended  = api.fetchRecommended()

    if let t = try? await trending {
        trendingSection.display(t)
    }
    if let r = try? await recommended {
        recommendedSection.display(r)
    }
}
```

### Each task updates UI when done

```swift
func loadSections() async {
    await withTaskGroup(of: Void.self) { group in

        group.addTask {
            let data = try? await api.fetchSection(.trending)
            await MainActor.run { self.trending.display(data) }
        }

        group.addTask {
            let data = try? await api.fetchSection(.forYou)
            await MainActor.run { self.forYou.display(data) }
        }

        group.addTask {
            let data = try? await api.fetchSection(.newReleases)
            await MainActor.run { self.newReleases.display(data) }
        }
    }
}
```

---

## Cancellation

### Cancelling a task

```swift
let task = Task {
    try await loadDashboard()
}

task.cancel()  // signals cancellation
```

Cancellation is cooperative — the task must check for it. Most built-in APIs (`URLSession`, `Task.sleep`) already respond to cancellation automatically.

### Checking for cancellation manually

```swift
func processItems(_ items: [Item]) async throws {
    for item in items {
        try Task.checkCancellation()  // throws CancellationError if cancelled
        await process(item)
    }
}

// Or check without throwing
func processItems(_ items: [Item]) async {
    for item in items {
        guard !Task.isCancelled else { return }
        await process(item)
    }
}
```

### Cancel on disappear (standard pattern)

```swift
override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    loadTask = Task { await loadData() }
}

override func viewDidDisappear(_ animated: Bool) {
    super.viewDidDisappear(animated)
    loadTask?.cancel()  // don't waste network on invisible screens
}
```

### withTaskCancellationHandler

Useful when wrapping a legacy callback API:

```swift
func fetchWithCancel() async throws -> Data {
    try await withTaskCancellationHandler {
        try await doActualFetch()
    } onCancel: {
        legacyRequest.cancel()  // runs immediately when cancelled
    }
}
```

---

## MainActor

`@MainActor` is a global actor that guarantees code runs on the main thread. It replaces `DispatchQueue.main.async` with a compiler-enforced guarantee.

### Annotating a whole class

```swift
@MainActor
class HomeViewController: UIViewController {
    // Every method here runs on the main thread automatically
    func updateLabel(_ text: String) {
        label.text = text  // safe — no dispatch needed
    }
}
```

### Annotating individual methods

```swift
class MyService {
    @MainActor
    func showAlert(_ message: String) {
        // guaranteed main thread
    }
}
```

### Jumping to main from async context

```swift
func loadData() async throws {
    let result = try await api.fetch()

    await MainActor.run {
        self.label.text = result.name
    }
}
```

### The rule

Any `UIView`, `UIViewController`, or SwiftUI state must be touched on `@MainActor`. If you're in an async function on a background task, use `MainActor.run { }` to hop back.

---

## Actors — thread-safe state

An `actor` is like a `class` but it serialises access to its properties automatically. No locks, no dispatch queues, no data races.

### Basic actor

```swift
actor UserCache {
    private var cache: [String: User] = [:]

    func get(_ id: String) -> User? {
        cache[id]
    }

    func set(_ id: String, user: User) {
        cache[id] = user
    }
}
```

### Using an actor

```swift
let cache = UserCache()

Task {
    await cache.set("u1", user: someUser)  // await = crossing actor boundary
    let user = await cache.get("u1")
}
```

The `await` isn't because the method is slow — it's the compiler saying "you're crossing a concurrency boundary." The actor serialises all access internally.

### Actor vs serial DispatchQueue

```swift
// GCD serial queue — manual, no compiler help
class ThreadSafeCache {
    private var store: [String: User] = [:]
    private let queue = DispatchQueue(label: "cache")

    func get(_ id: String) -> User? {
        queue.sync { store[id] }
    }

    func set(_ id: String, user: User) {
        queue.async { self.store[id] = user }
    }
}

// Actor — compiler enforced, cleaner
actor UserCache {
    private var store: [String: User] = [:]

    func get(_ id: String) -> User? { store[id] }
    func set(_ id: String, user: User) { store[id] = user }
}
```

Same behaviour, but the actor version is enforced at compile time. Accessing `store` directly from outside is a compile error, not a runtime race.

### nonisolated

Mark a method `nonisolated` if it doesn't touch actor state and you don't want the caller to `await`:

```swift
actor Analytics {
    var eventCount = 0

    nonisolated var description: String {
        "Analytics tracker"  // doesn't touch actor state — no await needed
    }
}
```

---

## The full pattern

This is the complete production pattern for a UIViewController loading data:

```swift
@MainActor
class HomeViewController: UIViewController {

    private var loadTask: Task<Void, Never>?

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        loadTask = Task { await loadData() }
    }

    override func viewDidDisappear(_ animated: Bool) {
        super.viewDidDisappear(animated)
        loadTask?.cancel()
    }

    private func loadData() async {
        showSkeleton()
        defer { hideSkeleton() }  // always runs, even if we throw

        do {
            async let user  = api.fetchUser()
            async let posts = api.fetchPosts()

            let (u, p) = try await (user, posts)

            // Already on @MainActor — no dispatch needed
            nameLabel.text    = u.name
            postsTable.data   = p
        } catch is CancellationError {
            // navigated away — do nothing
        } catch {
            showError(error)
        }
    }
}
```

---

## GCD vs async/await equivalents

| GCD | Swift Concurrency |
|---|---|
| `DispatchQueue.global().async { }` | `Task { }` or `Task.detached { }` |
| `DispatchQueue.main.async { }` | `await MainActor.run { }` or `@MainActor` |
| `DispatchGroup` notify | `async let` or `TaskGroup` |
| Serial `DispatchQueue` | `actor` |
| Concurrent queue + barrier | `actor` |
| `DispatchSemaphore` | `TaskGroup` with bounded concurrency |
| `DispatchWorkItem.cancel()` | `task.cancel()` |
| `DispatchQueue.asyncAfter` | `try await Task.sleep(for: .seconds(2))` |

### Task.sleep

```swift
// GCD
DispatchQueue.main.asyncAfter(deadline: .now() + 2.0) {
    doSomething()
}

// async/await
try await Task.sleep(for: .seconds(2))
doSomething()
```

`Task.sleep` respects cancellation — if the task is cancelled during the sleep, it throws `CancellationError`. `asyncAfter` has no such mechanism.

---

*For older codebases or fine-grained thread control, GCD still applies. For all new Swift code, structured concurrency is the standard.*

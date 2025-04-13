# RBC April 3rd

## Round 1 - More of Swift Questions
1. **Struct vs Class - enums structs**
2. **String vs Weak vs Unowned**
3. **Actors**: Are they value or reference types? Discuss advantages.
4. **Automatic Reference Counting (ARC)**
5. **Protocols - Generic Protocols**: What are the advantages? Protocol to handle sum of int and strings.
6. **GCD - NSOperations - Dispatch Queues - NSLocks - Semaphores**
7. **Code Example**:
    ```swift
    class A {
    }

    struct B {
    }

    let a = a1

    a++
    did increments
    what will print
    ...
    ```
8. **Structured Concurrency**
   - Firing all API calls at a time.
   - Fire all API calls, keep showing UI. If you have one response exit and show that data.
9. **Design a view that has multiple posts**: How to cache image data using set/dict for cache.
10. **Disadvantage of closures**

## Round 2 - More Swift
1. **Never vs Void**
2. **Type Eraser**
3. **Custom Property Wrappers**
4. **Converting async function to closure and vice versa**
5. **How to implement enum that can be extended outside of module** - Implement struct with static let.
6. **Projected Value vs Wrapped Value**
7. **_ vs self**
8. **Handling Test Cases with Multiple Dependencies**
9. **Macros vs Swift Data**
10. **Test Case for Token Refresher**: How to avoid waiting in real time.
11. **Code Example**:
    ```swift
    func doSomething() async -> Int
    var someInt: Int {
        // How to implement doSomething here and return
        // Can we make use of dispatch queues
        // Will it cause deadlocks
    }
    ```

## Round 3 - Pure SwiftUI
1. **Explain all property wrappers**
2. **Why use environmental variables** when we can manually pass dependencies in inits.
3. **Parent to Child Views**: How to keep child view intact even if parent view refreshes.
4. **Passing view constraints to child from parent and vice versa**
5. **Handling 100 Images**: Implementation strategy.
6. **Data Passing from Parent to Child**
7. **Life Cycle of View**
8. **Task vs OnAppear**
9. **How to Refresh View in SwiftUI**
10. **Binding**: If child is updating something, how can parent have that value?
11. **Lazy VStack**
12. **Actors**
13. **Code Example with TapGesture**:
    ```swift
    VStack {
        topView
        bottomView
    }
    .tapGesture {
        print(1)
    }

    topView
        .frame
        .padding
    .tapGesture {
        print(2)
    }

    bottomView
        .padding
        .frame
    ```
14. **Compilition Error**: Returning two different types (Button vs CustomType).
    ```swift
    var cta: some View {
        case .one:
            return Button
        case .two:
            return Another type
    }
    ```
15. **Alignment vs Frame Leading**:
    ```swift
    VStack(alignment: .leading) {
        label1
        label2
    }
    .frame(alignment: .leading)
    ```

### SwiftUI (2019) Property Wrappers
- State: 
    - It is used inside `View` objects. It allows your view to respond to any changes made to @State
    - Use state for properties owned by the view
    - This will be always a `private` property
    - Very good for `primitive` types
- Binding:
    - Referes to a value type owned by different view
    - Changes to binding will effect the remote object also, as it have both read and write access
- Bindable:
    - Used to create bindings to the properties on `@observableObjects`  
- State Object:
    -  Similar to state but applied for `@observableObjects`  
- Observed Object:
    - Refer to instance of external class that conforms to `@observable` protocol 
- State Object vs Observed Object:
    |Observed Object|State Object|
    |-|-|
    |Used to observe & react to changes in externally provided observable objects|Used to create and own the life cycle of observable objects in a view|
- App Storage:
    - Read and write values from userDefaults 
- Environment Object:

### Actors
- SwiftUI provides the @StateObject and @ObservedObject property wrappers that are used with reference types, such as classes. However, these property wrappers are not sufficient for sharing state across multiple views in a way that is `safe and efficient`.
- In Swift 5.5, Apple introduced __actors__, a new concept for writing `concurrent code`. Actors are a reference type that can encapsulate state and allow you to interact with that state safely from __multiple threads.__ Actors help you avoid common concurrency issues, such as race conditions and data races.
```
actor Counter {
    var value: Int = 0
    
    func increment() {
        value += 1
    }
    
    func decrement() {
        value -= 1
    }
}
```

### MainActor
- The MainActor attribute was introduced in Swift 5.5 to mark classes, structures, or actors whose members `must be accessed exclusively from the main thread`. This feature is particularly useful in SwiftUI, where the user interface should only be updated from the main thread to avoid concurrency issues.

# Important Questions

# Basics
[Delegates](#Delegates)

[Protocols](#Protocols)

[NSNotifications](#NSNotifications)

[Optionals](#Optionals)

[Struct-Class](#Structs-Class)

[Notifications](#Notifications)

[enum](#enums)

[Generics](#Generics)

[Synthesize-Dynamic](#Synthesize-Dynamic)

[Frame-Bound](#Frame-Bound)

[Any-AnyObject](#Any-AnyObject)

[Accessibility-Label-Identifier](#Accessibility-Label-Indetifier)

[Closures-Blocks](#Closures-Blocks)

[CoreData](#CoreData)

[Memory-Management](Memory-Management)

[MethodSwizzling](#MethodSwizzling)

[Type-Casting](#Type-Casting)

[Access-Modifiers](#Access-Modifiers)

[Life-Cycle-of-App](#Life-Cycle-of-App)

[Life-Cycle-of-ViewController](#Life-Cycle-of-ViewController)

[AppDelegate](#AppDelegate)

[OOPS-Concepts](#OOPS-Concepts)

## Delegates
- Delegate is a design pattern that allows one object to send message to another object. One Object in a program acts on behalf of another.
- Delegate is an object that receive message from different object. Its a reference to the class which confirms that protocol.
- Example: Object A calls object B to perform an action. Once action is completed object A should know that B has completed task and took necessary action. This can be achieved. with Delegates.
  - A is a delegate of object B
  - B has reference of A
  - A will implement delegate methods of B
  - B will notify A through delegate methods
## Protocols
- Set of methods should be implemented by the class which confirms to that protocol.
```
Example:
  protocol Animal {
    var name: String { get }
    func makeSound()
}
```
- Protocol Extension - protocols can be extended to provide method and property implementations. Extend protocols to avoid duplication.
```
let names = []
let games = []
extension: collection {
func summarize() {
}
}
names.summarize()
games.summarize()
```
- Equatable Protocol: Its a protocol to do comparison.
- Protocol composition: Protocol composition is a feature that allows you to combine multiple protocols into a single.
```
protocol Swimmer {
    func swim()
}

protocol Flyer {
    func fly()
}

struct Bird: Swimmer, Flyer {
    func swim() {
        print("Bird is swimming")
    }
    
    func fly() {
        print("Bird is flying")
    }
}
```
## NSNotifications
- Super easy way to send updates from one part of code to another part of code.
- ```NotificationCenter.default.post(name: NSNotification.Name("MyNotificationName"), object: nil, userInfo: ["key": "value"])```
- We can listen to notifications by adding  `observers`
- From iOS9 no need to removeObserver, it uses weak references to know registered targets and de-allocates accordingly.
- While `delegation` is one to one; `notifications` are one to many.
## Optionals
- In swift variables /properties are non optional by default
- Optional is a type to indicate absence of value. It is defined by adding `Question mark ?` operator after type declaration
- Advantage: To check errors during compile time rather than run time. Avoid nil related crashes
  
```  
var optionalInt: Int?
var optionalString: String? 
```
- Force Wrapping: If we are sure about the value and we can use it, if there is no value app will crash. `use ! to unwrap`
- Optional Binding: Not sure about the value presence, if there is no value don't execute the block of code. `if let / if var`
- Implicit unwrapping: Sure about the value and don't want to unwrap always. `use ! while declaring` `let name: String! = "Apple"`
- Nil Coalescing: Some tike we want to use default value if optional is nil. `let resultName = name ?? "no name"`
- Optional Chaining:  Safely access properties and methods of an optional without unwrapping, and the result will be an optional itself. `let personName: Person = person?.name`
- Guard: Helps to avoid complex nested `if let` statements. Another benefit is early exit. `guard let unwrapped = optional else { return }`
## Struct-Class

Both of them will have properties, methods, initializers and conform to the protocol

Struct:
- Each instance will keep the unique copy of data - `Copy on writes`
- Structs are value types
- When struct is assigned to a var, constant or passed to a function, its value is copied instead of increasing the reference count
- Thread safe because its not sharable
- They are allocated on stack and occupies less memory and fast
  
Class:
- Each instance will share the same copy of data
- Class are reference type
- When a class is assigned to var, constant or passed to function, its reference count is increased
- They also have extra stuff like inheritance & de-initializers
- They are allocated in on heap memory

|Value Type|Reference Type|
|-|-|
|Int, Double, String, Array, Dictionary, Set, Struct, enum & Tuple|Classes, function & closures|

## Notifications
## enums
- Enums are data types allows to represent multiple cases/possibilities. Declare enum name with Capital letter and enum case with small letter
- Its a data types consists of named values called members.
  - Associated values:
    - Swift enums can store associated values of any types
      ```
      enum Devices {
          case iPhone(String)
          case iPad(String)
      }
      ```
  - Raw Values:
    - Enums have raw values associated to each member
      ```
      enum direction {
          case up = 1
          case down
          case right
      }
      ``` 
## Generics
## Synthesize-Dynamic
## Frame-Bound
## Any-AnyObject
## Accessibility-Label-Identifier
## Closures-Blocks
- They are referred as anonymous functions; self-contained blocks of code that can be assigned to variables, passed as arguments to functions, and returned from functions
- They are known as blocks in obj-c
  ```
  let closureName: (InputType) -> OutputType = { (parameters) in
      // Code that does something
      return result
  }
  ```
## Escaping Closures & Non escaping closures
- A non-escaping closure is one that is guaranteed to finish execution before the function containing it returns. This means that the closure is synchronous and its execution is confined within the scope of the function call.
  ```
  func doSomething(with closure: () -> Void) {
    closure()
  }

  doSomething {
    print("Closure executed.")
  }
  ```
- Escaping closure that might outlive the function containing it. It can be stored and executed at a later time, even after the function has returned. This is typically seen when dealing with asynchronous operations
## CoreData
## Memory-Management
- Strong: Its default behavior, used to create the strong reference of the object; It increases the reference count of the object, Reference will be maintained through out the life-cycle of the object. Example: `ViewControllers`
- Weak: We are pointing to the object but not increasing the reference count. Often used while creating parent child relation ship. Example: `IBOutlets`, `delegates`, `subViews` & `closures`
- unowned: Same as weak but `must not be optional`
---

Retain: Increases the retain count of object and takes the ownership of object
Release: Decreases the retain count of object and relinquishes ownership of object

---

Copy: It is required when object is mutable. It's commonly used with mutable objects to ensure that changes made to the copied object don't affect the original.

---

Retain Count: 
- The way in which memory is managed in Objective - C
- Create an object, retain count is +1
- Send object to retain message, retain count is +1
- Send object to release message, retain count is -1
- Send object to auto-release message, retain count is -1
- When retain count is `0` object is de-allocated memory is free

---

ARC (Automatic Reference Counting):
- A technique used to track and mange apps memory
- When a new class is created, it creates a memory and stores the information
- When the class is no longer needed, ARC frees up the memory used by that instance
- In simple words; A memory management technique in iOS/macOS development where the compiler automatically inserts retain, release, and auto-release messages, eliminating the need for manual memory management
- Object Life Cycle
  - allocation: takes memory
  - initialization: init
  - usage
  - de initialization: deinit
  - de-allocation: return memory
    
---

|Reference Count|Retain Count|
|-|-|
|Reference count refers to the number of references (pointers) to an object|Retain count is a specific type of reference count, represents the number of ownership claims on an object|

Reference cycles in closures:
- As closures are reference types, they cause cycles
- Capture list defines a relationship between closure and other objects in captures
  ```
  {[weak self] in
  ....
  }
  ```
---
|Atomic|Non Atomic|
|-|-|
|Allows only one thread to access the variable|Allows multiple threads to access the variable|
|Thread safe|Thread unsafe|
|Slow performance|Fast performance|

---

Memory Leaks
- A Situation where two objects are no longer required, but hold references to each other
- Since each has non-zero reference count, object cant de-allocate
- Strong reference cycles fools ARC and prevents it from cleaning
- Unless specified all the references are strong & impact reference count
  - Weak reference do not impact reference count of object also don't participate in life cycle management of an object
  - As weak references are optional, when count is zero, the reference can be automatically set to `nil`

___

||var|let|optional|non optional|
|-|-|-|-|-|
|strong|✅|✅|✅|✅|
|weak|✅|❌|✅|❌|
|unowned|✅|✅|❌|✅|

---

## MethodSwizzling
## Type-Casting
## Access-Modifiers
## Life-Cycle-of-App
## Life-Cycle-of-ViewController
## AppDelegate
## OOPS-Concepts

# Multi-Threading
[GCD](#GCD)

[NSOperations](#NSOperations)

## GCD
- Grand Central Dispatch (GCD) is a low-level, powerful, and efficient concurrency framework provided by Apple in macOS, iOS, and other platforms. 
- It offers a way to perform tasks concurrently and asynchronously, taking advantage of multi core processors to improve the performance and responsiveness of applications.

key concepts in GCD:

`Dispatch Queues`:
Dispatch queues are the fundamental building blocks of GCD. They are responsible for managing the execution of tasks in a concurrent or serial manner. There are two types of dispatch queues:

`Serial Dispatch Queue`: Executes tasks in a First-In-First-Out (FIFO) order, one at a time.
`Concurrent Dispatch Queue`: Executes tasks concurrently, allowing multiple tasks to run at the same time.
`Global Dispatch Queues`: GCD provides global dispatch queues with different quality of service (QoS) levels. These queues are already available for you to use without needing to create your own. The QoS levels include user-interactive, user-initiated, utility, and background.

`Creating Custom Dispatch Queues`:
You can create your own custom dispatch queues if you need more control over task execution. This allows you to define your own concurrency behavior.

`Dispatch Work Items`:
A dispatch work item encapsulates a task or closure along with associated flags and can be submitted to a dispatch queue for execution.

`Synchronous and Asynchronous Execution`:
You can submit tasks to dispatch queues synchronously (blocking the current thread until the task completes) or asynchronously (allowing the current thread to continue while the task executes).

`Dispatch Groups`:
Dispatch groups allow you to monitor the completion of multiple tasks. You can use them to wait for a group of tasks to finish before executing additional code.

`Semaphore`:
Semaphores can be used to control access to a resource by multiple threads, limiting the number of threads that can access the resource concurrently.

`Barrier Tasks`:
Barrier tasks allow you to insert a task into a concurrent dispatch queue that enforces a synchronization point, ensuring that all tasks before the barrier task are complete before the barrier task begins, and no tasks after the barrier task start until it completes.

Using GCD, we can write more responsive and efficient code by offloading time-consuming tasks to background threads, managing concurrent execution, and avoiding the complexities of low-level thread management.

```
DispatchQueue.global(qos: .background).async {
    // Perform some time-consuming task here
    print("Task performed on a background queue.")
}
```

## NSOperations
- Performs multithreading in obj-c/swift fashion. GCD follows c fashion.
- Its an API built on top on GCD
- Advantages
  - Make one operation dependent on another
  - reorder queues
  - change execution priority
  - Always follow concurrent, no serial
  - Cancel operation
  - completion blocks 
# Networking
[sync-async](#sync-async)
## sync-async
- Sync: The thread that initiated that operation will wait for the task to finish before continuing.
- Async: thread will not wait for the completion.
# Design Patterns TO DO - creational - structural - behavior - decorator - observer
[MVC](#MVC)

[Singleton](#Singleton)
## MVC
MVC stands for Model-View-Controller, and it is a widely used design pattern for architecture of software applications. In iOS development, the MVC pattern is commonplace and many of Apple's frameworks are built around it.

The MVC pattern is made up of three main objects: the Model, the View, and the Controller.

- The **Model** is where your data resides. Things like persistence, model objects, parsers, managers, and networking code live there.
- The **View** layer is the face of your app. Its classes are often reusable as they don’t contain any domain-specific logic. For example, a UILabel is a view that presents text on the screen, and it’s reusable and extensible.
- The **Controller** mediates between the view and the model via the delegation pattern. In an ideal scenario, the controller entity won’t know the concrete view it’s dealing with. Instead, it will communicate with an abstraction via a protocol. A classic example is the way a UITableView communicates with its data source via the UITableViewDataSource protocol
![image](https://github.com/nithinyell/iOS-Important-Questions/assets/18254027/6428c299-45b3-4c7d-baff-2ed12b74fb63)

## Singleton
- A design pattern ensure there is only one instance exists for the given class and there is a global access point for that instance.
- For the first time it will use lazy loading to create the single instance.
  ```
  class MySingleton {
    static let shared = MySingleton() // The single instance
    
    private init() {
        // Private initializer prevents external instantiation
    }
    
    func someMethod() {
        // Code for the singleton's functionality
    }
  }
  ```
- userDefaults, notificationCenter, fileManger are few examples for singleton from apple.
# Misc
[CI-CD](#CI-CD)
[Push-Notifications](#Push-Notifications)
[SSH-Pinning](#SSH-Pinning)
## CI-CD
## Push-Notifications
## SSH-Pinning

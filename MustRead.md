# Important Questions

# Basics
[Delegates](#Delegates)

[Protocols](#Protocols)

[NSNotifications](#NSNotifications)

[Optionals](#Optionals)

[Structs-Class](#Structs-Class)

[Notifications](#Notifications)

[emuns](#enums)

[Generics](#Generics)

[Synthesize-Dynamic](#Synthesize-Dynamic)

[Frame-Bound](#Frame-Bound)

[Any-AnyObject](#Any-AnyObject)

[Accessibility-Label-Indetifier](#Accessibility-Label-Indetifier)

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
- Equatable Protocol: Its a protocol to do comparisions.
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
- From iOS9 no need to removeObserver, it uses weak references to know registered targets and deallocates accordingly.
- While `delegation` is one to one; `notifications` are one to many.
## Optionals

## Structs-Class
## Notifications
## enums
## Generics
## Synthesize-Dynamic
## Frame-Bound
## Any-AnyObject
## Accessibility-Label-Indetifier
## Closures-Blocks
## CoreData
## Memory-Management
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

[Dispatch-Group](#Dispatch-Group)

[Semaphore](#Semaphore)

## GCD
- Grand Central Dispatch (GCD) is a low-level, powerful, and efficient concurrency framework provided by Apple in macOS, iOS, and other platforms. 
- It offers a way to perform tasks concurrently and asynchrosly, taking advantage of multicore processors to improve the performance and responsiveness of applications.

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

# Networking
[sync-async](#sync-async)
## sync-async
- Sync: The thread that initiated that operation will wait for the task to finish before continuing.
- Async: thread will not wait for the completion.
# Design Patterns TO DO - creational - structural - behaviour - decorator - observer
[MVC](#MVC)

[Singleton](#Singleton)
## MVC
## Singleton

# Misc
[CI-CD](#CI-CD)
[Push-Notifications](#Push-Notifications)
[SSH-Pinning](#SSH-Pinning)
## CI-CD
## Push-Notifications
## SSH-Pinning

# Important Questions

# Basics
[Delegates](#Delegates)

[Protocols](#Protocols)

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
## NSOperations
## Dispatch-Group
## Semaphore

# Networking
[sync-async](#sync-async)
## sync-async

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

## Generics
- Generics enable developer to write flexible and reusable functions that can work with any type
- Advantages are `optimization` and `reusability`
- In swift, arrays and dictionaries are generics
- Example - Array of Int / Array of Strings
  
## Synthesize-Dynamic
|Synthesize|Dynamic|
|-|-|
|It will generate setter and getter methods for the property|It will tell compiler that setter and getter are implemented somewhere in the super class|

## Frame-Bound
|Frame|Bound|
|-|-|
|Defines the position and size of a view within its superview's coordinate system|Defines the position and size of a view's content or subviews within the view's own coordinate system|

## Any-AnyObject
- Any is for Value or Reference type
- Anyobject is for only reference type
  
## AccessibilityLabel and AccessibilityIdentifier
- Lable is used for Screen reading (Voice Over)
- Id is used for identifying UI elements for UI testing

## Dynaminc vs Static Dispatch
- Static dispatch, also known as `early binding`, occurs at compile time.
- Dynamic dispatch, also known as `late binding`, occurs at runtime.

## Dynamic vs Static Libraries
- A static library is compiled and linked with the application code at compile time.
- A dynamic library is compiled separately from the application and linked at runtime.

## Stack vs Heap
  |Stack|Heap|
  |-|-|
  |Used for `static memory` allocation|Used for `dynamic memory` allocation|
  |`Value type`s are stored in stack|`Reference types` are stored in heap|
  |Allocation happens during compile time|Allocation happens during runtime|
  |LIFO||
  
## Access Modifiers
  - Final: Cannot override and subclass
  - Public: Only visisble outside module, cant override or subclass 
  - Open: Not only visible outside of module, we can override and subclass
  - Private: Visible only enclosing declerations and in the extension of same source file
  - FilePrivate: Visible only in the declared source file
  - Internal: default access level, visible in the same module

## lazy keyword
  - A lazy var is a property whose initial value is not calculated until the first time it’s called.
  - If it's never called, the var/function is never run, so it does help save processing time and memory.
    
## static keyword
  - We can directly access the static properties and methods of a class with help of class itself `instead of creating the instance of the class`.
    
## defer keyword
  - The code inside the defer block will be executed just before the function returns.
  - When you use multiple defer statements, the code within each defer block will be executed in reverse order of their appearance
    ```
    func exampleFunction() {
    print("Start")

    defer {
        print("First defer")
    }

    defer {
        print("Second defer")
    }

    print("End")
    }
    
    exampleFunction()

    // Output
    Start
    End
    Second defer
    First defer
    ```
    
## mutating func
  - Value types, such as structs and enums, are immutable by default, meaning their properties cannot be changed within methods.
  - `mutating` keyword is used to declare methods within a structure (value type) or an enumeration (enum) <b>that modify the instance itself</b>
  ```
  struct Stack<Element> {
    private var elements = [Element]()

    // Push an element onto the stack
    mutating func push(_ element: Element) {
        elements.append(element)
    }

    // Pop the top element from the stack
    mutating func pop() -> Element? {
        return elements.popLast()
    }

    // Peek at the top element without removing it
    func peek() -> Element? {
        return elements.last
    }

    // Check if the stack is empty
    var isEmpty: Bool {
        return elements.isEmpty
    }

    // Get the number of elements in the stack
    var count: Int {
        return elements.count
    }
  }

  // Example usage:
  var myStack = Stack<Int>()
  
  myStack.push(1)
  myStack.push(2)
  myStack.push(3)
  ```
## MethodSwizzling
  - Method swizzling is a technique in Objective-C (and Swift) that allows you to change the implementation of an existing method at runtime
  - It works by exchanging the implementations of two methods
  - Commonly used for debugging, analytics, or modifying framework behavior
  - Must be done carefully as it can lead to unexpected behavior
  ```swift
  extension UIViewController {
      static func swizzleViewDidAppear() {
          let originalSelector = #selector(viewDidAppear(_:))
          let swizzledSelector = #selector(swizzled_viewDidAppear(_:))
          
          guard let originalMethod = class_getInstanceMethod(UIViewController.self, originalSelector),
                let swizzledMethod = class_getInstanceMethod(UIViewController.self, swizzledSelector) else {
              return
          }
          
          method_exchangeImplementations(originalMethod, swizzledMethod)
      }
      
      @objc func swizzled_viewDidAppear(_ animated: Bool) {
          // Custom implementation
          print("View appeared!")
          swizzled_viewDidAppear(animated) // Calls original implementation
      }
  }
  ```

## Type-Casting
  ### Down Casting
    - Optional Down casting `as?`
    - Forced Down casting `as!`
  ### Type Checking
    - `is` used for type cehcking
  
## Life-Cycle-of-App
  ![life cycle app](https://github.com/nithinyell/iOS-Important-Questions/assets/18254027/d95d9a41-d17c-4f4c-8665-23b22f56a7bb)

## Life-Cycle-of-ViewController
  The lifecycle of a UIViewController follows a specific sequence of method calls:
  
  1. **init**: ViewController is initialized
  2. **loadView**: Creates the view hierarchy (called only once)
  3. **viewDidLoad**: View has been loaded into memory (called only once)
     - Good place for one-time setup
     - Set up UI elements, register notifications
  4. **viewWillAppear**: View is about to appear on screen
     - Can be called multiple times
     - Update UI with fresh data
  5. **viewWillLayoutSubviews**: View is about to layout its subviews
  6. **viewDidLayoutSubviews**: View has laid out its subviews
  7. **viewDidAppear**: View has appeared on screen
     - Start animations, start timers
  8. **viewWillDisappear**: View is about to be removed from screen
     - Pause operations, save state
  9. **viewDidDisappear**: View has been removed from screen
     - Stop timers, cleanup
  10. **deinit**: ViewController is being deallocated
  
  **Memory Warning**: `didReceiveMemoryWarning` - Called when system is low on memory

## self vs Self
  |self|Self|
  |-|-|
  |Refers to the current instance of a class or struct|Refers to type|
  
## Clean Architecture
  - Refer to [Clean-Code.md](Clean-Code.md) for comprehensive clean architecture practices
  - Key principle: Separation of concerns with clear dependency rules
  - Layers: UI → Presentation → Domain → Data
  - Benefits: Testability, maintainability, flexibility, independence from frameworks

## AppDelegate Methods
  **Application Lifecycle Methods:**
  - `application(_:didFinishLaunchingWithOptions:)` - App has launched, perform initial setup
  - `applicationWillResignActive(_:)` - App is about to become inactive (phone call, etc.)
  - `applicationDidEnterBackground(_:)` - App is now in background, save data
  - `applicationWillEnterForeground(_:)` - App is about to enter foreground
  - `applicationDidBecomeActive(_:)` - App is active and ready to use
  - `applicationWillTerminate(_:)` - App is about to terminate (not called if suspended)
  
  **Other Important Methods:**
  - `application(_:didRegisterForRemoteNotificationsWithDeviceToken:)` - Push notification registration
  - `application(_:didReceiveRemoteNotification:)` - Received remote notification
  - `application(_:open:options:)` - Handle URL schemes / Universal Links

## OOPS-Concepts
  **Four Pillars of Object-Oriented Programming:**
  
  1. **Encapsulation**: 
     - Bundling data and methods that operate on that data within a single unit (class/struct)
     - Hide internal implementation details using access modifiers (private, public, etc.)
     
  2. **Abstraction**: 
     - Hide complex implementation details and show only essential features
     - Achieved through protocols and abstract interfaces
     
  3. **Inheritance**: 
     - Ability to create new classes based on existing classes
     - Promotes code reuse (Classes only, not structs)
     
  4. **Polymorphism**: 
     - Ability to present the same interface for different data types
     - Method overriding and protocol conformance
     ```swift
     protocol Animal {
         func makeSound()
     }
     
     class Dog: Animal {
         func makeSound() {
             print("Bark")
         }
     }
     
     class Cat: Animal {
         func makeSound() {
             print("Meow")
         }
     }
     ```

## POP (Protocol Oriented Programming)
  - Programming paradigm that prioritizes protocol and value types over classes
  - Introduced by Apple at WWDC 2015 as "Swift is a Protocol-Oriented Language"
  
  **Key Benefits:**
  - **Value semantics**: Prefer structs over classes (no reference counting overhead)
  - **Protocol extensions**: Add default implementations to protocols
  - **Composition over inheritance**: Combine multiple protocols instead of deep class hierarchies
  - **Testability**: Easy to create mocks and test doubles
  
  **Example:**
  ```swift
  protocol Drivable {
      var speed: Double { get set }
      func drive()
  }
  
  extension Drivable {
      func drive() {
          print("Driving at \(speed) km/h")
      }
  }
  
  struct Car: Drivable {
      var speed: Double = 0
  }
  
  struct Bike: Drivable {
      var speed: Double = 0
  }
  ```
  
  **POP vs OOP:**
  |POP|OOP|
  |-|-|
  |Protocols and value types|Classes and inheritance|
  |Composition|Inheritance|
  |No reference counting|Reference counting (ARC)|
  |Thread-safe by default|Requires careful management|

# Misc
## SSH-Pinning
  - iOS app ensures a secure connection to a specific SSH server by "pinning" or validating the server's SSH key fingerprint
  - Avoid middle man attacks
  - Private Key Pinning - Client will use same private key as server
  - Certificate Pinning - Client will use the same certificate of server. Developer must keep updating the certificates depending on expiry dates
## CI-CD
## Push-Notifications
## CoreData

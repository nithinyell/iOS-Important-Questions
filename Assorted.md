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
    
## Stack vs Heap
  |Stack|Heap|
  |-|-|
  |Used for `static memory` allocation|Used for `dynamic memory` allocation|
  |Value types are stored in stack|Reference types are stored in heap|
  |Allocation happens during compile time|Allocation happens during runtime|
  |LIFO||
  
## mutating func
## CoreData
## MethodSwizzling
## Type-Casting
## Access-Modifiers
## Life-Cycle-of-App
  ![life cycle app](https://github.com/nithinyell/iOS-Important-Questions/assets/18254027/d95d9a41-d17c-4f4c-8665-23b22f56a7bb)
## Life-Cycle-of-ViewController
## AppDelegate Methods
## OOPS-Concepts
## POP (protocol oriented programming)
## self vs Self
  |self|Self|
  |-|-|
  |Refers to the current instance of a class or struct|Refers to type|
## Clean Architecture

# Misc
## CI-CD
## Push-Notifications
## SSH-Pinning
  - iOS app ensures a secure connection to a specific SSH server by "pinning" or validating the server's SSH key fingerprint
  - Avoid middle man attacks
  - Private Key Pinning - Client will use same private key as server
  - Certificate Pinning - Client will use the same certificate of server. Developer must keep updating the certificates depending on expiry dates

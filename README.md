# iOS-Important-Questions
<details>
  <summary>Dependency Injection</summary>
Dependency Injection is a powerful design pattern that promotes loose coupling and enhances the testability and flexibility of your code. It allows you to inject dependencies into a class rather than having the class create or obtain them itself. Here are some examples of different types of dependency injection in Swift:

1. **Constructor Injection**:
   ```swift
   protocol DataService {
       func fetchData() -> [String]
   }

   class DataServiceImp: DataService {
       func fetchData() -> [String] {
           return ["Data 1", "Data 2", "Data 3"]
       }
   }

   class MyClass {
       let dataService: DataService

       init(dataService: DataService) {
           self.dataService = dataService
       }

       func doSomething() {
           let data = dataService.fetchData()
           // Use the data
       }
   }

   let dataService = DataServiceImp()
   let myClass = MyClass(dataService: dataService)
   myClass.doSomething()
   ```
   In this example, `MyClass` has a required dependency on a `DataService` object. The dependency is passed into the constructor of `MyClass` and stored in a property. `MyClass` can then use the `DataService` object to perform some action, such as fetching data.

2. **Property Injection**:
   ```swift
   protocol DataService {
       func fetchData() -> [String]
   }

   class DataServiceImp: DataService {
       func fetchData() -> [String] {
           return ["Data 1", "Data 2", "Data 3"]
       }
   }

   class MyClass {
       var dataService: DataService!

       func doSomething() {
           let data = dataService.fetchData()
           // Use the data
       }
   }

   let dataService = DataServiceImp()
   let myClass = MyClass()
   myClass.dataService = dataService
   myClass.doSomething()
   ```
   In this example, `MyClass` has a property that represents a `DataService` dependency. The dependency is assigned to the property after `MyClass` has been initialized. `MyClass` can then use the `DataService` object to perform some action.

3. **Method Injection**:
   ```swift
   protocol DataService {
       func fetchData() -> [String]
   }

   class DataServiceImp: DataService {
       func fetchData() -> [String] {
           return ["Data 1", "Data 2", "Data 3"]
       }
   }

   class MyClass {
       func doSomething(dataService: DataService) {
           let data = dataService.fetchData()
           // Use the data
       }
   }

   let dataService = DataServiceImp()
   let myClass = MyClass()
   myClass.doSomething(dataService: dataService)
   ```
   In this example, `MyClass` has a method that requires a `DataService` dependency as a parameter. The dependency is passed into the method when it is called. `MyClass` can then use the `DataService` object to perform some action within the method.

By employing dependency injection, you achieve greater modularity, easier testing, and improved maintainability of your code. It enables you to decouple the creation and management of dependencies, allowing for more flexible and reusable components.
  </details>
  <details>
  <summary>SOLID Principles</summary>
  Certainly! Here's a consolidated explanation of the SOLID principles with detailed examples:

1. Single Responsibility Principle (SRP):
   The Single Responsibility Principle states that a class should have only one reason to change. It should have a single responsibility or purpose. Breaking down responsibilities into separate classes makes code more modular and maintainable.

   Example:
   ```swift
   struct Car {
       let make: String
       let model: String
       let color: String
       var numberOfWheels: Int
   }
   ```

   In this example, the `Car` struct is responsible for representing a car's properties. If we want to add behavior related to updating the number of wheels, it's better to extract that responsibility into a separate `CarWheelManager` struct.

2. Open/Closed Principle (OCP):
   The Open/Closed Principle states that classes or entities should be open for extension but closed for modification. You should be able to add new functionality without modifying existing code.

   Example:
   ```swift
   protocol CarFeature {
       func getFeature() -> String
   }

   struct CarSoundSystem: CarFeature {
       func getFeature() -> String {
           return "Bose Premium Sound System"
       }
   }

   struct Car {
       let make: String
       let model: String
       let color: String
       var numberOfWheels: Int
       var features: [CarFeature]
   }
   ```

   In this example, the `Car` struct is open for extension by allowing different car features to be added via the `CarFeature` protocol. We can create additional feature structs that conform to the protocol, such as `CarSoundSystem`, and add them to the `features` array.

3. Liskov Substitution Principle (LSP):
   The Liskov Substitution Principle states that subtypes must be substitutable for their base types without affecting the correctness of the program. It ensures that objects of a superclass can be replaced with objects of its subclasses without breaking the expected behavior.

   Example:
   ```swift
   class Vehicle {
       func startEngine() {
           print("Engine started")
       }
   }

   class Car: Vehicle {
       override func startEngine() {
           print("Car engine started")
       }
   }

   class ElectricCar: Car {
       override func startEngine() {
           print("Electric car engine started")
       }
   }
   ```

   In this example, the `Car` and `ElectricCar` classes are substitutable for the `Vehicle` class, as they inherit from it and provide the expected behavior of starting the engine. Code that depends on the `Vehicle` class can work correctly with instances of `Car` or `ElectricCar` without needing to know the specific subtype.

4. Interface Segregation Principle (ISP):
   The Interface Segregation Principle states that clients should not be forced to depend on interfaces they do not use. It promotes splitting large interfaces into smaller and more specific ones, tailored to the needs of the clients.

   Example:
   ```swift
   protocol CanFly {
       func fly()
   }

   protocol CanSwim {
       func swim()
   }

   struct Bird: CanFly {
       func fly() {
           print("Flying")
       }
   }

   struct Fish: CanSwim {
       func swim() {
           print("Swimming")
       }
   }
   ```

   In this example, we have separate interfaces `CanFly` and `CanSwim` that define specific behaviors. The `Bird` and `Fish` structs implement the respective interfaces they need, and clients can depend on the specific interface(s) they require, rather than a single interface with unnecessary methods.

5. Dependency Inversion Principle (DIP):
   The Dependency Inversion Principle states that high-level modules should not depend on low-level modules. Instead, both should depend on abstractions.
  
  Example:
  ```swift
  protocol Database {
      func save(data: String)
  }

  class MySQLDatabase: Database {
      func save(data: String) {
          print("Data saved to MySQL database: \(data)")
      }
  }

  class PostgreSQLDatabase: Database {
      func save(data: String) {
          print("Data saved to PostgreSQL database: \(data)")
      }
  }

  class DataManager {
      private let database: Database

      init(database: Database) {
          self.database = database
      }

      func saveData(data: String) {
          database.save(data: data)
      }
  }
  ```
In this updated example, we have added a `PostgreSQLDatabase` class that conforms to the `Database` protocol. The `DataManager` class still depends on the `Database` protocol, allowing different database implementations to be injected.

Now, you can create instances of `DataManager` with either a `MySQLDatabase` or `PostgreSQLDatabase` object, providing flexibility in choosing the specific database implementation without modifying the `DataManager` class.
  </details>

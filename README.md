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

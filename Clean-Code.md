# 🧼 Clean Code Practices in iOS

Writing clean code ensures your app is **readable**, **testable**, **scalable**, and **maintainable**. Clean code applies at both the **code level** (functions, variables) and **architectural level** (layers, responsibilities).

---

## ✍️ Code-Level Clean Code Practices

| Principle                        | Description & Example                                                              |
|----------------------------------|-------------------------------------------------------------------------------------|
| **Meaningful Names**            | Use clear, descriptive names. <br> ✅ `fetchUserProfile()` instead of `getData()`    |
| **Small Functions**             | Functions should do one thing and be short.                                        |
| **Single Responsibility**       | Each class/function should have only one reason to change.                        |
| **Avoid Deep Nesting**          | Use `guard` statements to flatten logic.                                           |
| **Consistent Formatting**       | Use consistent indentation, spacing, and braces.                                   |
| **Avoid Magic Numbers**         | Replace hardcoded values with named constants.                                     |
| **DRY (Don't Repeat Yourself)** | Reuse logic and avoid duplication.                                                 |
| **KISS (Keep It Simple, Stupid)**| Simpler is better; avoid overengineering.                                          |
| **YAGNI**                       | Don’t write code for future needs that may never come.                             |
| **Proper Error Handling**       | Use `do-catch`, `Result`, or `throws` effectively.                                 |
| **Readable Test Code**          | Test names should clearly describe the scenario.                                   |
| **Comment Why, Not What**       | Avoid obvious comments; explain intent or edge cases.                              |
| **Minimize Side Effects**       | Prefer pure functions when possible.                                               |
| **Refactor Regularly**          | Continuously clean up old code (Boy Scout Rule).                                   |

---

## 🏛️ Architectural-Level Clean Code Practices

Clean architecture structures your app so that **each layer has a clear purpose**, dependencies are **inverted**, and business logic is **isolated** from frameworks.

---

### 1. Separation of Concerns

Organize your app into logical layers:

- **UI Layer**: SwiftUI / UIKit Views  
- **Presentation Layer**: ViewModels, Presenters  
- **Domain Layer**: UseCases / Interactors  
- **Data Layer**: API clients, DB, Caching

> ✅ Makes testing, onboarding, and maintenance easier

---

### 2. Dependency Rule

**Inner layers should never depend on outer layers**. Invert dependencies using **protocols** and **dependency injection**.

```swift
protocol UserRepository {
    func fetchUser(id: String) -> User
}

class UserUseCase {
    let repo: UserRepository
    init(repo: UserRepository) {
        self.repo = repo
    }
}
```

---

### 3. Protocol-Oriented Abstraction

- Abstract dependencies behind protocols (e.g., `UserService`, `DatabaseClient`)
- Enables easy mocking and unit testing

---

### 4. Apply SOLID Principles

| Principle | Meaning                                                  |
|-----------|----------------------------------------------------------|
| **S**     | Single Responsibility – One class = one responsibility   |
| **O**     | Open/Closed – Extendable, not modifiable                 |
| **L**     | Liskov Substitution – Subtypes must replace base types   |
| **I**     | Interface Segregation – Use small, focused protocols     |
| **D**     | Dependency Inversion – Depend on abstractions            |

---

### 5. Feature-Based Modularization

Structure your project **by feature**, not by type.

```
📁 Features/
    📁 Login/
        - LoginView.swift
        - LoginViewModel.swift
        - LoginUseCase.swift
        - LoginRepository.swift
```

> ✅ Encourages separation, scalability, and reusability

---

### 6. Testability First

- **UI** = thin  
- **ViewModel** = testable logic  
- **UseCases** = business logic  
- Inject dependencies using **protocols**

---

### 7. Single Source of Truth

- Centralize state (e.g., in `ViewModel` or a `Store`)
- Prevent data duplication and inconsistencies

---

### 8. Composition Over Inheritance

- Prefer **protocol composition** instead of deep class hierarchies

---

### 9. Thin UI Layer

ViewControllers/SwiftUI Views should:
- Display data from ViewModel  
- Send user actions to ViewModel  
- Avoid business logic inside views

---

### 10. Avoid Leaky Abstractions

- Don’t expose framework objects (e.g., `NSManagedObject`, `URLSession`) to inner layers
- Use **DTOs (Data Transfer Objects)** to map and sanitize data at boundaries

---

## 🧭 Recommended Clean Architecture Flow

```
[ UI (SwiftUI / ViewController) ]
             ↓
[ ViewModel / Presenter ]
             ↓
[ UseCase / Interactor ]
             ↓
[ Repository (Protocol) ]
             ↓
[ API / DB / Cache (Concrete Implementation) ]
```

---

## 🔑 Summary Table

| Practice                     | Benefit                                       |
|-----------------------------|-----------------------------------------------|
| Separation of concerns      | Clear responsibilities, easy testing          |
| Protocol abstraction        | Decouples layers, easy to mock                |
| SOLID principles            | Scalable and maintainable architecture        |
| Feature-based modularization| Better project organization                   |
| Focus on testability        | Reliable and maintainable codebase            |
| Clean data boundaries       | No leaky abstractions or framework lock-in    |

---

> 💡 Clean architecture adds initial structure and planning but results in **long-term stability**, **flexibility**, and **developer happiness** in large-scale or evolving projects.

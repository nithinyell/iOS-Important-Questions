RBC April 3rd

### Round 1 - More of Swift Questions
1. Struct vs Class - enums structs 
2. String vs Weak vs Unknowed
3. Actors are value / reference - advantage 
4. ARC
5. Protocols - Generic Protocols ? What are advantages
6. GCD - NSOperations - Dispatch Queues - NSLocks - Senaphores
7. ```
   class A {
   }

   struct B {
   
   }

   let a = a1

   a ++
   did increments
   what will print
   ...
   ```
8. Structured concurreny
    - firing all API calls at a time
    - fire all API calls - keep showing UI - if you have one reponse exit and show that data 
9. Design a view that have multiple posts - have multiple image urls - how to cache image data (use set / dict for cache)
10. Disadvantage of closures
11.  

### Round 2 - More Swift 
1. Never vs Void
2. type eraser
3. custom property wrappers
4. how to convert async func to closure abd vice versa
5. how to implement enum that can be extended in outside of module - implement struct { static let ... }
6. projected value vs wrapped value
7. How to handle a test cases that have multiple dependencies
8. macros vs
9. How to make a test case for the token refresher - dont want to wait real time
10. ```
    func doSomething() aync -> Int
    var someInt: Int {
    how to implement doSomething here and return
      - can we make use of dispatch queues
      - will cause dead locks
    }
    ```

### Round 3 - Pure Swift UI
1. Explain all property wrappers
2. Why do we use environmental variable we can maually pass deps in inits
3. parent to child views {how to keep child view intact even if parent view refreshes}
4. How to pass view constraints to child from parent and vice versa
5. There are 100 images - how to implement -
6. How to pass data from parent to child
7. Binding - if child is updating something how can parent have that value
8. lazy vstack
9. actors
10. ```
      vstack {
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
11. ```
      This will give compilition error saying returing two diff types button vs customType
      var cta: some View {
      case .one:
        return Button
      case .two:
        return Another type
      }
   ```
12. ```
       what is #1 leading for vs frame leading
       vstack(alignment: .leading) {
         label1
         label2
       }
       .frame(alignment: .leading)
    ```

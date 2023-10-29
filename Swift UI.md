### Swift UI Property Wrappers
- State: 
    - It is used inside `View` objects. It allows your view to respond to anychanges made to @state
    - USe state for properties owned by the view
    - This will be always a `private ` property
    - Very good for `premitive` types
- Binding:
    - Referes to a value type owned by different view
    - Chnages to binding will effect the remote object also, as it have both read and write access
- Bindable:
    - Used to create bindings to the properties on `@observableObjects`  
- State Object:
    -  Similar to state but applied for `@observableObjects`  
- Observed Object:
    - Refer to instance of external class that conforms to `@observable` protocol 
- State Object vc Observed Object:
    |Observed Object|Stage Object|
    |-|-|
    |Useed to observe & react to changes in externally provided observable objects|Used to create and own the life cycle of observable objects in a view|
- App Storage:
    - Read and write values from userDefaults 
- Environment Object:

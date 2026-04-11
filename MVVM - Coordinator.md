Coordinator — the only one who pushes/presents screens. Owns UINavigationController.
ViewModel — holds state (@Published), calls the service, and tells the Coordinator when to navigate. Zero UIKit imports.
ViewController — renders UI and binds to ViewModel with Combine. Never pushes itself.
Model/Service — plain structs + a protocol so you can swap in a mock for tests.

The golden rule: when a user taps something → VC calls viewModel.didSelectItem() → ViewModel calls coordinator.showDetail() → Coordinator creates the next VC and pushes it. Navigation logic never leaks into VCs or ViewModels.

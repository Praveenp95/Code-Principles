<img width="1402" height="1122" alt="ChatGPT Image Apr 23, 2026, 10_59_15 PM" src="https://github.com/user-attachments/assets/ce729266-0266-4c6c-b18b-1d38354d4731" />

1. Introduction
2. Naming (Variables, Functions, Types)
3. Functions
4. Comments
5. Formatting & Readability
6. Objects & Data Structures
7. Error Handling
8. SOLID Principles
9. Testing
10. Concurrency (Swift)
11. Architecture (iOS)
12. UIKit Practical Problems
13. Principle Conflicts
14. Overengineering
15. Code Review Thinking
16. Philosophy

1. Introduction

🧑‍💻 What This Repo Covers

Each principle in this repo explains:

What it is
When to use
When to avoid
Tradeoffs
Real-world Swift example
Code review discussion

🚀 Goal

Write code that is easy to understand and easy to change — nothing more.

2. Naming (Variables, Functions, Types)

🧾 What it is

Naming is about choosing identifiers that clearly express intent.

🎯 Why it matters
Reduces time to understand code
Avoids misinterpretation
Improves maintainability

Real-world Examples

❌ Over Generic
```swift
func process(data: [String: Any]) { }
```
✅ Clear Intent
```swift
func processUserResponse(data: [String: Any]) { }
```
❌ Overly Verbose
```swift
let userAgeValueFromDatabaseResponse = 25
```
✅ Balanced
```swift
let userAge = 25
```

3. Functions

| Approach           | Benefit          | Risk               |
| ------------------ | ---------------- | ------------------ |
| Small functions    | Clear, testable  | Too many jumps     |
| Large functions    | Easy flow        | Hard to understand |
| Reusable functions | Less duplication | Overengineering    |

Real-world iOS Examples
❌ Bad (Multiple Responsibilities)
```swift
func loadProducts() {
    showLoader()
    
    API.fetchProducts { response in
        let products = parse(response)
        self.products = products
        self.tableView.reloadData()
        self.hideLoader()
    }
}
```
Problems:
Networking + Parsing + UI update in one place
Hard to test
Hard to reuse

✅ Better (Separated Responsibilities)
```swift
func loadProducts() {
    showLoader()
    fetchProducts()
}

private func fetchProducts() {
    API.fetchProducts { [weak self] response in
        guard let self else { return }
        let products = self.parse(response)
        self.updateUI(with: products)
    }
}

private func parse(_ response: Data) -> [Product] {
    // parsing logic
}

private func updateUI(with products: [Product]) {
    self.products = products
    self.tableView.reloadData()
    self.hideLoader()
}
```
Clear separation:
Fetching
Parsing
UI update

4. Comments

Code should be clear enough to reduce the need for comments.
Use comments only when they add value.
Comments explain why something is done, not what the code is doing.

❌ Avoid
```
// get user count
let count = users.count
```

👉 Code already explains this.

✅ Use
```
// API may return duplicate users, so filtering unique IDs
let uniqueUsers = removeDuplicates(users)
```

👉 Explains reasoning, not action.

🎯 Rule

If code can be improved to remove the comment → do it
If logic needs explanation → comment it

5. Formatting & Readability

Code should be formatted to make it easy to scan and understand quickly.

🧾 Core Principle

Structure code so that intent is visible at a glance.

❌ Poor Formatting
```
func fetch(){API.call{res in if res.success{self.data=res.data;tableView.reloadData()}}}
```
👉 Hard to read, no visual structure

✅ Better Formatting
```
func fetch() {
    API.call { [weak self] response in
        guard let self else { return }
        
        if response.success {
            self.data = response.data
            self.tableView.reloadData()
        }
    }
}
```
👉 Clear structure, easy to follow

📐 Spacing & Grouping
❌ Bad
```
let name = user.name
let age = user.age
print(name)
print(age)
```
✅ Better
```
let name = user.name
let age = user.age

print(name)
print(age)
```
👉 Group related logic

📏 Line Length
❌ Bad
```
let result = users.filter { $0.isActive }.map { $0.name }.sorted().joined(separator: ", ")
```
✅ Better
```
let result = users
    .filter { $0.isActive }
    .map { $0.name }
    .sorted()
    .joined(separator: ", ")
```
🎯 Rule

If code is hard to scan, it’s hard to understand.

6. Objects & Data Structures

Use the right type to represent data clearly and safely.

🧾 Core Principle

Choose between struct and class based on behavior and ownership, not habit.

🎯 struct vs class
Use struct	Use class
Value type (copy)	Reference type (shared)
Immutable data	Shared mutable state
Safer, predictable	Needed for identity/lifecycle
❌ Wrong Choice
```
class User {
    var name: String
    var age: Int
}
```
👉 No need for reference semantics

✅ Better
```
struct User {
    let name: String
    let age: Int
}
```
👉 Safer and simpler

📦 Encapsulation
❌ Bad
```
struct BankAccount {
    var balance: Double
}
```
👉 Anyone can modify directly

✅ Better
```
struct BankAccount {
    private(set) var balance: Double = 0

    mutating func deposit(_ amount: Double) {
        balance += amount
    }
}
```
👉 Controlled access

📱 iOS Example
❌ Bad
```
class Product {
    var price: Double
}
```
✅ Better
```
struct Product {
    let price: Double
}
```
👉 Models should usually be value types

⚠️ When to Use class
```
class SessionManager {
    static let shared = SessionManager()
    private init() {}
}
```
👉 Shared state across app

🎯 Rule

Prefer struct by default.
Use class only when you need shared state or identity.

7. Error Handling

Errors are part of the normal flow — handle them clearly and safely.

🧾 Core Principle

Handle errors explicitly based on impact.
Do not crash. Do not ignore.

🎯 Why it matters

Prevents crashes
Makes failures predictable
Improves debugging

❌ Common Mistakes
1. Force Unwrap (Crash Risk)
```
let url = URL(string: urlString)!
```
👉 Crashes if invalid

2. Ignoring Errors
```
try? saveData()
```
👉 Failure is hidden

3. No Failure Handling
```
func fetchUser() {
    API.call { data in
        self.user = data
    }
}
```
👉 No error path

✅ Better Approach
Explicit Handling
```
do {
    try saveData()
} catch {
    log(error)
}
```
Result Type (Clear Flow)
func fetchUser(completion: (Result<User, Error>) -> Void)
📱 Real-world iOS Example
❌ Bad
```
func fetchProducts() {
    API.call { data in
        self.products = data
        self.tableView.reloadData()
    }
}
```
✅ Better
```
func fetchProducts() {
    API.call { [weak self] result in
        guard let self else { return }
        
        switch result {
        case .success(let products):
            self.updateUI(products)
        case .failure(let error):
            self.showError(error)
        }
    }
}
```
⚠️ Important: [weak self]
❌ Problem (Strong Reference)
```
API.call {
    self.doSomething()
}
```
👉 Can cause memory leak

✅ Solution
```
API.call { [weak self] in
    guard let self else { return }
    self.doSomething()
}
```
👉 Prevents retain cycle

When to Use
Async closures (API, background tasks)
When self may be deallocated
When NOT Required
```
UIView.animate(withDuration: 0.3) {
    self.view.alpha = 0
}
```
👉 Short-lived closure

🧠 Error Types Thinking
Type	Example	Handling
Recoverable	Network fail	Retry / show UI
User Error	Invalid input	Show message
Critical	Data corruption	Stop flow
🎯 Rule

Handle errors where decisions are needed
Use [weak self] in async closures
Avoid silent failures

🧠 Final Take

Don’t crash
Don’t ignore
Don’t over-handle

👉 Handle based on impact

8. SOLID Principles

SOLID helps structure code to be flexible, maintainable, and scalable.

🧾 Core Principle

Write code that is easy to extend without breaking existing logic.

🔹 S — Single Responsibility Principle (SRP)

A class/function should have only one reason to change.

❌ Bad
```
class UserManager {
    func fetchUser() { }
    func saveUser() { }
    func validateUser() { }
}
```
👉 Multiple responsibilities

✅ Better
```
class UserService {
    func fetchUser() { }
}

class UserValidator {
    func validateUser() { }
}
```
👉 Clear separation

🔹 O — Open/Closed Principle (OCP)

Code should be open for extension, closed for modification.

❌ Bad
```
func calculatePrice(type: String) -> Double {
    if type == "A" {
        return 100
    } else {
        return 200
    }
}
```
✅ Better
```
protocol Pricing {
    func price() -> Double
}

class TypeA: Pricing {
    func price() -> Double { 100 }
}

class TypeB: Pricing {
    func price() -> Double { 200 }
}
```
👉 Add new types without modifying existing code

🔹 L — Liskov Substitution Principle (LSP)

Subclasses should replace parent without breaking behavior.

❌ Bad
```
class Bird {
    func fly() { }
}

class Penguin: Bird {
    override func fly() {
        fatalError("Can't fly")
    }
}
```
👉 Breaks expectation

✅ Better
```
protocol Bird { }

protocol Flyable {
    func fly()
}

class Sparrow: Bird, Flyable {
    func fly() { }
}

class Penguin: Bird { }
```
🔹 I — Interface Segregation Principle (ISP)

Don’t force classes to implement unused methods.

❌ Bad
```
protocol Worker {
    func work()
    func eat()
}

class Robot: Worker {
    func work() { }
    func eat() { } // not needed
}
```
✅ Better
```
protocol Workable {
    func work()
}

protocol Eatable {
    func eat()
}
```
🔹 D — Dependency Inversion Principle (DIP)

Depend on abstractions, not concrete implementations.

❌ Bad
```
class ProductViewController {
    let api = APIService()
}
```
✅ Better
```
protocol APIServiceProtocol {
    func fetch()
}

class ProductViewController {
    let api: APIServiceProtocol
}
```
10. Concurrency (Swift)

Concurrency helps run tasks without blocking the main thread.

🧾 Core Principle

Keep the UI thread free.
Do heavy work in the background, update UI on main thread.

🎯 Why it matters

Prevents UI freeze
Improves performance
Keeps app responsive

❌ Common Mistakes
1. Blocking Main Thread
```
func loadData() {
    let data = try! Data(contentsOf: url)
    self.updateUI(data)
}
```
👉 UI freezes

2. Updating UI from Background Thread
```
DispatchQueue.global().async {
    self.label.text = "Done"
}
```
👉 Unsafe UI update

✅ Better Approach
Background Work + Main Thread Update
```
DispatchQueue.global().async {
    let data = fetchData()
    
    DispatchQueue.main.async {
        self.updateUI(data)
    }
}
```
🚀 Modern Swift (Preferred)
async / await
```
func loadData() async {
    let data = await fetchData()
    updateUI(data)
}
Calling Async Function
Task {
    await loadData()
}
⚠️ Important: MainActor
@MainActor
func updateUI(_ data: Data) {
    label.text = "Updated"
}
```
👉 Ensures UI runs on main thread

⚠️ Combine with [weak self]
```
Task { [weak self] in
    guard let self else { return }
    let data = await fetchData()
    self.updateUI(data)
}
```
👉 Avoid memory leaks in async tasks

🧠 Concurrency Thinking
Task Type	Where to Run
Network call	Background
Parsing	Background
UI update	Main thread
⚖️ Tradeoffs
Approach	Benefit	Risk
GCD	Simple	Hard to manage
async/await

11. Architecture (iOS)

Architecture defines how responsibilities are structured in your app.

🧾 Core Principle

Separate concerns so that UI, business logic, and data handling don’t mix.

🎯 Why it matters

Improves maintainability
Makes code testable
Reduces coupling

❌ Common Problem (Massive ViewController)
```
class ProductViewController: UIViewController {
    
    func fetchProducts() { }
    func parseData() { }
    func setupUI() { }
    func handleTap() { }
}
```
👉 Everything in one place
👉 Hard to test and maintain

✅ Better Separation (Basic MVVM)
```
class ProductViewController: UIViewController {
    let viewModel = ProductViewModel()
}
class ProductViewModel {
    func fetchProducts() { }
}
```
👉 UI and logic separated

📱 MVC (Default UIKit)
Model → Data
View → UI
Controller → Handles everything

👉 Problem: Controller becomes huge

📱 MVVM (Better for Scaling)
View → UI
ViewModel → Logic
Model → Data

👉 Cleaner separation

⚠️ When NOT to Overuse Architecture
```
class ProductCoordinator { }
class ProductInteractor { }
class ProductPresenter { }
```
👉 Too many layers for small feature

⚖️ Tradeoffs
Approach	Benefit	Risk
MVC	Simple	Massive ViewController
MVVM	Better separation	Slight complexity
Advanced (VIPER)	Scalable	Overengineering

# iOS-Coding-Conventions

## Goal ##

The document enlist code snippets focusing on common mistakes and their suggested solution.

### 1. Optionals ###

* It is better to use a non-optional or regular optional property, except when using @IBOutlets. Don't use force unwraps (`!`, `as!`, `try!`, etc)

__Not Preferred__

```
let name: String!
let number = text as! Int
```

__Preferred__

```
let name: String?
let number = text as? Int
```
---
* When unwrapping optionals, `if let` is better suited for situations where you still need to move forward after failing. For other cases, use `guard let` to improve readability by focusing on the happy-path and can also reduce nesting by keeping your code flatter.

__Not Preferred__

```
if let id = json[Constants.id] as? Int {
    if let firstName = json[Constants.firstName] as? String {
        if let lastName = json[Constants.lastName] as? String {
            if let initials = json[Constants.initials] as? String {
                /* ... */
            }
        }
    }
}

OR

// Nested
if let url = URL(string: string) {
    UIApplication.shared.open(url)
    // Make API Call
    // Log data
}
```

__Preferred__

```
if
    let id = json[Constants.Id] as? Int,
    let firstName = json[Constants.firstName] as? String,
    let lastName = json[Constants.lastName] as? String,
    let initials = json[Constants.initials] as? String {
        /* ... */
}

// Flat
guard let url = URL(string: string) else {
    return
}
UIApplication.shared.open(url)
// Make API Call
// Log data
```

### 2. Properties ###

* Use `let` over `var` if property is read-only and use lazy initialization wherever possible.


__Not Preferred__

```
var apiKeyConstant = "asdqwe123"

// Instantiates everytime
var locationManager = makeLocationManager()

```

__Preferred__

```
// Constant
let apiKeyConstant = "asdqwe123"

// Instantiates when required
lazy var locationManager = makeLocationManager()

private func makeLocationManager() -> CLLocationManager {
  let manager = CLLocationManager()
  manager.desiredAccuracy = kCLLocationAccuracyBest
  manager.delegate = self
  manager.requestAlwaysAuthorization()
  return manager
}
```

### 3. Switch ###

* Use multiple values in a single case wherever applicable.

__Not Preferred__

```
switch someCharacter {
case "a":
    print("\(someCharacter) is a vowel")
case "e":
    print("\(someCharacter) is a vowel")
case "i":
    print("\(someCharacter) is a vowel")
case "o":
    print("\(someCharacter) is a vowel")
case "u":
    print("\(someCharacter) is a vowel")
default:
    print("\(someCharacter) is not a vowel")
}
```

__Preferred__

```
switch someCharacter {
case "a", "e", "i", "o", "u":
    print("\(someCharacter) is a vowel")
default:
    print("\(someCharacter) is not a vowel")
}
```

---
* break is not needed between statements, cases do not fallthrough by default

__Not Preferred__

```
switch someCountryEnum {
case .india:
    print("New Delhi")
    break
case .usa:
    print("Washington DC")
    break
case .china:
    print("Beijing")
    break
default:
    print("Not Applicable")
    break
}
```

__Preferred__

```
switch someCountryEnum {
case .india:
    print("New Delhi")
case .usa:
    print("Washington DC")
case .china:
    print("Beijing")
default:
    print("Not Applicable")
}
```

### 4. Closures ###

* Use shorthand argument syntax for simple one-line closure implementations

__Not Preferred__

```
func transform(action: (String) -> String) {
}

transform { (input: String) -> String in
    return input.reverse()
}
```

__Preferred__

```
func transform(action: (String) -> String) {
}

transform { $0.reverse() }
```
---
* Use capture lists to resolve strong reference cycles in closures. We typically want [weak self] here to avoid the strong capturing of self inside the block, as it is less likely to lead to retain cycles that cause memory leaks

__Not Preferred__

```
UserAPI.registerUser(user) { result in
    if result.success {
        self.doSomethingWithResult(result)
    }
}
```

__Preferred__

```
UserAPI.registerUser(user) { [weak self] result in
    if result.success {
        self?.doSomethingWithResult(result)
    }
}
```

### 5. Protocols ###

* When adding protocol conformance to a class, prefer adding a separate class extension for the protocol methods.

__Not Preferred__

```
class MyViewcontroller: UIViewController, UITableViewDataSource, UIScrollViewDelegate {
  // class stuff here
  // table view data source methods here
  // scroll view delegate methods here
}
```

__Preferred__

```
class MyViewcontroller: UIViewController {
  // class stuff here
}

// MARK: - UITableViewDataSource
extension MyViewcontroller: UITableViewDataSource {
  // table view data source methods here
}

// MARK: - UIScrollViewDelegate
extension MyViewcontroller: UIScrollViewDelegate {
  // scroll view delegate methods here
}
```
---
* Use weak optional vars for delegate variables to avoid retain cycles

__Not Preferred__

```
//SomeTableCell.swift

protocol SomeTableCellDelegate: class {
    func cellButtonWasTapped(cell: SomeTableCell)
}

class SomeTableCell: UITableViewCell {
    var delegate: SomeTableCellDelegate? // Creates retain cycle
}
```

__Preferred__

```
//SomeTableCell.swift

protocol SomeTableCellDelegate: class {
    func cellButtonWasTapped(cell: SomeTableCell)
}

class SomeTableCell: UITableViewCell {
    weak var delegate: SomeTableCellDelegate?
}
```

### 6. Higher order functions and loops ###

* Use higher order functions like `map()`, `reduce()`, `filter()`, `flatmap()`, etc instead of writing same operation yourself.

__Not Preferred__

```
let evenNumbers = []
let allNumbers = [1,2,3,4,5,6]
for number in allNumbers {
    if number % 2 == 0 {
        allNumbers.append(number)
    }
}
print(evenNumbers)
```

__Preferred__

```
let allNumbers = [1,2,3,4,5,6]
let evenNumbers = allNumbers.filter { $0 % 2 == 0 }
print(evenNumbers)
```
---
* Use the `enumerated()` function if you need to loop over a Sequence and use both index and element in the loop.

__Not Preferred__

```
for index in 0..<someArray.count {
    print(index)
    print(someArray[index])
}
```

__Preferred__

```
for (index, element) in someArray.enumerated() {
    print(index)
    print(element)
}
```
---
* When the entirety of a for loop???s body would be a single if block testing a condition of the element, the test is placed in the where clause of the for statement instead.

__Not Preferred__

```
for item in collection {
  if item.hasProperty {
    // ...
  }
}
```

__Preferred__

```
for item in collection where item.hasProperty {
  // ...
}
```

### 7. Patterns ###

* Extract complex property observers into methods

__Not Preferred__

```
class TextField {
  var text: String? {
    didSet {
      guard oldValue != text else {
        return
      }
      // Do a bunch of operations (API, analytics)
    }
  }
}
```

__Preferred__

```
class TextField {
  var text: String? {
    didSet { textDidUpdate(from: oldValue) }
  }

  private func textDidUpdate(from oldValue: String?) {
    guard oldValue != text else {
      return
    }
    // Do a bunch of operations (API, analytics)
    // All in meaningful methods
  }
}
```

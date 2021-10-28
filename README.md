# Swift-Style-Guide

## Goals ##

The document enlist code snippets focusing on common mistakes and their suggested solution.

### 1. Optionals ###

* The only time you should be using implicitly unwrapped optionals is with @IBOutlets. In every other case, it is better to use a non-optional or regular optional property. Yes, there are cases in which you can probably "guarantee" that the property will never be nil when used, but it is better to be safe and consistent. Similarly, don't use force unwraps (`!`, `as!`, `try!`, etc)

_Not Preferred_

```
let name: String!
let number = text as! Int
```

_Preferred_

```
let name: String?
let number = text as? Int
```

* When unwrapping optionals, `if let` is better suited for situations where you still need to move forward after failing. For other cases, use `guard let` to improve readability by focusing on the happy-path and can also reduce nesting by keeping your code flatter.

_Not Preferred_

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

_Preferred_

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


_Not Preferred_

```
var apiKeyConstant = "asdqwe123"

// Instantiates everytime
var locationManager = makeLocationManager()

```

_Preferred_

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

_Not Preferred_

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

_Preferred_

```
switch someCharacter {
case "a", "e", "i", "o", "u":
    print("\(someCharacter) is a vowel")
default:
    print("\(someCharacter) is not a vowel")
}
```


* break is not needed between statements, cases do not fallthrough by default

_Not Preferred_

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

_Preferred_

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

_Not Preferred_

```
func transform(action: (String) -> String) {
}

transform { (input: String) -> String in
    return input.reverse()
}
```

_Preferred_

```
func transform(action: (String) -> String) {
}

transform { $0.reverse() }
```

* Use capture lists to resolve strong reference cycles in closures. We typically want [weak self] here to avoid the strong capturing of self inside the block, as it is less likely to lead to retain cycles that cause memory leaks

_Not Preferred_

```
UserAPI.registerUser(user) { result in
    if result.success {
        self.doSomethingWithResult(result)
    }
}
```

_Preferred_

```
UserAPI.registerUser(user) { [weak self] result in
    if result.success {
        self?.doSomethingWithResult(result)
    }
}
```

### 5. Protocols ###

* When adding protocol conformance to a class, prefer adding a separate class extension for the protocol methods.

_Not Preferred_

```
class MyViewcontroller: UIViewController, UITableViewDataSource, UIScrollViewDelegate {
  // class stuff here
  // table view data source methods here
  // scroll view delegate methods here
}
```

_Preferred_

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

* Use weak optional vars for delegate variables to avoid retain cycles

_Not Preferred_

```
//SomeTableCell.swift

protocol SomeTableCellDelegate: class {
    func cellButtonWasTapped(cell: SomeTableCell)
}

class SomeTableCell: UITableViewCell {
    var delegate: SomeTableCellDelegate? // Creates retain cycle
}
```

_Preferred_

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

_Not Preferred_

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

_Preferred_

```
let allNumbers = [1,2,3,4,5,6]
let evenNumbers = allNumbers.filter { $0 % 2 == 0 }
print(evenNumbers)
```

* Use the `enumerated()` function if you need to loop over a Sequence and use both index and element in the loop.

_Not Preferred_

```
for index in 0..<someArray.count {
    print(index)
    print(someArray[index])
}
```

_Preferred_

```
for (index, element) in someArray.enumerated() {
    print(index)
    print(element)
}
```

* When the entirety of a for loop’s body would be a single if block testing a condition of the element, the test is placed in the where clause of the for statement instead.

_Not Preferred_

```
for item in collection {
  if item.hasProperty {
    // ...
  }
}
```

_Preferred_

```
for item in collection where item.hasProperty {
  // ...
}
```

### 7. Patterns ###

* Extract complex property observers into methods

_Not Preferred_

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

_Preferred_

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

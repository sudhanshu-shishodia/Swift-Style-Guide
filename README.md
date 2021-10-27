# Swift-Style-Guide

## Goals ##

Following this style guide should:

* Make it easier to read and begin understanding unfamiliar code.
* Make code easier to maintain.
* Reduce simple programmer errors.

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

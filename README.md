# A guide to our Swift style and conventions.

This is an attempt to encourage patterns of coding that help accomplish the following goals (in
a rough priority order):

1. increased rigor and decreased likelihood of programmer error
2. increased clarity of intent
3. reduced verbosity
4. fewer repetitive debates about aesthetics

## One rule to rule them all ---> **KISS** <---

* Don't be a smartass.
* Read, understand, and apply [KISS](https://en.wikipedia.org/wiki/KISS_principle).

***Trivia:*** The phrase was coined by the lead engineer of SR-71 Blackbird.

## Table of Contents
1. [Whitespace](#whitespace)
2. [Code organization](#code-organization)
3. [Code segmentation](#code-segmentation)
4. [Minimal Imports](#minimal-imports)
5. [Naming](#naming)
6. [Constants](#constants)
7. [Comments](#comments)
8. [Language](#language)
9. [Closures](#closures)
10. [Closure expressions](#closure-expressions)
11. [Computed Properties](#computed-properties)
12. [Delegates](#delegates)
13. [Functions](#functions)
14. [Prefer `let`-bindings over `var`-bindings wherever possible](#prefer-let-bindings-over-var-bindings-wherever-possible)
15. [Return and break early](#return-and-break-early)
16. [Optionals](#optionals)
17. [Always specify access control explicitly for top-level definitions](#always-specify-access-control-explicitly-for-top-level-definitions)
18. [Style](#style)
19. [Patterns](#patterns)
20. [Prefer structs over classes](#prefer-structs-over-classes)
21. [Make classes `final` by default](#make-classes-final-by-default)
22. [Protocol Conformance](#protocol-conformance)
23. [Types](#types)
24. [Use Type Inferred Context](#use-type-inferred-context)
25. [Operators](#operators)
26. [Lazy Initialization](#lazy-initialization)
27. [Forbidden](#forbidden)

## Whitespace

* Tabs, not spaces.
* End files with a new line.
* Make liberal use of vertical whitespace to divide the code into logical chunks.
* Don’t leave trailing whitespace. Xcode has an option for this in `Preferences -> Automatically trim trailing whitespace and Including whitespace-only lines`.
* Not even leading indentation on blank lines.

## Code organization

* Use extensions to organize your code into logical blocks of functionality.
* Each extension should be set off with a `// MARK: - Code chunk` comment to keep things well-organized

```swift
// 1. Imports

import UIKit

// 2. Clases, structs, enums

class Person {

  // 2.1. IBOutles
  @IBOutlet private weak var closeButton: UIButton!
  @IBOutlet private weak var loginButton: UIButton!
  @IBOutlet private weak var titleLabel: UILabel!
  
  // 2.2 public, internal properties
  let name: String
  private(set) var oib: Int

  // 2.3. private properties
  unowned private let bankCard: BankCard

  // 2.4. Initializers
  init(name: String, bankCard: BankCard)
}

// 2.5. Public, internal functions
public extension Person {
    func hasValidBankCard() -> Bool
}

// 2.6. Private functions
private extension Person {
    func setupPerson() -> Void
}

// 3. extensions, protocol implementations
extension Person: CustomStringConvertible {

  var description: String {
    return "\(name) has \(bankCard.name) card."
  }
  
  var age: String {
    "\(name) is \(age) years old."
  }
}
```

Segments `2.5.` and  `2.6` should be grouped into meaningful extension blocks which can then be private or public.

## Code segmentation

* Use `MARK:`, aka `PRAGMA MARK:`.  (`MARK: Outlets`, `MARK: - Outlets`, `MARK: - Outlets -`)
* Add one empty row after `MARK`.

## Minimal Imports

* Import only the modules a source file requires
* For example, don't import UIKit when importing Foundation will suffice. Likewise, don't import Foundation if you must import UIKit.

## Naming

* __Be mindful about grammar!__
* Names should be meaningful and compact.
* Use `UpperCamelCase` for types and protocols, `lowerCamelCase` for everything else
* Include all needed words while omitting needless ones
* Use names based on roles, not types
* Strive for fluent usage
* Begin factory methods with make
* Name methods for their side effects
* Verb methods follow the -ed, -ing rule for the non-mutating version
* Noun methods follow the formX rule for the mutating version
* Boolean types should read like assertions
* Protocols that describe what something is should read as nouns
* Protocols that describe a capability should end in -able or -ible
* Use terms that don't surprise experts or confuse beginners
* Generally avoid abbreviations
* Prefer methods and properties to free functions
* Case acronyms and initialisms uniformly up or down
* Give the same base name to methods that share the same meaning
* Avoid overloads on return type
* Choose good parameter names that serve as documentation
* Prefer to name the first parameter instead of including its name in the method name, except as mentioned under Delegates
* Label closure and tuple parameters
* Take advantage of default parameters
* Name booleans like isSpaceship, hasSpacesuit, etc. to it clear that they are booleans and not other types
* Names should be written with their most general part first and their most specific part last
* Include a hint about type in a name if it would otherwise be ambiguous
* Avoid Objective-C-style acronym prefixes
* Event-handling functions should be named like past-tense sentences

## Constants

 * Constants are defined using the `let` keyword and variables with the `var` keyword
 * Always use `let` instead of `var` if the value of the variable will not change
 * You can define constants on a type rather than on an instance of that type using type properties
 * Type properties declared in this way are generally preferred over global constants because they are easier to distinguish from instance properties
 * To declare a type property as a constant simply use `static let`

### Preferred:

```swift
enum Math {
  static let e = 2.718281828459045235360287
  static let root2 = 1.41421356237309504880168872
}

let hypotenuse = side * Math.root2
```

### Not Preferred:

```swift
let e = 2.718281828459045235360287  // pollutes global namespace
let root2 = 1.41421356237309504880168872

let hypotenuse = side * root2 // what is root2?
```

## Comments

* Use comments to explain why a particular piece of code does something
* Comments must be kept up-to-date or deleted
* Explain the why, not how
* Avoid the use of C-style comments (/* ... */)
* Prefer the use of double- or triple-slash
* Use comments when the code is not self-explanatory:
    * Rx
    * API calls
    * dark magic

## Language

### Preferred:

```swift
let color = "red"
```

### Not Preferred:

```swift
let colour = "red"
```

_Rationale:_ Use US English spelling to match Apple's API.

## Closures
* If the last argument of the function is a closure, use trailing closure syntax only when it makes sense and it's clear what the closure is doing
* If it's the only argument, parentheses may be omitted
* Unused closure arguments should be replaced with _ (or fully omitted if no arguments are used)
* In most cases, argument types should be inferred
* If **capturing self**,  use `weak` and  `unowned` appropriately.
* Give the closure parameters descriptive names

## Closure expressions

### Preferred

```swift
UIView.animate(withDuration: 0.3) {
    self.myView.alpha = 0
}

UIView.animate(
    withDuration: 0.3,
    animations: {
        self.myView.alpha = 0
    },
    completion: { _ in
        self.myView.removeFromSuperview()
    }
)
```

### Not Preferred:

```swift
UIView.animate(withDuration: 0.3, animations: {
    self.myView.alpha = 0
})

UIView.animate(withDuration: 0.3, animations: {
    self.myView.alpha = 0
}) { (finished) in
    self.myView.removeFromSuperview()
}
```

* For single-expression closures where the context is clear, use implicit returns

```swift
    attendeeList.sort { a, b in
      a > b
    }
```

* Chained methods using trailing closures should be clear and easy to read in context.

### Preferred:

```swift
  let value1 = numbers
    .map { $0 * 2 }
    .filter { $0 > 50 }
    .map { $0 + 10 }
```

### Not Preferred:

```swift
let value = numbers.map { $0 * 2 }.filter { $0.isMultiple(of: 3) }.map { $0 + 10 }
```

* Name unused closure parameters as underscores (`_`)
* Single-line closures should have a space inside each brace

## Computed Properties

* For conciseness, if a computed property is read-only, omit the get clause
* The get clause is required only when a set clause is provided

### Preferred:

```swift      
var diameter: Double {
  return radius * 2
}

var area: Double {
    radius * radius * pi
}
```

### Not Preferred:

```swift      
var diameter: Double {
  get {
    return radius * 2
  }
}    
```

## Delegates
* When creating custom delegate methods, an unnamed first parameter should be the delegate source (UIKit contains numerous examples of this.)

### Preferred:

```swift
func namePickerView(_ namePickerView: NamePickerView, didSelectName name: String)
func namePickerViewShouldReload(_ namePickerView: NamePickerView) -> Bool
```

### Not Preferred:

```swift
func didSelectName(namePicker: NamePickerViewController, name: String)
func namePickerShouldReload() -> Bool
```

## Functions

Keep short function declarations on one line including the opening brace

```swift
func reticulateSplines(spline: [Double]) -> Bool {
  // reticulate code goes here
}
```

For functions with long signatures, put each parameter on a new line and add an extra indent on subsequent lines

```swift 
func reticulateSplines(
    spline: [Double], 
    adjustmentFactor: Double,
    translateConstant: Int, 
    comment: String
) -> Bool {
    // reticulate code goes here
}
```

* Don't use (Void) to represent the lack of an input; simply use (). Use Void instead of () for closure and function outputs

### Preferred:

```swift
func updateConstraints() {
  // magic happens here
}

typealias CompletionHandler = (result) -> Void
```

### Not Preferred:

```swift        
func updateConstraints() -> () {
  // magic happens here
}

func updateConstraints() -> Void {
  // magic happens here
}

typealias CompletionHandler = (result) -> ()
typealias CompletionHandler = (result) -> Void
```

* Omit Void return types from function definitions

### Preferred:

```swift
func doSomething() {
  ...
}
```

### Not Preferred:

```swift
func doSomething() -> Void {
  ...
}
```

* Mirror the style of function declarations at call sites. Calls that fit on a single line should be written as such:

```swift
let success = reticulateSplines(splines)
```

* Long function invocations should also break on each argument if there is more than 3 arguments. Put the closing parenthesis on the last line of the invocation

### Preferred:

```swift
universe.generateStars(
  at: location,
  count: 5,
  color: starColor,
  withAverageDistance: 4
)
```

### Not Preferred:

```swift
universe.generateStars(at: location, count: 5, color: starColor, withAverageDistance: 4)

universe.generateStars(
    at: location,
 count: 5,
 color: starColor,
 withAverageDistance: 4
radimuin)

universe.generate(5,
  .stars,
  at: location)

universe.generateStars(
  at: location,
  count: 5,
  color: starColor,
  withAverageDistance: 4)

universe.generate(
  5,
  .stars,
  at: location)
```

* Separate long function declarations with line breaks before each argument label and before the return signature
* Put the open curly brace on the next line so the first executable line doesn't look like it's another parameter


### Preferred:

```swift
class Universe { }

    func generateStars(
      at location: Point,
      count: Int,
      color: StarColor,
      withAverageDistance averageDistance: Float
    ) -> String {
      populateUniverse()
    }

    func generateStars(
      at location: Point,
      count: Int,
      color: StarColor,
      withAverageDistance averageDistance: Float
    ) throws -> String {
      populateUniverse()
    }
}
```

### Not Preferred:

```swift
class Universe {

  func generateStars(at location: Point, count: Int, color: StarColor, withAverageDistance averageDistance: Float) -> String {
    // This is too long and will probably auto-wrap in a weird way
  }

  func generateStars(at location: Point,
                     count: Int,
                     color: StarColor,
                     withAverageDistance averageDistance: Float) -> String
  {
    // Xcode indents all the arguments
  }

  func generateStars(
    at location: Point,
    count: Int,
    color: StarColor,
    withAverageDistance averageDistance: Float) -> String {
    populateUniverse() // this line blends in with the argument list
  }

  func generateStars(
    at location: Point,
    count: Int,
    color: StarColor,
    withAverageDistance averageDistance: Float) throws
    -> String {
    populateUniverse() // this line blends in with the argument list
  }
}
```

## Prefer `let`-bindings over `var`-bindings wherever possible

Use `let foo = …` over `var foo = …` wherever possible (and when in doubt). Use `var` only if you absolutely have to (i.e., you *know* that the value might change, for example, when using the `weak` storage modifier).

_Rationale:_ The intent and meaning of both keywords is clear, but *let-by-default* results in safer and clearer code.

A `let`-binding guarantees and *clearly signals to the programmer* that its value will never change. Subsequent code can thus make stronger assumptions about its usage.

It becomes easier to reason about code. Had you used `var` while still making the assumption that the value never changed, you would have to manually check that.

Accordingly, whenever you see a `var` identifier being used, assume that it will change and ask yourself why.

Prefer immutable values whenever possible. Use map and compactMap instead of appending to a new collection. Use filter instead of removing elements from a mutable collection.

### Preferred:

```swift
let results = input.map(transform)

let results = input.compactMap(transformThatReturnsAnOptional)
```
### Not Preferred:

```swift
var results = [SomeType]()
for element in input {
  let result = transform(element)
  results.append(result)
}

var results = [SomeType]()
for element in input {
  if let result = transformThatReturnsAnOptional(element) {
    results.append(result)
  }
}
```

## Return and break early

When you have to meet certain criteria to continue execution, try to exit early. So, instead of this:

```swift
if n.isNumber {
    // Use n here
} else {
    return
}
```
use this:

```swift
guard n.isNumber else {
    return
}
// Use n here
```

You can also do it with an `if` statement, but using `guard` is preferred because a `guard` statement without `return`, `break`, or `continue` produces a compile-time error, so exit is guaranteed.
The guard statement avoids an unnecessary block compared to an if and avoids an indentation level, which is especially nice for early returns.

```swift
for element in input {
  guard let result = transformThatReturnsAnOptional(element) else { continue }
  results.append(result)
}
```

## Optionals

* Declare variables and function return types as optional with ? where a nil value is acceptable
* If you have a `foo` identifier of the `FooType?` or `FooType!` type, don't force unwrap it to get to the underlying value (`foo!`) if possible. Instead, do this:

```swift
if let foo = foo {
    // Use unwrapped `foo` value in here
} else {
    // If appropriate, handle the case where the optional is nil
}

// or this

guard let foo = foo else {
    return
}

// Use unwrapped `foo` value in here
```

* Alternatively, you might want to use Swift's Optional Chaining in some of these cases, such as:

```swift
// Call the function if `foo` is not nil. If `foo` is nil, forget we ever tried to make the call.
foo?.callSomethingIfFooIsNotNil()

// Call the map function only if `foo` is not nil. In this case, `result` is also optional but it is another type.
let result = foo.flatMap(mapToResult)
```

_Rationale:_ Explicit `if let`-binding of Optionals results in safer code. Force unwrapping is more prone to lead to runtime crashes.

* Where possible, use `let foo: FooType?` instead of `let foo: FooType!` if `foo` may be nil (note that, in general, `?` can be used instead of `!`).

_Rationale:_ Explicit optionals result in safer code. Implicitly unwrapped optionals have the potential of crashing at runtime.

* When naming optional variables and properties, avoid naming them like `optionalString` or `maybeView` since their optional-ness is already in the type declaration
* For optional binding, shadow the original name whenever possible rather than using names like `unwrappedView` or `actualLabel`

## Always specify access control explicitly for top-level definitions

Access control should be at the strictest level possible. Prefer public to open and private to fileprivate unless you need that behavior.
Top-level functions, types, and variables should always have explicit access control specifiers:

```swift
public var whoopsGlobalState: Int
internal struct TheFez {}
private func doTheThings(things: [Thing]) {}
```

However, the definitions within those can leave access control implicit, where appropriate:

```swift
internal struct TheFez {
  var owner: Person = Joshaber()
}
```

_Rationale:_ It's rarely appropriate for top-level definitions to be specifically `internal`, and being explicit ensures that careful thought is put into that decision. Within a definition, reusing the same access control specifier is just duplicative, and the default is usually reasonable.

##  `self`

* When accessing properties or methods on `self`, leave the reference to `self` implicit by default:

```swift
private class History {
  var events: [Event]

  func rewrite() {
    events = []
  }
}
```

* Include the explicit keyword only when required by the language—for example, in a closure, or when the parameter names conflict:

```swift
extension History {
  init(events: [Event]) {
    self.events = events
  }

  var whenVictorious: () -> Void {
    return {
      self.rewrite()
    }
  }
}
```

* Don't use `self` unless it's necessary for disambiguation or required by the language

### Preferred:

```swift
private func increaseCapacity(by amount: Int) {
    capacity += amount
    save()
}
```

### Not Preferred:

```swift
private func increaseCapacity(by amount: Int) {
    self.capacity += amount
    self.save()
}
```

* Bind to `self` when upgrading from a weak reference

### Preferred:

```swift
class MyClass {

  func request(completion: () -> Void) {
    API.request { [weak self] response in
      guard let self = self else { return }
      // Do work
      completion()
    }
  }
}
```

### Not Preferred:

```swift
class MyClass {

  func request(completion: () -> Void) {
    API.request { [weak self] response in
      guard let self = self else { return }
      // Do work
      completion()
    }
  }
}
```

## Style

* Name members of tuples for extra clarity

### Preferred:

```swift
func whatever() -> (x: Int, y: Int) {
  return (x: 4, y: 4)
}

let thing = whatever()
print(thing.0)
```

### Not Preferred:

```swift
func whatever() -> (Int, Int) {
  return (4, 4)
}

let thing = whatever()
print(thing.0)
```

* Place the colon immediately after an identifier, followed by a space

### Preferred:

```swift
var something: Double = 0

class MyClass: SuperClass {
  // ...
}

var dict = [KeyType: ValueType]()
```

### Not Preferred:

```swift
var something : Double = 0

class MyClass : SuperClass {
  // ...
}

var dict = [KeyType:ValueType]()
var dict = [KeyType : ValueType]()
```

* Place a space on either side of a return arrow for readability

### Preferred:

```swift
func doSomething() -> String {
  // ...
}
```

### Not Preferred:

```swift
func doSomething()->String {
  // ...
}
```


* Omit unnecessary parentheses

### Preferred:

```swift
if userCount > 0 { ... }
switch someValue { ... }
let evens = userCounts.filter { number in number % 2 == 0 }
let squares = userCounts.map { $0 * $0 }
```

### Not Preferred:

```swift
if (userCount > 0) { ... }
switch (someValue) { ... }
let evens = userCounts.filter { (number) in number % 2 == 0 }
let squares = userCounts.map() { $0 * $0 }
```

* Omit enum associated values from case statements when all arguments are unlabelled


### Preferred:

```swift
if case .done = result { ... }

switch animal {
case .dog:
  ...
}
```

### Not Preferred:

```swift
if case .done(_) = result { ... }

switch animal {
case .dog(_, _, _):
  ...
}
```
    
* Omit type parameters where possible

### Preferred:

```swift
struct Composite<T> {
  …
  func compose(other: Composite) -> Composite {
    return Composite(self, other)
  }
}
```

### Not Preferred:

```swift
struct Composite<T> {
  …
  func compose(other: Composite<T>) -> Composite<T> {
    return Composite<T>(self, other)
  }
}
```

_Rationale:_ Omitting redundant type parameters clarifies the intent, and makes it obvious by contrast when the returned type takes different type parameters.

* Use whitespace around operator definitions

### Preferred:

```swift
func <| (lhs: Int, rhs: Int) -> Int
func <|< <A>(lhs: A, rhs: A) -> A
```

### Not Preferred:

```swift
func <|(lhs: Int, rhs: Int) -> Int
func <|<<A>(lhs: A, rhs: A) -> A
```

_Rationale:_ Operators consist of punctuation characters, which can make them difficult to read when immediately followed by punctuation for a type or a value parameter list. Adding whitespace separates the two more clearly.

* Place function/type attributes on the line above the declaration


### Preferred:

```swift
@objc
class Spaceship {

  @discardableResult
  func fly() -> Bool {
  }
}
```

### Not Preferred:

```swift
@objc class Spaceship {

  @discardableResult func fly() -> Bool {
  }
}
```

* Multi-line arrays should have each bracket on a separate line

### Preferred:

```swift
let rowContent = [
  listingUrgencyDatesRowContent(),
  listingUrgencyBookedRowContent(),
  listingUrgencyBookedShortRowContent()
]
```

### Not Preferred:

```swift
let rowContent = [listingUrgencyDatesRowContent(),
                  listingUrgencyBookedRowContent(),
                  listingUrgencyBookedShortRowContent()]
```

* Multi-line conditional statements should break after the leading keyword

### Preferred:

```swift
guard let galaxy = galaxy,
      galaxy.name == "Milky Way" // Variable width indentation (6 spaces)
else { … }

if
  let galaxy = galaxy,
  galaxy.name == "Milky Way"
{ … }

guard
  let galaxy = galaxy,
  galaxy.name == "Milky Way"
else { … }

guard
  let earth = unvierse.find(
    .planet,
    named: "Earth"),
  earth.isHabitable
else { … }

if let galaxy = galaxy {
  …
}

guard let galaxy = galaxy else {
  …
}
```

### Not Preferred:

```swift
if let galaxy = galaxy,
  galaxy.name == "Milky Way" // Indenting by two spaces fights Xcode's ^+I indentation behavior
{ … }

guard let earth = unvierse.find(
  .planet,
  named: "Earth"),
  earth.isHabitable // Blends in with previous condition's method arguments
else { … }
```

* Use constructors instead of  `Make()` functions for NSRange and others

### Preferred:

```swift
let range = NSRange(location: 10, length: 5)
```

### Not Preferred:

```swift
let range = NSMakeRange(10, 5)
```

For standard library types with a canonical shorthand form (Optional, Array, Dictionary), prefer using the shorthand form over the full generic form

### Preferred:

```swift
let optional: String? = nil
let array: [String] = []
let dictionary: [String: Any] = [:]
```

### Not Preferred:

```swift
let optional: Optional<String> = nil
let array: Array<String> = []
let dictionary: Dictionary<String, Any> = [:]
```

* Prefer using guard at the beginning of a scope (see example at [Return and Break Early](#return-and-break-early))
* Prefer methods within type definitions instead of global functions

## Patterns

* Prefer initializing properties at init time whenever possible, rather than using implicitly unwrapped optionals

### Preferred:

```swift
class MyClass {

  var someValue: Int

  init() {
    someValue = 0
    super.init()
  }
}
```

### Not Preferred:

```swift
class MyClass {
  
  var someValue: Int!

  init() {
    super.init()
    someValue = 5
  }
}
```

* Avoid performing any meaningful or time-intensive work in init()
* Extract complex property observers into methods

### Preferred:

```swift
class TextField {
  var text: String? {
    didSet { textDidUpdate(from: oldValue) }
  }

  private func textDidUpdate(from oldValue: String?) {
    guard oldValue != text else {
      return
    }

    // Do a bunch of text-related side-effects.
  }
}
```

### Not Preferred:

```swift
class TextField {
  var text: String? {
    didSet {
      guard oldValue != text else {
        return
      }

      // Do a bunch of text-related side-effects.
    }
  }
}
```

_Rationale_: This reduces nestedness, separates side-effects from property declarations, and makes the usage of implicitly-passed parameters like oldValue explicit.

* Extract complex callback blocks into methods 
* If you need to reference self in the method call, make use of guard to unwrap self for the duration of the callback if you are using `weak`.

### Preferred:

```swift
class MyClass {

  func request(completion: () -> Void) {
    API.request { [weak self] response in
      guard let self = self else { return }
      self.doSomething(with: self.property, response: response)
      completion()
    }
  }

  func doSomething(with nonOptionalParameter: SomeClass, response: SomeResponseClass) {
    // Processing and side effects
  }
}
```

### Not Preferred:

```swift
class MyClass {

  func request(completion: () -> Void) {
    API.request { [weak self] response in
      if let self = self {
        // Processing and side effects
      }
      completion()
    }
  }
}
```

_Rationale_: This limits the complexity introduced by weak-self in blocks and reduces nestedness. 

## Prefer structs over classes

Unless you require functionality that can be provided only by a class (like identity or deinitializers), implement a struct instead.

Note that inheritance is (by itself) usually _not_ a good reason to use classes because polymorphism can be provided by protocols, and implementation reuse can be provided through composition.

For example, this class hierarchy:

```swift
class Vehicle {
    let numberOfWheels: Int

    init(numberOfWheels: Int) {
        self.numberOfWheels = numberOfWheels
    }

    func maximumTotalTirePressure(pressurePerWheel: Float) -> Float {
        return pressurePerWheel * Float(numberOfWheels)
    }
}

class Bicycle: Vehicle {
    init() {
        super.init(numberOfWheels: 2)
    }
}

class Car: Vehicle {
    init() {
        super.init(numberOfWheels: 4)
    }
}
```

could be refactored into these definitions:

```swift
protocol Vehicle {
    var numberOfWheels: Int { get }
}

func maximumTotalTirePressure(vehicle: Vehicle, pressurePerWheel: Float) -> Float {
    return pressurePerWheel * Float(vehicle.numberOfWheels)
}

struct Bicycle: Vehicle {
    let numberOfWheels = 2
}

struct Car: Vehicle {
    let numberOfWheels = 4
}
```

_Rationale:_ Value types are simpler, easier to reason about, and they behave as expected with the `let` keyword.

## Make classes `final` by default

Classes should start as `final` and only be changed to allow subclassing if a valid need for inheritance has been identified. Even in that case, as many definitions as possible _within_ the class should be `final` as well, following the same rules.

_Rationale:_ Composition is usually preferable to inheritance, and opting _in_ to inheritance hopefully means that more thought will be put into the decision.

## Protocol Conformance

In particular, when adding protocol conformance to a model, prefer adding a separate extension for the protocol methods
_Rationale_: This keeps the related methods grouped together with the protocol and can simplify instructions to add a protocol to a class with its associated methods

### Preferred:

```swift
class MyViewController: UIViewController {
  // class stuff here
}

// MARK: - UITableViewDataSource -

extension MyViewController: UITableViewDataSource {
  // table view data source methods
}

// MARK: - UIScrollViewDelegate -

extension MyViewController: UIScrollViewDelegate {
  // scroll view delegate methods
}
```

### Not Preferred:

```swift
class MyViewController: UIViewController, UITableViewDataSource, UIScrollViewDelegate {
  // all methods
}
```

## Types

* Always use Swift's native types and expressions when available

### Preferred:

```swift
let width = 120.0                                    // Double
let widthString = "\(width)"                         // String
```

### Less Preferred:

```swift
let width = 120.0                                    // Double
let widthString = (width as NSNumber).stringValue    // String
```

### Not Preferred:

```swift
let width: NSNumber = 120.0                          // NSNumber
let widthString: NSString = width.stringValue        // NSString
```

* In drawing code, use CGFloat if it makes the code more succinct by avoiding too many conversions

## Use Type Inferred Context

* Use compiler inferred context to write shorter, clear code.

### Preferred:

```swift
let selector = #selector(viewDidLoad)
view.backgroundColor = .red
let toView = context.view(forKey: .to)
let view = UIView(frame: .zero)
```
### Not Preferred:

```swift
let selector = #selector(ViewController.viewDidLoad)
view.backgroundColor = UIColor.red
let toView = context.view(forKey: UITransitionContextViewKey.to)
let view = UIView(frame: CGRect.zero)
```

## Operators

* Infix operators should have a single space on either side
* Prefer parenthesis to visually group statements with many operators rather than varying widths of whitespace

### Preferred:

```swift
let capacity = 1 + 2
let capacity = currentCapacity ?? 0
let mask = (UIAccessibilityTraitButton | UIAccessibilityTraitSelected)
let capacity = newCapacity
let latitude = region.center.latitude - (region.span.latitudeDelta / 2.0)
```

### Not Preferred:

```swift
let capacity = 1+2
let capacity = currentCapacity   ?? 0
let mask = (UIAccessibilityTraitButton|UIAccessibilityTraitSelected)
let capacity=newCapacity
let latitude = region.center.latitude - region.span.latitudeDelta/2.0
```


### Ternary Operator

* The Ternary operator, ?: , should only be used when it increases clarity or code neatness 
* Single condition is usually all that should be evaluated. 
* Evaluating multiple conditions is usually more understandable as an if statement or refactored into instance variables.
* In general, the best use of the ternary operator is during assignment of a variable and deciding which value to use.

### Preferred:

```swift
let value = 5
result = value != 0 ? x : y

let isHorizontal = true
result = isHorizontal ? x : y
```

### Not Preferred:

```swift
result = a > b ? x = c > d ? c : d : y    
```

## Lazy Initialization

 * Consider using lazy initialization for finer grained control over object lifetime
 * You can either use a closure that is immediately called { }() or call a private factory method
 
```swift
lazy var locationManager = makeLocationManager()

private func makeLocationManager() -> CLLocationManager {
  let manager = CLLocationManager()
  manager.desiredAccuracy = kCLLocationAccuracyBest
  manager.delegate = self
  manager.requestAlwaysAuthorization()
  return manager
}
```

## Forbidden

* Types should never have prefixes because their names are already implicitly mangled and prefixed by their module name.
* Semicolons are obfuscative and should never be used.
* Unused (dead) code, including Xcode template code and placeholder comments should be removed.
* Don't use `#imageLiteral` or `#colorLiteral`, use Redbreast.

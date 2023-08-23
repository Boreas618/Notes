# Cheatsheet 

Initializer syntax

```swift
let emptyArray:[String]=[]
let emptyDictionary:[String:Float]=[:]
```

In an `if` statement, the conditional must be a Boolean expression—this means that code such as `if score { ... }` is an error, not an implicit comparison to zero.

Use `if` and `let` together to work with values that might be missing.

```swift
var optionalName: String? = "Sun Yi"
var greeting:String = "Hello!"
var nickname:String = "Sun Xiaochuan"
if let name = optionalName{
  greeting = "Hello, \(name)"
}

if let nickname{
  print("Hello, \(nickname)")
}
```

`..<` omits the upper value, while `..` includes both values.

```swift
func greet(name:String,day:String)->{
  return "Hello,\(name). Today is \(day)"
}
greet(person:"Bob",day:"Tuesday")
```

A tuple can be returned by a function.

```swift
return (min,max,sum)
```

A function can be nested in another function. A function can be an argument for another function. Expressed like `(Int) -> Bool`.

A closure without a name:

```swift
numbers.map({(number:Int)->Int in
            let result = 3*number
            return result
})
```

```swift
let sortedNumbers = numbers.sorted { $0 > $1 }
print(sortedNumbers)
```

Class

```swift
class Square: NamedShape{
  var sideLength: Double
  
  init(sideLength:Double,name:String){
    self.sideLength = sideLength
    super.init(name:name)
    numberOfSides = 3//The property is from super class
  }
}
```

Getter/Setter

```swift
var perimeter:Double{
  get{
    return 3.0 * sideLength
  }
  set{
    sideLength = newValue/3.0
  }
}
```

The new value has a implicit value called `newValue`, and you can also provide an explicit name use parentheses after the `set`.

`willSet` and `didSet`: 

```swift
class Square{
  
  var width:Double{
    willSet{
      self.height = width
    }
  }
  
  var height:Double{
    willSet{
      self.width = height
    }
  }
  
  init(width:Double, height:Double){
    self.width = width
    self.height = height
  }
}
```

 Enumerations and Structures

```swift
enum Rank:Int{
  case ace = 1
  case two,three,four,five,six,seven,eight,nine,ten
  case jack,queen,king
  
  func simpleDescription()->String{
    switch self{
      case .ace:
      	return "ace"
      case .jack:
      	return "jack"
      case .queen:
      	return "queen"
      case .king:
      	return "king"
      default:
      	return String(self.rawValue)
    }
  }
}

//The initializer for enum: Rank(rawValue:)

if let convertedRank = Rank(rawValue:3){
  let threeDescription = convertedRank.simpleDescription()
}
```

Inside the switch, the type of self is already known. Therefore, you can use the abbreviated form.

`async let`:

```swift
func connectUser(to server: String) async{
  async let userID = fetchUserID(from:server)
  async let username = fetchUsername(from:server)
  
  let greeting = await "Hello \(username),user ID \(userID)"
  print(greeting)
}
```

Use `Task` to call asynchronous functions from synchronous code, without waiting for them to return.

```swift
Task {
	await connectUser(to: "primary")
}
```

Handle error:

`do{} catch error{}`

`try?`

You can use `defer` to write setup and cleanup code next to each other, even though they need to be executed at different times.

# Function

**Function Argument Labels and Parameter Names**

**Variadic Parameters**`func arithmeticMean(_ numbers:Double...)->Double{}`

**In-Out Parameters**：

If you want a function to modify a parameter’s value, and you want those changes to persist after the function call has ended, define that parameter as an *in-out parameter* instead.

Use an ampersand before a variable's name when you pass it as an argument to an in-out parameter.

```swift
func swapTwoInts(_ a: inout Int, _b: inout Int) {
  let temporaryA = a
  a = b
  b = temporaryA
}
```

**function type**

`(Int,Int) -> Int`

`() -> Void`

A different function with the same matching type can be assigned to the same variable, in the same way as the nonfunction types:

```swift
mathFunction = multiplyTwoInts
```

# Collection Types

**List:**

```swift
shoppingList[4..6] = ["Bananas","Apples"]
shoppingLIst.insert("Maple Syrup",at:0)
let mapleSyrup = shoppingList.remove(at:0)

for (index, value) in shoppingList.enumerated() {
    print("Item \(index + 1): \(value)")
}
```

**Set:**

```swift
var letters = Set<Character>()//Initializer syntax
var favouriteGenres:Set<String> = ["Rock","Classical","Hip hop"]

for genre in favouriteGenres.sorted(){
  print("\(genre)")
}
```

**Dictionary**:

To remove a key-value pair from the dictionary:

```swift
//subscript syntax
airports["APL"] = nil

//This method removes the key-value pair if it exists and returns the removed value, or returns nil if no value existed:

if let removedValue = airports.removeValue(forKey: "DUB") {
    print("The removed airport's name is \(removedValue).")
} else {
    print("The airports dictionary doesn't contain a value for DUB.")
}
```

Iterating:

```swift
for (airportCode, airportName) in airports {
    print("\(airportCode): \(airportName)")
}
// LHR: London Heathrow
// YYZ: Toronto Pearson
```

# Strings and Characters

<img src="https://docs.swift.org/swift-book/_images/multilineStringWhitespace_2x.png" style="zoom:50%;" />

## Special Characters in String Literals

 `\0` (null character)

 `\\` (backslash)

`\t` (horizontal tab)

 `\n` (line feed)

 `\r` (carriage return)

 `\"` (double quotation mark) 

 `\'` (single quotation mark)

------

An arbitrary Unicode scalar value, written as `\u{`*n*`}`, where *n* is a 1–8 digit hexadecimal number (Unicode is discussed in [Unicode](https://docs.swift.org/swift-book/LanguageGuide/StringsAndCharacters.html#ID293) below)

## Initializing an Empty String

```swift
//Initializer syntax
var str = String()

//empty string literal
var str1 = ""
```

**Strings Are Value Types.If you create a new `String` value, that `String` value is *copied* when it’s passed to a function or method, or when it’s assigned to a constant or variable.**

## Concatenating Strings

Good start? Bad start?

```swift
let badStart = """
one
two
"""
let end = """
three
"""
print(badStart + end)
// Prints two lines:
// one
// twothree

let goodStart = """
one
two

"""
print(goodStart + end)
// Prints three lines:
// one
// two
// three
```

## Extended Grapheme Clusters

Every instance of swift `Character` type represents an extended grapheme clusters. An extended grapheme cluster is a sequence of one or more Unicode scalars.

The length of an `NSString` is based on the number of 16-bit code units within the string’s UTF-16 representation and not the number of Unicode extended grapheme clusters within the string.

## Index

```swift
let greeting = "Guten Tag!"
greeting[greeting.startIndex]
// G
greeting[greeting.index(before: greeting.endIndex)]
// !
greeting[greeting.index(after: greeting.startIndex)]
// u
let index = greeting.index(greeting.startIndex, offsetBy: 7)
greeting[index]
// a
```

# Control flow

## Switch

* no empty cases, use `break`
* `fallthrough`

## Labeled Statements

It’s sometimes useful to be explicit about which loop or conditional statement you want a `break` statement to terminate. 

# Closures

## Closure Expressions

The sorted method:

```swift
func backward(_ s1: String, _ s2: String) -> Bool {
  return s1 > s2
}

var reversedNames = names.sorted(by: backward)
```

Closure Expression Syntax:

![截屏2022-09-13 上午12.00.32](https://tva1.sinaimg.cn/large/e6c9d24egy1h649l48nssj20i605adfy.jpg)

```swift
reversedNames = names.sorted(by: { (s1: String, s2: String) -> Bool in return s1 > s2 } )
```

Inferring Type From Context

```swift
reservedNames = names.sorted(by: s1, s2 in return s1 > s2)
```

Implicit Returns from Single-Expression Closures

```swift
reservedNames = names.sorted(by:{ s1, s2 in s1 > s2})
```

Shorthand Argument Names

```swift
reservedNames = names.sorted(by: {$0 > $1 })
```

Operator Methods

```swift
reservedNames = names.sorted(by: >)
```

## Trailing Closures

You write a trailing closure after the function call's parentheses, even though the trailing closure is still an argument to the function. 

```swift
reservedNames = names.sorted(){ $0 > $1 }
```

If a closure expression is provided as the function’s or method’s only argument and you provide that expression as a trailing closure, you don’t need to write a pair of parentheses `()` after the function or method’s name when you call the function:

```swift
reservedNames = names.sorted{ $0 > $1 }
```

```swift
let strings = numbers.map{ (number) -> String in
                         var number = number
                         var output = ""
                         repeat{
                           output = digitNames[number % 10]! + output
                           number /= 10
                         }while number > 0
                          return output
                         }
```

The situation where a function takes mulitiple functions:

```swift
func loadPicture(from server: Server, completion:(Picture) -> Void, onFailure: () -> Void){
  if let picture = download("photo.jpg", from: server{
    completion(picture)
  } else {
    onFailure()
  }
}
```

```swift
loadPicture(from someSever) { picture in someView.currentPicture = picture} onFailure: {
  print("Couldn't download the next picture.")
}
```

## Capturing Values

```swift
func makeIncrement(forIncrement amount: Int){
  var runningTotal = 0
  func increment() -> Int {
    runningTotal += amount
    return runningTotal
  }
  return increment
}
```

The `increment()` function captures a reference to `runningTotal` and amount from the surrounding function body.

> However, as an optimization, Swift may instead capture and store a copy of a value isn't mutated by a closure,  and if the value isn’t mutated after the closure is created.

```swift
let incrementByTen = makeIncrement(forIncrement: 10)
let incrementBySeven = makeIncrement(forIncrement: 7)

print(incrementByTen())
print(incrementBySeven())
```

**Closures and functions are reference types.**

# Structures and Classes

Classes have additional capabilities that structures don’t have:

- Inheritance enables one class to inherit the characteristics of another.
- Type casting enables you to check and interpret the type of a class instance at runtime.
- Deinitializers enable an instance of a class to free up any resources it has assigned.
- Reference counting allows more than one reference to a class instance.

Unlike structures, class instances don’t receive a default memberwise initializer. Initializers are described in more detail in [Initialization](https://docs.swift.org/swift-book/LanguageGuide/Initialization.html).

**Structures and Enumerations Are Value Types.** Just like strings.

**Classes Are Reference Types.**

Note that *identical to* (represented by three equals signs, or `===`) doesn’t mean the same thing as *equal to* (represented by two equals signs, or `==`). **Identical to** means that two constants or variables of class type refer to exactly the same class instance. **Equal to** means that two instances are considered equal or equivalent in value, for some appropriate meaning of *equal*, as defined by the type’s designer.

# Type Casting

*Type casting* is a way to check the type of an instance, or to treat that instance as a different superclass or subclass from somewhere else in its own class hierarchy.

```swift
var movieCount = 0
var songCount = 0

for item in library{
	if item is Movie {
    movieCount += 1
	} else if item is Song {
    songCount += 1
  }
}

print("Media Libraryncontains \(movieCount) movies and \(songCount) songs.")
```

## Downcasting

```swift
for item in library {
  if let moive = item as? Movie {
    print("Movie: \(movie.name), dir. \(movie.director)")
  } else if let song = item as? Song {
    print("Song: \(song.name), by \(song.artist)")
  }
}
```

## Type Casting for Any and AnyObject

- `Any` can represent an instance of any type at all, including function types.
- `AnyObject` can represent an instance of any class type.

The `Any` type represents values of any type, including optional types. Swift gives you a warning if you use an optional value where a value of type `Any` is expected. If you really do need to use an optional value as an `Any` value, you can use the `as` operator to explicitly cast the optional to `Any`, as shown below.

```swift
let optionalNumber: Int? = 3
things.append(optionalNumber) //Warning
things.append(optionalNumber as Any)  //No warning
```

# Protocol

```swift
protocol SomeProtocol{
  var mustBeSettable : Int { get set }
  var doesNotNeedToBeSettable : Int { get }
}
```

## $\Delta$ instance property requirement &  type property requirement

## type and instance

```swift
struct User{
  let name: String
  
  func sayHello() -> String{
    "Hello, \(name)"
  }
  
  
  static func createUser(with name: String) -> User{
    User(name: name)
  }
}

let jane = User(name:"Jane")

let john = User.createUser(with:"John")
```

`User` is a type. `jane` is a instance of `User`.

`sayHello()` is an instance method and should be called on a `User` instance.

`createUser` is a type method and should be prefixed with a `static`. **The method cannot be called on a instance.**

----

```swift
class Book{
  let title: String
  let author: String
  
  init(title: String, author: String){
    self.title = title
    self.author = author
  }
  
  static func favourites() -> [Book]{
  	[
      Book(title:"LOTR", author: "Tolkien")
    ]
  }
  
  class func moreFavourites() -> [Book]{
  	[
      Book(title:"The Hobbit", author: "Tolkien")
    ]
  }
}

Book.favourites()
Book.moreFavourites()
```

`moreFavourite` is also a type method.

--------

### The difference between `static` and `class` keyword

`static` methods cannot be overwritten , while the function with the keyword `class` can be overwritten.

Enum and struct don't support inheritance.

## Method Requirements

```swift
protocol SomeProtocol{
  static func someTypeMethod()
}

protocol RandomNumberGenerator{
  func random() -> Double
}
```

## Mutating Method Requirements

```swift
protocol Togglable {
  mutating func toggle()
}
```

The method is allowed to modify the instance it belongs to.

You don't need to write `mutating` for a implementation of the method in a class. But in case of a structure or an enumeration, you need to.

```swift
enum OnOffSwitch: Togglable {
  case off, on
  mutating func toggle() {
    switch self {
    	case .off:
				self = .on
      case .on
      	self = .off
    }
  }
}

var lightSwitch = OnOffSwitch.off
lightSwitch.toggle()
```

## Initializer Requirements

```swift
protocol SomeProtocol {
  init(someParameter: Int)
}
```

```swift
protocol SomeProtocol {
  init()
}

class SomeSuperClass {
  init() {
    //initializer implementation goes here
  }
}

class SomeSubClass: SomeSuperClass, SomeProtocol {
  required override init() {
    
  }
}
```

## Protocols as Types

Protocols are types. Begin their names with a capital letter. Just like `Int`, `String`...

```swift
class Dice {
  let sides: Int
  let generator: RandomNumberGenerator
  init(sides:Int, generator: RandomNumberGenerator){
    self.sides = sides
    self.generator = generator
  }
  func roll() -> Int {
    return Int(generator.random() * Double(sides)) + 1
  }
}
```

## Delegation

```swift
protocol DiceGame {
  var dice: Dice {get}
  func play()
}

protocol DiceGameDelegate: AnyObject {
  func gameDidStart(_ game: DiceGame)
  func game(_ game: DiceGame, didStartNewTurnWithDiceRoll diceRoll: Int)
  func gameDidEnd(_ game: DiceGame)
}
```

```swift
class SnakesAndLadders: DiceGame {
    let finalSquare = 25
    let dice = Dice(sides: 6, generator: LinearCongruentialGenerator())
    var square = 0
    var board: [Int]
    init() {
        board = Array(repeating: 0, count: finalSquare + 1)
        board[03] = +08; board[06] = +11; board[09] = +09; board[10] = +02
        board[14] = -10; board[19] = -11; board[22] = -02; board[24] = -08
    }
    weak var delegate: DiceGameDelegate?
    func play() {
        square = 0
        delegate?.gameDidStart(self)
        gameLoop: while square != finalSquare {
            let diceRoll = dice.roll()
            delegate?.game(self, didStartNewTurnWithDiceRoll: diceRoll)
            switch square + diceRoll {
            case finalSquare:
                break gameLoop
            case let newSquare where newSquare > finalSquare:
                continue gameLoop
            default:
                square += diceRoll
                square += board[square]
            }
        }
        delegate?.gameDidEnd(self)
    }
}
```

To prevent strong reference cycles, delegates are declared as weak references. For info about weak references, see  [Strong Reference Cycles Between Class Instances](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html#ID51).

```swift
class DiceGameTracker: DiceGameDelegate {
    var numberOfTurns = 0
    func gameDidStart(_ game: DiceGame) {
        numberOfTurns = 0
        if game is SnakesAndLadders {
            print("Started a new game of Snakes and Ladders")
        }
        print("The game is using a \(game.dice.sides)-sided dice")
    }
    func game(_ game: DiceGame, didStartNewTurnWithDiceRoll diceRoll: Int) {
        numberOfTurns += 1
        print("Rolled a \(diceRoll)")
    }
    func gameDidEnd(_ game: DiceGame) {
        print("The game lasted for \(numberOfTurns) turns")
    }
}
```

```swift
let tracker = DiceGameTracker()
let game = SnakeAndLadders()
game.delegate = tracker
game.play()
```

## Adding Protocol Conformance with an Extension

```swift
protocol TextRepresentable {
  var textualDescription: String { get }
}

extension Dice: TextRepresentable {
  var textualDescription: String {
    return "A \(sides)-sided dice"
  }
}
```

```swift
let d12 = Dice(sides:12, generator:generator: LinearCongruentialGenerator())

print(d12.textualDescription)
```

### Conditionally Conforming to a Protocol

```swift
extension Array: TextRepresentable where Element: TextRepresentable {
  var textualDescription: String{
    let itemAsText = self.map{ $0.textualDescription }
    return "[" + itemsAsText.joined(separator:", ") + "]"
  }
}
```

### Declaring Protocol Adoption with an Extension

Already conforming the protocol

```swift
struct Hamster {
  var name: String
  var textualDescription: String {
    return "A hamster named \(name)"
  }
}

extension Hamster: TextRepresentable {}
```

## Adopting a Protocol Using a Synthesized Implementation

Swift can automatically provide the protocol conformance for `Equatable`, `Hashable`, and `Comparable` in many simple cases.

Swift provides a synthesized implementation of `Equatable` for the following kinds of custom types:

- Structures that have only stored properties that conform to the `Equatable` protocol
- Enumerations that have only associated types that conform to the `Equatable` protocol
- Enumerations that have no associated types

Swift provides a synthesized implementation of `Hashable` for the following kinds of custom types:

- Structures that have only stored properties that conform to the `Hashable` protocol
- Enumerations that have only associated types that conform to the `Hashable` protocol
- Enumerations that have no associated types

Swift provides a synthesized implementation of `Comparable` for enumerations that don’t have a raw value. If the enumeration has associated types, they must all conform to the `Comparable` protocol. To receive a synthesized implementation of `<`, declare conformance to `Comparable` in the file that contains the original enumeration declaration, without implementing a `<` operator yourself. The `Comparable` protocol’s default implementation of `<=`, `>`, and `>=` provides the remaining comparison operators.

```swift
enum SkillLevel: Comparable {
    case beginner
    case intermediate
    case expert(stars: Int)
}
var levels = [SkillLevel.intermediate, SkillLevel.beginner,
              SkillLevel.expert(stars: 5), SkillLevel.expert(stars: 3)]
for level in levels.sorted() {
    print(level)
}
// Prints "beginner"
// Prints "intermediate"
// Prints "expert(stars: 3)"
// Prints "expert(stars: 5)"
```

## Collections of Protocol Types

```swift
let things: [TextRepresentable] = [game, d12, simonTheHamster]

for thing in things {
  print(thing.textualDescription)
}

//thing is of type TextRepresenable, but the actual instance behind it may be Dice, or DiceGame, or Hamster...
```

## Protocol Inheritance

The protocol can add requirements on top of the requirements it inherits.

```swift
protocol PrettyTextRepresentable: TextRepresentable {
  var prettyTextualDescription: String { get }
}
```

```swift
extension SnakesAndLadders: PrettyTextRepresentable {
  var prettyTextualDescription: String {
    //...
  }
}
```

## Class-Only Protocols

```swift
protocol SomeClassOnlyProtocol: AnyObject {
  // class-only protocol definition goes here
}
```

## Protocol Composition

```swift
protocol Named {
  var name: String { get }
}

protocol Aged {
  var age: Int { get }
}

struct Person: Named, Aged {
  var name: String
  var age: Int
}

func wishHappyBirthday(to celebrator: Named & Aged){
	print("Happy birthday, \(celebrator.name), you're \(celebrator.age)!")
}

let birthdayPerson = Person(name: "Malcolm", age:21)
wishHappyBirthday(to: birthdayPerson)
```

## Checking for Protocol Conformance

- The `is` operator returns `true` if an instance conforms to a protocol and returns `false` if it doesn’t.
- The `as?` version of the downcast operator returns an optional value of the protocol’s type, and this value is `nil` if the instance doesn’t conform to that protocol.
- The `as!` version of the downcast operator forces the downcast to the protocol type and triggers a runtime error if the downcast doesn’t succeed.

## Optional Protocol Requirements

`@objc` protocols can be adopted only by classes that inherit from Objective-C classes or other `@objc` classes. 

When you use a method or property in an optional requirement, its type automatically becomes an optional.  From`(Int) -> Stirng` to `((Int) -> String)?`.

```swift
@objc protocol CounterDataSource {
  @objc optional func increment(forCount count: Int) -> Int
  @objc optional var fixedIncrement: Int { get }
}
```

You meant to implement either of the source.

```swift
class Counter {
  var count = 0
  var dataSource: CounterDataSource?
  
  func increment() {
    if let amount = dataSource?.increment?(forCount: count){
      count += amount
    } else if let amount = dataSource?.fixedIncrement {
      count += amount
    }
  }
}
```

## Protocol Extensions

```swift
extension RandomNumberGenerator {
  func rendomBool() -> Bool {
    return rendom() > 0.5
  }
}
//RandomNumberGenerator has been defined as a protocol
```

### Providing Default Implementations

```swift
extension PrettyTextRepresentable {
  var prettyTextualDescription: String {
    return textualDescription
  }
}
```

Using extension to provide default implementations.

## Adding Constraints to Protocol Extensions

```swift
extension Collection where Element: Equtable {
	func allEqual() -> Bool {
		for element in self{
      if element != self.first {
        return false
      }
    }
    return true
	}
}
```

> If a conforming type satisfies the requirements for multiple constrained extensions that provide implementations for the same method or property, Swift uses the implementation corresponding to the most specialized constraints.

# Extensions

Extensions in Swift can:

* Add computed instance properties and computer properties
* Define instance methods and type methods
* Provide new initializers
* Define subscripts
* Define and use new nested types
* Make an existing type conform to a protocol

```Swift
extension SomeType: SomeProtocol, AnotherProtocol {
  //implementation of protocol requirements goes here
}
```

## Computed Properties

```swift
extension Double {
  var km: Double {return self * 1_000.0}
  var m: Double {return self}
  var cm: Double {return self / 100.0}
  var mm: Double {return self / 1_000.0}
  var ft: Double {return self / 3.28084}
}
```

## Initializers

```swift
extension Rect {
  init(center: Point, size: Size){
    let originX = center.x - (size.width / 2)
    let originY = center.y - (size.height / 2)
    self.init(origin: Point(x:originX, y:originY), size:size)
  }
}
```

## Methods

```swift
extension Int {
  func repetitions(task: () -> Void){
    for _ in 0..< self {
      task()
    }
  }
}
```

```swift
extension Int {
  mutating func square() {
    self = self * self
  }
}

var someInt = 3
someInt.square()
```

## Nested Types

```swift
extension Int {
  enum Kind {
    case negative, zero, positive
  }
  
  var Kind: Kind {
    switch self {
      case 0:
      	return .zero
      case let x where x > 0:
      	return .positive
      default:
      	return .negative
    }
  }
}
```

```swift
func printIntegerKinds(_ numbers: [Int]) {
  for number in numbers {
    switch number.kind{
      case .negative:
      	print("- ",terminator:"")
      case .zero:
      	print("0 ",terminator:"")
      case .positive:
      	print("+ ",terminator:"")
    }
    print("")
  }
}
```

# Generics

```swift
func swapTwoInts(_ a: inout Int, _ b: inout Int)
func swapTwoValues<T>(_ a: inout T, _ b: inout T)
```

```swift
var someInt = 3
var anotherInt = 107
swapTwoValues(&someInt, &anotherInt)
```

```swift
struct Stack<Element> {
  var items: [Element] = []
  mutating func push(_ item: Element) {
    items.append(item)
  }
  mutating func pop() -> Element {
    return items.removeLast()
  }
}

var stackOfStrings = Stack<String>()
```

## Extending a Generic Type

```swift
extension Stack {
  var topItem: Element? {
    return items.isEmpty ? nil : items[items.count - 1]
  }
}
```

## Type Constraints

```swift
func someFunction<T: SomeClass, U: SomeProtocol>(someT: T, someU: U){
  //function body goes here
}
```

```swift
func findIndex<T: Equatable>(of valueToFind: T, in array:[T]) -> Int? {
  for (index, value) in array.enumerated () {
    if value == valueToFind {
      return index
    }
  }
  return nil
}
```

## Associated Types

```swift
protocol Container {
  associatedtype Item
  mutating func append(_ item: Item)
  var count: Int {get}
  subscript(i: Int) -> Item { get }
}
```

```swift
struct IntStack: Container {
  var items: [Int] = []
  mutating func push(_ item: Int) {
    items.append(item)
  }
  mutating func pop() -> Int {
    return items.removeLast()
  }
  typealias Item = Int
  mutating func append(_ item: Int){
    self.push(item)
  }
  var count: Int {
    return items.count
  }
  subscript(i: Int) -> Int {
    return items[i]
  }
}
```

Because `IntStack` conforms to all of the requirements of the `Container` protocol, Swift can infer the appropriate `Item` to use, simply by looking at the type of the `append(_:)` method’s `item` parameter and the return type of the subscript. Indeed, if you delete the `typealias Item = Int` line from the code above, everything still works, because it’s clear what type should be used for `Item`.

```swift
struct Stack<Element>: Container {
    // original Stack<Element> implementation
    var items: [Element] = []
    mutating func push(_ item: Element) {
        items.append(item)
    }
    mutating func pop() -> Element {
        return items.removeLast()
    }
    // conformance to the Container protocol
    mutating func append(_ item: Element) {
        self.push(item)
    }
    var count: Int {
        return items.count
    }
    subscript(i: Int) -> Element {
        return items[i]
    }
}
```

### Extending an Existing Type to Specify an Associated Type

For example, the `Array` match the requirements of the `Container` protocol. So, you can extend the `Array` to specify an associated type.

```swift
extension Array: Container {}
```

Now you can use `Array` as an `Container`.

### Adding Constraints to an Associated Type

```swift
protocol Container {
  associatedtype Item: Equatable
  mutating func append(_ item: Item)
  var count: Int {get}
  subscript(i: Int) -> Item{ get }
}
```

### Using a Protocol in Its Associated Type's Constraints

```swift
protocol SuffixableContainer: Container {
  associatedtype Suffix: SuffixableContainer where Suffix.Item == Item
  func suffix(_ size: Int) -> Suffix
}
```

The `Suffix` has two constraints: it must conform to the `SuffixableContainer` protocol and its `Item` type must be the same as the container's `Item` type.

```swift
extension Stack: SuffixableContainer {
  func suffix(_ size: Int) -> Stack {
    var result = Stack()
    for index in (count-size)..<count{
      result.append(self[index])
    }
    return result
  }
}
```

## Generic Where Clauses

A generic `where` clause enables you to require that an associated type must conform to a certain protocol, *or that certain type parameters and associated types must be the same.*

```swift
func allItemsMatch<C1:Container, C2:Container> (_ someContainer: C1, _ anotherContainer: C2) -> Bool where C1.Item == C2.Item, C1.Item: Equatable{
  if someContainer.count != anotherContainer.count {
    return false
  }
  
   for i in 0..<someContainer.count {
     if someContainer[i] != anotherContainer[i] {
       return false
     }
   }
   
  return true
}
```

## Extensions with a Generic Where Clause

```swift
extension Stack where Element: Equatable {
    func isTop(_ item: Element) -> Bool {
        guard let topItem = items.last else {
            return false
        }
        return topItem == item
    }
}
```

If you call `isTop(_:)` method on stack whose elements aren't equatable, you'll get a compile-time error.

Separate each requirement in the list with a comma.

## Contextual Where Clauses

```swift
extension Container {
  func avergae() -> Double where Item == Int {
    var sum = 0.0
    for index in 0..<count {
      sum += Double(count)
    }
  }
  
  func endsWith(_ item: Item) -> Bool where Item: Equatable {
    return count >= 1 && self[count-1] == item
  }
}
```

## Associated Types with a Generic Where Clause

```swift
protocol Container {
  associatedtype Item
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }

    associatedtype Iterator: IteratorProtocol where Iterator.Element == Item
    func makeIterator() -> Iterator
}
```

```swift
protocol ComparableContainer: Container where Item: Comparable { }
```

## Generic Subscripts

```swift
extension Container {
    subscript<Indices: Sequence>(indices: Indices) -> [Item]
        where Indices.Iterator.Element == Int {
            var result: [Item] = []
            for index in indices {
                result.append(self[index])
            }
            return result
    }
}
```

# Opaque Types

```swift
struct Square: Shape {
  var size: Int
  func draw() -> String {
    let line = String(repeating:"*", count: size)
    let result = Array<String>(repeating:line,count: size)
    return result.joined(separator:"\n")
  }
}

func makeTrapezoid() -> some Shape {
  let top = Triangle(size: 2)
  let middle = Square(size: 2)
  let bottom = FlippedShape(shape: top)
  let trapezoid = JoinedShape(
  	top: top,
    bottom: JoinedShape(top: middle, bottom: bottom)
  )
  return trapezoid
}

let trapezoid = makeTrapezoid()
print(trapezoid.draw())
```

Otherwise, you have to be like this:

```swift
struct Trapezoid<Rect1: Shape, Rect2: Shape, Square: Shape>: Shape {
  var rect1: Rect1
  var rect2: Rect2
  //...
}
```

You have to expose the details of the constrcuting of the trapezoid.

If a function with a opaque return type returns form multiple places, all of the possible return values must have the same type.

A example for correcting the invalid function with a Opaque type:
Before:

```swift
func invalidFlip<T: Shape>(_ shape: T) -> some Shape {
  if shape is Square {
    return shape 
  }
  return FlippedShape(shape: shape)
}
```

After:

```swift
//Let's div into the FlippedShape
struct FlippedShape<T: Shape>: Shape {
    var shape: T
    func draw() -> String {
        if shape is Square {
            return shape.draw()
        }
        let lines = shape.draw().split(separator: "\n")
        return lines.reversed().joined(separator: "\n")
    }
}
```

The requirement to always return a single type doesn’t prevent you from using generics in an opaque return type. 

```swift
func `repeat`<T: Shape>(shape: T, count: Int) -> some Collection {
    return Array<T>(repeating: shape, count: count)
}
```

## Difference Between Opaque Types and Protocol Types

```swift
func protoFlip<T: Shape>(_ shape: T) -> Shape {
  if shape is Square {
    return shape
  }
  
  return FlippedShape(shape: shape)
}
```

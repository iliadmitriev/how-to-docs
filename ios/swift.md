
# Classes

```swift
// classes
class Person {
    // attribute
    var name = ""
    // constructor
    init(name: String) {
        self.name = name
    }
}

class BlogPost {
    // attributes
    var title: String?
    var body = "hey"
    var author: Person?
    var numberOfComments: Int = 0
    
    // method returning value
    func addComment() -> Int {
        self.numberOfComments += 1
        return self.numberOfComments
    }
    // method
    func shareArticel() {
        print(body)
    }
    // calculated property
    var fullTitle: String {
        if nil != self.title && nil != self.author {
            return self.title! + " by " + self.author!.name
        } else if nil != self.title {
            return self.title!
        } else {
            return "No title"
        }
    }
}
// instance
let myPost = BlogPost()

myPost.title = "Title of post"
myPost.author = Person(name: "John")
print(myPost.fullTitle)

let myPostTwo = BlogPost()
myPost.title = "Another title of post"
if let myTitle = myPostTwo.title {
    print(myTitle)
}
```

# Types

## Strings

```swift
// Strings
let personName: String = "Steve Jobs"
let banner: String = """
    line 1
    line 2
    """
// print string
print(personName)
// print utf-8 string
print(personName.utf8)
// print utf-16 string
print(personName.utf16)

// print string length
print(personName.count)

// get index of first 5 letters
let first_idx=personName.index(personName.startIndex, offsetBy: 5)
// get first 5 letters
let first1 = personName[..<first_idx]
// or prefix
let first2 = personName.prefix(upTo: first_idx)
// or
let first3 = personName.prefix(5)

// get last 4 letters
let index = personName.index(personName.endIndex, offsetBy: -4)
let last1 = personName[index...]
// or suffix
let last2 = personName.suffix(from: index)
// or
let last3 = personName.suffix(4)


// get index of first met space or index of string's end
let firstSpace = personName.firstIndex(of: " ") ?? personName.endIndex
// get name (all characters between beginning of string and first met space
let firstName = personName[..<firstSpace]
```

## Conditionals

```swift
// conditionals
var v = 12
var y = v > 10 ? 5 : nil
if let val = y {
    print(val)
}
```

## Any type

```swift
// any type
var some: Any
some = 100
print(some)
some = "This is what?"
print(some)
```

## Enums

```swift
// enums
enum States: Int {
    case New = 0
    case InProgress = 1
    case Aborted = 2
    case Completed = 3
}

var current: States = States.New

print(current)

current = States.InProgress

print(current)
print(type(of: current))

current = .Completed

if let val = States(rawValue: 2)  {
    current = val
}
print(current)

func checkState(_ value: States) {
    switch value {
    case .New,.InProgress:
        print("Not Ready")
    case .Completed:
        print("Done")
    default:
        print("Unknown")
    }
}

checkState(current)
```

# Protocols

Protocols is analogue of Java interface
```swift
// apple naming conventions
// DataSource - implies data related functionality
// Delegate - protocol with functionality that has to do with user iteraction
protocol CarDataSource {
    var color: String { get set } // ability to get and set
    func isAllWheelDrive() -> Bool
    func drive() -> Double
}

class Car {

}

class BMW: Car, CarDataSource {
    var currentSpeed: Double
    var color: String {
        set {
            
        }
        get {
            return "red"
        }
    }
    
    
    override init() {
        self.currentSpeed = 0.0
        super.init()
    }
    
    func isAllWheelDrive() -> Bool {
        return true
    }
    
    func drive() -> Double {
        currentSpeed = 1.0
        return currentSpeed
    }
    
    func speedUP(up: Double = 1.1) -> Double{
        currentSpeed *= up
        return currentSpeed
    }
    
    
}
```

# Strong and weak memmory

```swift
// Strong vs Weak
//
class Baloon {
    var color: String = "red"
    init(color: String) {
        self.color = color
    }
}

class Child {
    weak var balloon: Baloon?
}

var jimmi = Child()

var b = Baloon(color: "yellow")
jimmi.balloon = b

if let bal = jimmi.balloon {
    print(bal.color)
}
```

# Closures

closures is lambda functions

```swift
func isGreaterThanThree(number: Int) -> Bool {
    number > 3
}
isGreaterThanThree(number: 2)

var myFunction: ((Int) -> Bool) = { number in
    number > 3
}
myFunction(4)

var mySecFunction: ((Int) -> Bool)?
mySecFunction = { number in
    number > 3
}

if let ret = mySecFunction {
    ret(4)
}
```

# Extensions

```swift
extension String {
    func index(from: Int) -> Index {
        return self.index(startIndex, offsetBy: from)
    }
    func substring(from: Int) -> String {
        let fromIndex = index(from: from)
        return String(self[fromIndex...])
    }
    func substring(to: Int) -> String {
        let toIndex = index(from: to)
        return String(self[..<toIndex])
    }
    func substring(with r: Range<Int>) -> String {
        let startIndex = index(from: r.lowerBound)
        let endIndex = index(from: r.upperBound)
        return String(self[startIndex..<endIndex])
    }
}
let str: String = "Steve Jobs"

print(str.substring(to: 5))
print(str.substring(from: 6))
print(str.substring(with: 0..<5))

```

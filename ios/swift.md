
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


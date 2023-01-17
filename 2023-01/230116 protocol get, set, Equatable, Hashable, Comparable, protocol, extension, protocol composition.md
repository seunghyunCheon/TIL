TIL (Today I Learned)
===
학습내용
---
- protocol get/set 요구사항
- Equatable, Hashable, Comparable 학습하기
- 프로토콜 학습하기
- required initializer
- Delegation

## 고민한 점 / 해결 방법
## protocol get/set 요구사항
(헛점을 알려준 vetto 감사합니다..ㅎ)
- 프로토콜 프로퍼티 요구사항에는 연산프로퍼티의 형식으로 제공되기 때문에 var로 선언한다.
- 프로토콜 프로퍼티 요구사항에 get만 정의할 경우 구현부에서 get/set을 내포한다.
- 구현부에서 let으로 일반 프로퍼티를 선언하거나 연산프로퍼티로 get만 정의할 경우 읽기전용이 된다.
```swift
import Foundation

protocol A {
    var a: Int { get } // get으로 구현하거나 get,set으로 구현하거나
    var b: Int { get set } // get,set으로 구현
    var c: Int { get }
}

struct B: A {
    var a: Int { // get,set의 가능성을 담고 있었지만 get만정의해서 read-only
        get {
            return 2
        }
    }
    var b = 2 // get, set으로 프로토콜 요구사항이 있으므로 get, set구현. var로 인해 set도 가능
    let c = 3 // get,set의 가능성을 담고 있었지만 let을 사용함으로써 read-only
}

var b = B()
// b.a = 2 a가 read-only이기 때문에 컴파일에러
b.b = 4
// b.c = 4 c가 read-only이기 때문에 컴파일에러
print(b.b) // 4

```


## Equatable, Hashable, Comparable
> 프로토콜 공식문서의 Adopting a Protocol Using a Synthesized Implementation챕터를 읽던 중 Equatable, Hashable, Comparable이 궁금해서 찾아보게 되었다. 

### Equatable
> 값의 동등성을 비교하는 타입(프로토콜)
- Equatable 프로토콜을 준수하는 타입들은 ==, !=을 통해 값을 비교할 수 있다. Swift의 기본 library는 Equatable 프로토콜을 준수한다.
- 시퀀스와 컬렉션 연산을 쉽게 만들어준다. Equatable을 준수하고 있기 때문에 contains메서드 내에서 값이 동일한 지 판단할 수 있는 것
```swift
let students = ["Kofi", "Abena", "Efua", "Kweku", "Akosua"]

let nameToCheck = "Kofi"
if students.contains(nameToCheck) {
    print("\(nameToCheck) is signed up!")
} else {
    print("No record of \(nameToCheck).")
}
// Prints "Kofi is signed up!"
```
<img src="https://i.imgur.com/uVeB8H6.png" width="400"/>

### Equatable Protocol 준수
- Equatable를 custom type에 추가하면 하나의 컬렉션안에서 특정 인스턴스를 찾기 수월해진다.
- Equatable은 Hashable과 Comparable 프로토콜의 base 프로토콜이다.
- 사용자 정의 타입에서 다음의 기준을 충족시키면 선언만해도 자동구현을 해준다.
> 1. 구조체의 모든 저장속성이 Equatable을 따른다
> 2. 열거형의 연관타입이 전부 Equatable을 따른다. (연관값이 없는 경우 Equatable 정의없이도 가능하다)
- 클래스는 직접 구현해야 한다..^
```swift
class StreetAddress {
    let number: String
    let street: String
    let unit: String?

    init(_ number: String, _ street: String, unit: String? = nil) {
        self.number = number
        self.street = street
        self.unit = unit
    }
}
extension StreetAddress: Equatable {
    static func == (lhs: StreetAddress, rhs: StreetAddress) -> Bool {
        return
            lhs.number == rhs.number &&
            lhs.street == rhs.street &&
            lhs.unit == rhs.unit
    }
}
```
- 재밌는 사실은 ==메서드 안에 number, street, unit이 아닌 number로만 비교해서 return 할 수도 있다. 
- 클래스 식별자 비교는 Equality와 다르다. ==은 값들을 비교하는 것이고 ===은 참조하고 있는 인스턴스 비교를 하는 것.

### Hashable
> 정수 해시값을 만들기 위해 Hasher를 통해 해쉬하는 타입(프로토콜)
- hasable프로토콜을 따르는 어떤 타입들을 set이나 딕셔너리 키로 사용할 수 있다.
- Swift 표준 라이브러리들은 equatable과 마찬가지로 hasable을 따르고 있다. 
- 연관값이 없는 열거형일 경우 Hashable프로토콜을 자동으로 얻는다.
- custom type에 hash(into:) 메서드를 구현함으로써 Hasable 프로토콜을 따를 수 있다.
- 저장속성이 모두 Hasable한 구조체나 연관값이 전부 Hasable한 열거형의 경우 컴일러는 자동으로 hash(into:)를 제공한다.

### Hasable protocol 준수
> 딕셔너리의 키나 set의 일부로 사용하기 위해서 Hasable프로토콜을 준수한다. Hashable 프로토콜은 Equatable을 상속받고있기 때문에 Equatable 요구사항도 구현해야 한다
```swift
struct GridPoint {
    var x: Int
    var y: Int
}

extension GridPoint: Hashable {
    static func == (lhs: GridPoint, rhs: GridPoint) -> Bool {
        return lhs.x == rhs.x && lhs.y == rhs.y
    }

    func hash(into hasher: inout Hasher) {
        hasher.combine(x) // 해당 타입의 모든 저장 프로퍼티를 전달. 이들은 hashable해야 함
        hasher.combine(y)
    }
}
var tappedPoints: Set = [GridPoint(x: 2, y: 3), GridPoint(x: 4, y: 1)]

let nextTap = GridPoint(x: 0, y: 1)
if tappedPoints.contains(nextTap) {
    print("Already tapped at (\(nextTap.x), \(nextTap.y)).")
} else {
    tappedPoints.insert(nextTap)
    print("New tap detected at (\(nextTap.x), \(nextTap.y)).")
}
// Prints "New tap detected at (0, 1).")
```
- 구조체의 프로퍼티들이 Hashable 프로토콜을 따르고 있다면 ==과 hash함수를 구현하지 않아도 직접 구현하지 않아도 자체적으로 구현이 된다.
```swift
extension GridPoint: Hashable { }
```

### Comparable
> 관계 연산자 <, <=, >=, >를 사용하여 비교될 수 있는 타입(프로토콜)
```swift
var measurements = [1.1, 1.5, 2.9, 1.2, 1.5, 1.3, 1.2]
measurements.sort()
print(measurements)
// Prints "[1.1, 1.2, 1.2, 1.3, 1.5, 1.5, 2.9]"
```
- array의 요소들이 Comparable을 따르고 있기 때문에 sort메서드로 오름차순을 할 수 있다.

### Comparable protocol 준수
- 구조체의 저장속성들이 Comparable할지라도 프로토콜의 메서드를 직접 구현해야 한다. Equatable의 경우에는 저장속성들이 다 해시값으로 되고 모두가 동등할 때 true를 리턴하지만 <. <=, >, >=들은 대소관계를 비교하는 것이기 때문에 어떤 값으로 비교할지 정해야 하기 때문
```swift

struct Person: Comparable {
    static func < (lhs: Person, rhs: Person) -> Bool {
        return lhs.age < rhs.age
    }
    
    var name = ""
    var age = 0
}
 
let brody = Person.init(name: "brody", age: 28)
let goatt = Person.init(name: "goatt", age: 30)
 
brody == goatt           // false
brody <= goatt          // true
```
- <만 정의해도 다른 대소관계가 지원된다. (==도 마찬가지)
- 클래스의 경우에는 ==과 <둘 다 정의해야 한다.


### 프로토콜 확장
> 이미 존재하는 타입에 프로토콜을 따르게 하기 위해 extension을 사용할 수 있다
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

### 조건적 프로토콜 따르기
- 특정 조건을 만족시킬때만 프로토콜을 따르도록 제한할 수 있다. 이는 where절을 이용한다. 다음예시는 Array중에 각 원소가 TextRepresentable인 경우에만 textualDescription프로퍼티를 가질 수 있게 만든다.
```swift
extension Array: TextRepresentable where Element: TextRepresentable {
    var textualDescription: String {
        let itemsAsText = self.map { $0.textualDescription }
        return "[" + itemsAsText.joined(separator: ", ") + "]"
    }
}
let myDice = [d6, d12]
print(myDice.textualDescription)
// Prints "[A 6-sided dice, A 12-sided dice]"
```

### 확장을 이용한 프로토콜 채택
- 만약 타입이 프로토콜의 요구사항을 다 구현했다면 프로토콜을 확장해서 빈 블록을 선언할 수 있다. 하지만 프로토콜 채택은 반드시 해줘야하기 때문에 확장에서 꼭 채택을 해야 한다.
```swift
struct Hamster {
    var name: String
    var textualDescription: String {
        return "A hamster named \(name)"
    }
}
extension Hamster: TextRepresentable {}
```

### 프로토콜 상속
> 프로토콜은 하나이상의 프로토콜을 상속할 수 있고 요구사항을 추가할 수 있다.
```swift
protocol A {
    var text: String { get }
}

protocol B: A {
    var prettyText: String { get }
}

class C: B {
    var prettyText: String = "a"
    
    var text: String = "b"
    
    var name: String = "brody"
}
```
- 또는 extension에서 계산속성을 이용할 수 있다.
```swift
class C {
    var name: String = "brody"
}

extension C: B {
    var prettyText: String {
        return "a"
    }
    
    var text: String {
        return "b"
    }
}
```
### 프로토콜 합성
> 동시에 여러 프로토콜을 따르는 타입이 필요할 때 유용. 
- 마치 결합된 여러 프로토콜의 일시적인 프로토콜을 정의한 것처럼 선언된다.
- &를 통해 많은 수의 프로토콜을 결합할 수 있다.
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
func wishHappyBirthday(to celebrator: Named & Aged) {
    print("Happy birthday, \(celebrator.name), you're \(celebrator.age)!")
}
let birthdayPerson = Person(name: "Malcolm", age: 21)
wishHappyBirthday(to: birthdayPerson)
// Prints "Happy birthday, Malcolm, you're 21!"
```
- 프로토콜 & 프로토콜 뿐만 아니라 프로토콜 & 클래스도 가능

### 옵셔널 프로토콜 요구사항
- 요구사항을 옵셔널형태로도 정의할 수 있다. 즉 구현부에서 정의를 해도되고 안해도 된다. 이는 @objc optional키워드를 붙여주며 @objc가 있기 때문에 클래스에서만 사용이 가능하다. 요구사항을 옵셔널로 만드려면 프로토콜도 옵셔널로 해줘야 한다.
```swift
@objc protocol CounterDataSource {
    @objc optional func increment(forCount count: Int) -> Int
    @objc optional var fixedIncrement: Int { get }
}
```
- 재미있는 사실은 increment함수가 @objc optional로 정의되어있기 때문에 옵셔널값을 리턴한다. 즉 (Int) -> String이 ((Int) -> String)?으로 되는 것이다.
- 요구사항이 전부 옵셔널로 구현이 되어있다면 구현부에서 구현하지 않아도 된다. 하지만 그건 좋은 dataSource는 아니다.
```swift
class Counter {
    var count = 0
    var dataSource: CounterDataSource?
    func increment() {
        if let amount = dataSource?.increment?(forCount: count) {
            count += amount
        } else if let amount = dataSource?.fixedIncrement {
            count += amount
        }
    }
}

class ThreeSource: NSObject, CounterDataSource {
    let fixedIncrement = 3
}
var counter = Counter()
counter.dataSource = ThreeSource()
for _ in 1...4 {
    counter.increment()
    print(counter.count)
}
// 3
// 6
// 9
// 12    
```
- ThreeSource에는 fixedIncrement만 정의하고 있기 때문에 Counter클래스내의 increment함수에서 dataSource?.increment?가 실패한다. 그리고 옵셔널 요구사항들은 전부 옵셔널을 리턴하기 때문에 사용하려면 바인딩을 통해 풀어줘야 한다.

---
- protocol get/set 요구사항
- Equatable, Hashable, Comparable 학습하기
- 프로토콜 학습하기
- extension

###### tags: `protocol get/set`, `Equatable`, `Hashable`, `Comparable`, `protocol`, `extension`, `protocol composition`


### Reference
- [apple developer Documentation - contains](https://developer.apple.com/documentation/swift/array/contains(_:))
- [apple developer Documentation - Equatable](https://developer.apple.com/documentation/swift/equatable)
- [apple developer Documentation - Hasable](https://developer.apple.com/documentation/swift/hashable)
- [apple developer Documentation - Comparable](https://developer.apple.com/documentation/swift/comparable)
- [개발자 소들이 - Comparable](https://babbab2.tistory.com/150)

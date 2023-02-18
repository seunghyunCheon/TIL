230218 제네릭(Generics)
===
TIL (Today I Learned)
===
학습내용
---
- 제네릭 학습하기

## ✏️ 제네릭(Generics)

제네릭은 어떤 타입과도 함께 작동할 수 있는 재사용가능하고 유연한 코드를 만들 수 있게 만드는 스위프트의 고급 기능이다. 
스위프트의 `Array`와 `Dictionary`는 제네릭이기 때문에 어떤 타입이든 수용할 수 있다.

### The Problem That Generics Solve
두 개의 `Int`타입의 변수를 바꾸는 함수 `swapTwoInts`가 있다.
```swift
func swapTwoInts(_ a: input Int, _ b: input In) {
    let tempA = a
    a = b
    b = tempA
}

var someInt = 3
var anotherInt = 107
swapTwoInts(&someInt, &anotherInt)
```

만약 `String`변수나 `Double`변수를 `swap`하려고 한다면 타입만 바꾼 새로운 함수를 정의해야 한다. 이러한 경우에 제네릭으로 문제를 해결할 수 있다.

### Generic Functions

제네릭 함수는 어떤 타입과도 함께 작동한다. 다음은 `swapTwoInts`를 제네릭함수로 바꾼 예시이다.
```swift
func swapTwoValues<T>(_ a: inout: T, _ b: inout T) {
    let tempA = a
    a = b
    b = tempA
}
```
`swapTwoValues`함수는 `swapTwoInts`와 함수 바디가 동일하지만 정의부는 다르다. 

```swift
func swapTwoInts(_ a: inout Int, _ b: inout Int)
func swapTwoValues<T>(_ a: inout T, _ b: inout T)
```

함수의 제네릭 버전은 이 경우에는 T라고 부르는 `placeholder type name`을 사용한다. `placeholder type name`은 T가 무엇이 되어야하는지에 대한 것을 말하지 않는다. 그러나 `a`와 `b`는 동일한 타입이어야 한다는 것은 말하고 있다. `T`의 실제 타입은 호출할 때 정해진다.

다른 차이점은 `swapTwoValues`함수 뒤에 `<T>`가 따라온다는 점이다. 이 부분에는 T가 아닌 다른 값을 사용할 수도 있다. 

### Type Paramenters
위 예시에서 `placeholder type`인 T를 타입 파라미터라고 부른다. 타입 파라미터는 함수 이름 뒤에 쓰여지며 `<>`안에서 네이밍을 한다.

일단 타입 파라미터를 명시하면 함수의 파라미터, 리턴타입, 타입 어노테이션으로 사용할 수 있다. 그리고 이는 함수가 호출될 때 전달하는 변수의 타입으로 대체가 된다. 

선호에 따라 타입 파라미터를 여러개 작성할 수 있다. (Dictionary가 그 예시)

### Naming Type Parameters
대부분의 경우에 타입 파라미터는 설명이되는 이름을 가진다. `Dictionary<Key, Value>`에서 `Key`와 `Value`, `Array<Element>`에서 `Element`는 독자가 제네릭 타입 및 함수와 타입 파라미터와의 관계를 쉽게 알 수 있도록 만든다. 
그러나 그것들 사이에서 특별한 의미가 존재하지 않는다면 `T`, `U`, `V`를 사용하는 게 관행이다.

### 👏 Note
타입 파라미터는 값이 아닌 타입 파라미터임을 나타내기 위해 항상 `upper camel case`로 네이밍을 한다.

### Generic Types
제네릭 함수에 더하여 제네릭 타입도 가능하게 만든다. `Array`와 `Dictionary`가 그 예시이다. `stack`은 `Array`와 비슷하지만 조금 더 제한된 연산의 집합이다. `Array`는 어떤 위치에서나 새로운 `item`을 삽입, 제거하지만 `stack`은 오로지 끝에서만 삽입하고 삭제할 수 있다. 

### 👏 Note
스택의 컨셉은 `UINavigationController` 클래스에서 사용도니다. `pushViewController`를 호출하면 맨 마지막에 `ViewController`를 넣게되고 `popViewControllerAnimated`를 호출하면 맨 마지막의 아이템을 제게한다.

제네릭이 아닌 버전의 `stack`의 코드는 다음과 같다.
```swift
struct IntStack {
    var items: [Int] = []
    mutating func push(_ item: Int) {
        items.append(item)
    }
    mutating func pop() -> Int {
        return items.removeLast()
    }
}
```

위 스택은 `Int`에 대해서만 사용이 가능하기에 제네릭으로 바꿔주는 게 좋다.

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
```
위 스택은 `Int` 스택과 타입 파라미터를 사용한 사실만 제외하면 동일하다. `Element`는 후에 실제 타입으로 대체되며 실제타입은 위 `struct`안 어디에서든지 쓰여질 수 있다. 

### Extending a Generic Type
제네릭 타입을 화장할 때, `extension` 정의부에서 타입 파라미터를 제공할 수 없다. 대신에 `extension`에서 타입 파라미터를 사용할 수 있다. 

```swift
extension Stack {
    var topItem: Element? {
        return items.isEmpty ? nil : items[items.count - 1]
    }
}
```
`topItem`은 실제 `items`의 요소를 제거하지않고 끝 요소를 반환하는 메서드이다. 

### Type Constraints
`swapTwoValues`와 `Stack` 타입은 어떤타입과도 동작할 수 있지만 때로는 특정 타입 제약을 주는 것이 유용하다. 타입제약은 타입 파라미터가 특정 클래스나 특정 프로토콜, 프로토콜 합성 등을 따르도록 하는 것을 강제한다.

예를들어 스위프트의 `Dictionary` 타입에서 `key`들은 `hashable`해야 한다. 이것은 `key`가 그 자체로 `unique`하게 표현이 되어야 한다는 것을 의미한다. 그렇기에 `Dictionary`의 `key`들은 `hasable`프로토콜을 따르고 있고 이는 이미 값이 존재하는지를 판별할 수 있게 만든다.

사용자 정의 타입제약을 사용자 정의 제네릭 타입에 따르게 할 수 있다. `Hasable`과 같은 추상적인 개념은 타입을 구체화하는 것보다는 개념적 특성의 관점에서 타입을 특성화한다.

### Type Constraint Syntax
타입 제약문법은 다음과 같이 사용한다.
```swift
func someFunction<T: SomeClass, U: SomeProtocol>(someT: T, someU: U) {
}
```
위 가상의 함수는 두가지 타입 파라미터를 가진다. 첫 번째 타입파라미터는 `SomeClass`
의 서브클래싱을 따라야 하고 두 번째 타입파라미터는 `SomeProtocol`을 따라야 한다.

### Type Constraints in Action
먼저 `non genneric` 함수를 살펴보자. `findIndex`함수는 첫 번째 인자로 찾고자하는 `String`을 넣고 두 번째 인자로 전체 리스트인 `String`배열을 넣는다. 
만약 리스트에 찾고자하는 문자열이 존재한다면 그 문자열의 인덱스를 반환하고 아니라면 `nil`을 반환한다.
```swift
func findIndex(ofString valueToFind: String, in array: [String]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == valueToFind {
            return index
        }
    }
    return nil
}

let strings = ["cat", "dog", "llama", "parakeet", "terrapin"]
if let foundIndex = findIndex(ofString: "llama", in: strings) {
    print("The index of llama is \(foundIndex)")
}
```

현재는 `String`만 받을 수 있는 함수이다. 이를 제네릭을 이용해서 모든 타입과 작동하도록 만든다.

```swift
func findIndex<T>(of valueToFind: T, in array:[T]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == valueToFind {
            return index
        }
    }
    return nil
}
```

찾고자 하는 문자열과 배열은 제네릭으로 받았지만 인덱스를 반환하는 것은 여전하기 때문에 `Int?`를 반환하고 있다.

위 코드는 사실 에러가 발생한다. 문제점은 `if value == valueToFind`에서 발생하는 동등성 체크이다. 스위프트의 모든 타입들이 동등성을 비교할 수 있는것은 아니다. 만약 복잡한 데이터 모델을 표현하기 위해 사용자 정의 타입의 클래스나 구조체를 만든다면 스위프트가 이들에 대해 동등성을 비교할 수 없게된다. 이 때문에 모든 `T`타입들이 동등성을 비교할 수 없어 컴파일 에러가 나오는 것이다.

하지만 스위프트는 `Equatable`이라는 프로토콜을 정의한다. 이 프로토콜은 특정 타입들이 비교가 가능하도록`==`과 `!=`을 구현하도록 요구한다. 

`Equatable`을 채택한 타입들은 `equal opearator`를 지우너하기 대문에 안전함을 보장한다. 이 사실을 표현하기 위해 타입 제약으로 `Equatable`을 `findIndex`의 타입 파라미터에 채택한다.

```swift
func findIndex<T: Equatable>(of valueToFind: T, in array:[T]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == valueToFind {
            return index
        }
    }
    return nil
}
```

### Associated Types
프로토콜을 정의할 때 때때로 프로토콜의 부분으로써 하나이상의 연관 타입을 작성하는 것이 유용하다. 연관 타입은 프로토콜의 부분으로써 사용되는 `placeholder name`을 제공한다. 실제 타입은 프로토콜을 채택할 때 구현이 되며 `associatedtype` 키워드와함께 명시한다.

### Associated Types in Action
`Item`이라고 불리는 연관 타입을 선언하는 `Container`라는 프로토콜 예시를 살펴보자.
```swift
protocol Container {
    associatedtype Item
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}
```
이 프로토콜은 `items`가 어떻게 저장되고 어떤타입으로 저장되는지 명시하지 않는다. 이 프로토콜은 오직 세개의 기능을만 지정한다.
`Container`를 채택하는 어떤 타입은 값이 저장하고 있는 것을 명시해야 한다. 특히 container에 추가되거나 subscript에 의해 반환되는 값에 대해 명확해야 한다.

이러한 요구사항을 정의하기위해 `Container`프로토콜은 구체적으로 어떤타입인지 아는 것 없이 container가 가지고있는 element의 타입을 추론할 방법이 필요하다. 즉 프로토콜의 placeholder를 제공하는 것.

이는 associated type으로 불려지는 Item을 정의함으로써 문제를 해결할 수 있다. 이 프로토콜에서는 Item이 어떤 타입인지 정의하는 것이 아니다. 타입에 대한 정보는 `Container`를 채택하는 타입에게 남겨두는 것이다. 그럼에도불구하고 Item은 프로토콜내부에서 추론이 될 수 있기 때문에 Item을 사용하는 공간에서는 전부 같은 타입을 사용하는 것을 예상할 수 있게된다.

```swift
struct IntStack: Container {
    // original IntStack implementation
    var items: [Int] = []
    mutating func push(_ item: Int) {
        items.append(item)
    }
    mutating func pop() -> Int {
        return items.removeLast()
    }
    // conformance to the Container protocol
    typealias Item = Int
    mutating func append(_ item: Int) {
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

`IntStack`은 `Containter`의 모든 요구사항을 구현했다. 
`typealias Item = Int`를 통해 `Item`의 타입을 `Int`로 구체화를 했다. 하지만 이를 지워도 잘 작동한다. 왜냐하면 `append`와 `subscript`에서 `Item`타입을 `Int`로 추론하고 있기 때문에 실제 `Item`이 타입추론되기 때문이다. 

```swift
struct Stack<Element>: Container {
    var items: [Element] = []
    mutating func push(_ item: Element) {
        items.append(item)
    }
    
    mutating func pop() -> Element {
        return items.removeLast()
    }
    
    // conformance to Container protocol
    mutating func append (_ item: Element) {
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

위 예시에서는 타입파라미터 `Element`가 `append`와 `subscript`에서 사용된다. 그러므로 스위프트는 `Element`를 `Item`으로서 적절한 타입으로 추론하게 된다.

### ✏️ Extending an Existing Type to Specify an Associated Type
존재하는 타입에 대해 확장을 이용해 프로토콜을 채택할 수 있다. 
Swift의 `Array`에는 이미 `append`, `count property`, `subscript`가 존재한다. 이에 대해 `Container`를 확장시킨다면 위의 `Stack`을 구현한 것처럼 사용할 수 있다.

```swift
extension Array: Container { }
```
Array의 존재하는 `append`메서드는 `Item`의 적절한 타입으로 추론해 사용할 것이다. 

### ✏️ Adding Constraints to an Associated Type
연관타입에 프로토콜 타입제약을 추가할 수 있다. 

```swift
protocol Container {
    associatedtype Item: Equatable
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}
```
이 프로토콜을 따르기 위해서 containter의 Item타입은 `Equatable`프로토콜을 따라야 한다.

### ✏️ Using a Protocol in Its Associated Type's Constraints
연관타입에 조건을 걸 수 있다. 이 경우에는 특정 타입인지, 특정 프로토콜을 따르고 있는지에 대해 사용될 수 있다. 다음 예시는 `Suffix`가 `SuffixableContainer`프로토콜을 따르고 `Item`타입이 반드시 `Container`의 `Item`타입이어야한다는 조건을 추가한 것이다.

```swift
protocol SuffixableContainer: Container {
    associatedtype Suffix: SuffixableContainer where Suffix.Item == Item
    func suffix(_ size: Int) -> Suffix
}

extension Stack: SuffixableContainer {
    func suffix(_ size: Int) -> Stack {
        var result = Stack()
        for index in (count-size)..<count {
            result.append(self[index])
        }
        return result
    }
    // Inferred that Suffix is Stack
}
var stackOfInts = Stack<Int>()
stackOfInts.append(10)
stackOfInts.append(20)
stackOfInts.append(30)
let suffix = stackOfInts.suffix(2)
// suffix contains 20 and 30
```

### ✏️ Generic Where Clauses
제네릭에서도 where절을 사용할 수 있다. 

```swift
func allItemsMatch<C1: Container, C2: Container>
    (_ someContainer: C1, _ anotherContainer: C2) -> Bool
    where C1.Item == C2.Item, C1.Item: Equatable {
        
        // 두개의 컨테이너가 같은 카운트를 가졌는지 체크
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
Container종류 자체가 달라도 상관없다. Container의 같은 위치의 인덱스 값이 모두 같기만한다면 true를 얻게된다.
```swift
var stackOfStrings = Stack<String>()
stackOfStrings.push("uno")
stackOfStrings.push("des")
stackOfStrings.push("tres")

var arrayOfStrings = ["uno", "des", "tres"]

if allItemsMatch(stackOfStrings, arrayOfStrings) {
    print("All items match.")
} else {
    print("Not all items match.")
}
```


### 참고자료
[Swift org - Generic](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/generics/)

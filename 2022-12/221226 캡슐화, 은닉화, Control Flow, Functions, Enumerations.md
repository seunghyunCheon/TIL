221226 캡슐화, 은닉화, Control Flow, Functions, Enumerations
===
TIL (Today I Learned)
===
학습내용
---
- 캡슐화, 은닉화 학습하기
- Swift Language Guide 기본문법 학습하기

## 은닉화와 캡슐화
#### 사전 지식
> 일반화는 어떤 개체에 대하여 공통된 특성이나 행위를 추출해 객체를 만드는 것을 말한다.
> 추상화는 복잡한 세계에서 이해하기 쉬운 수준으로 단순화하는 것을 말한다.

은닉화는 외부에서 객체의 속성에 접근하지 못하게 하는 것을 말한다. swift에서 은닉화는 private와 같은 접근제어자를 통해 설정할 수 있다.
캡슐화는 사용자 입장에서는 쉽게 보이지만 내부의 로직들을 감추도록 하는 것을 말한다. 
캡슐화의 예시로는 자판기의 버튼이 있다. 사용자 입장에서는 버튼을 누르기만 하면 원하는 음료수를 뽑을 수가 있다. 하지만 어떻게 선택된 음료가 자판기 밑으로 내려오는지와 같은 부분들은 알 수가 없다. 
추상화의 개념과 혼동될 수 있기에 아래 예시를 만들었다.
```swift
struct BeverageMachine() {
    // 사용자 입장에서 보이는 부분 -> 캡슐화
    func beverageButtonTapped() {
        choiceBeverage()
        processBerverage()
    }
    // 복잡한 로직들이 처리되고 네이밍은 단순화된 부분 -> 추상화
    private func choiceBeverage()
    private func processBeverage()
}
```

## Control Flow
### Switch
값이 간격에 포함되어있는지 확인 할 수 있다.
```swift
let approximateCount = 8
let naturalCount: String
switch approximateCount {
case 0:
    naturalCount = "no"
case 1..<5:
    naturalCount = "a few"
case 5..<12:
    naturalCount = "several"
default:
    naturalCount = "many"
}
print(naturalCount) // output: "several"
```

튜플을 사용해서 가능성 있는 값과 매칭시킬 수 있다.
```swift
let somePoint = (1, 1)
switch somePoint {
case (0, 0):
    print("\(somePoint) is at the origin")
case (_, 0):
    print("\(somePoint) is on the x-axis")
case (0, _):
    print("\(somePoint) is on the y-axis")
case (-2...2, -2...2):
    print("\(somePoint) is inside the box")
default:
    print("\(somePoint) is outside of the box")
}
// Prints "(1, 1) is inside the box"
```

값을 바인딩 할 수 있다. 파라미터로 들어오는 상수가 둘 다 바인딩되서 사용하고자 할 때 let키워드를 앞으로 묶을 수 있다.
```swift
let anotherPoint = (2, 0)
switch anotherPoint {
case (let x, 0):
    print("on the x-axis with an x value of \(x)")
case (0, let y):
    print("on the y-axis with a y value of \(y)")
case let (x, y):
    print("somewhere else at (\(x), \(y))")
}
// Prints "on the x-axis with an x value of 2"
```
switch에서 break는 case를 무시하고자 할 때 사용한다.

### while
while반복문에 label을 달아 control을 제어를 함으로써 논리를 읽고 이해하는데 도움을 준다. 만약 switch문에서 break만 주었다면 단순히 switch문을 탈출하는 역할만 했을 것이다.
```swift
gameLoop: while square != finalSquare {
    diceRoll += 1
    if diceRoll == 7 { diceRoll = 1 }
    switch square + diceRoll {
    case finalSquare:
        // diceRoll will move us to the final square, so the game is over
        break gameLoop
    case let newSquare where newSquare > finalSquare:
        // diceRoll will move us beyond the final square, so roll again
        continue gameLoop
    default:
        // this is a valid move, so find out its effect
        square += diceRoll
        square += board[square]
    }
}
print("Game over!")
```

### Checking API Availability 
Swift에서 컴파일러는 프로젝트에서 지정한 배포 대상에서 코드에서 사용된 모든 API를 사용할 수 있는지 확인하고 사용할 수 없는게 존재한다면 컴파일 오류를 발생시킨다.
사용할 API가 런타임에 사용 가능한지 여부에 따라 if문 또는 guard문에서 조건부로 코드 블록을 실행한다.
```swift
if #available(iOS 10, macOS 10.12, *) {
    // Use iOS 10 APIs on iOS, and use macOS 10.12 APIs on macOS
} else {
    // Fall back to earlier iOS and macOS APIs
}
```
첫번째 인자는 플랫폼 이름, 두번째 인자는 버전을 나타낸다. 인수로 전달되는 버전은 최소 배포 버전을 의미한다. 만약 iOS 10이 조건으로 들어온다면 iOS10이상에서만 이용이 가능하다는 뜻이다. (여러 플랫폼, 버전에 따라 다른 논리를 적용하고자 할 때 사용하는 것으로 추측)

## Function

### 튜플 옵셔널 리턴타입
함수의 리턴값으로 튜플 옵셔널을 줄 수 있다. 이는 함수 내부에서 nil값을 줄 수도 있는 경우에 해당한다.
```swift
func minMax(array: [Int]) -> (min: Int, max: Int)? {
    if array.isEmpty { return nil }
    var currentMin = array[0]
    var currentMax = array[0]
    for value in array[1..<array.count] {
        if value < currentMin {
            currentMin = value
        } else if value > currentMax {
            currentMax = value
        }
    }
    return (currentMin, currentMax)
}
```
만약 호출부에서 빈 배열이 들어온다면 array[0]에 접근할 때 런타임에러를 발생시킬 것이다. 이 때문에 빈 배열인지 체크하는 if문이 필요하고 nil를 리턴함으로써 보다 안정적이게 처리할 수 있게 된다. 

개인적인 생각: 함수에서 값을 리턴할 때 발생가능한 값의 형태를 예측하는 것이 매우 중요해보인다. minMax에서 옵셔널을 적용하기 전에는 빈 배열이 들어온다는 예외를 생각하지 못했는데 예시를 통해 안전하게 리턴하는 것이 중요한 부분이라는 것을 알았다. 함수를 작성할 때 옵셔널에 대한 여부를 잊지말고 작성해야겠다.

### 가변 파라미터
가변 파라미터는 인자로 0개 혹은 여러개를 받는다.
가변 파라미터를 포함해 여러개의 파라미터를 받을 때 가변 파라미터 이후에 오는 파라미터는 가변파라미터와 명확하게 구분지어야 하기 때문에 아규먼트 레이블을 사용해야 한다. 


### in-out 파라미터
- 함수의 body 스코프 밖의 영향을 주고 싶을 때 사용한다. 
- 기본값을 가질 수 없고 가변파라미터, 상수, 리터럴을 사용할 수 없다. 

### 중첩함수
중첩함수는 기본적으로 외부로부터 숨겨져 있다. 하지만 여전히 둘러싸고 있는 함수에 의해 사용될 수 있다. 
```swift
func chooseStepFunction(backward: Bool) -> (Int) -> Int {
    func stepForward(input: Int) -> Int { return input + 1 }
    func stepBackward(input: Int) -> Int { return input - 1 }
    return backward ? stepBackward : stepForward
}
var currentValue = -4
let moveNearerToZero = chooseStepFunction(backward: currentValue > 0)
// moveNearerToZero now refers to the nested stepForward() function
while currentValue != 0 {
    print("\(currentValue)... ")
    currentValue = moveNearerToZero(currentValue)
}
print("zero!")
// -4...
// -3...
// -2...
// -1...
// zero!
```

## Enumeration
Enumeration은 연관된 값들을 그룹으로 지어 안전한 방법으로 값을 사용할 수 있게 만드는 타입이다.

### CaseIterable
CaseIterable프로토콜을 채택하면 allCases 속성으로 모든 case들을 Collection으로 나타낼 수 있다.

### Associated Values(연관 값)
Enumeration에서 추가정보를 포함하여 인스턴스를 생성시키고 싶다면 Associated Values(연관값)을 사용할 수 있다.
```swift
switch productBarcode {
case .upc(let numberSystem, let manufacturer, let product, let check):
    print("UPC: \(numberSystem), \(manufacturer), \(product), \(check).")
case .qrCode(let productCode):
    print("QR code: \(productCode).")
}
// Prints "QR code: ABCDEFGHIJKLMNOP."
```

### Raw Values(원시 값)
연관값의 대안으로 미리 동일타입의 기본값을 위치시킴으로써 case에 특정 값을 넣을 수 있다.
```swift
enum ASCIIControlCharacter: Character {
    case tab = "\t"
    case lineFeed = "\n"
    case carriageReturn = "\r"
}
```
원시 값은 위 ASCII 코드처럼 열거를 정의할 때 미리 채워진 값으로 설정된다. 하지만 연관 값은 인스턴스 생성 시점에 값이 채워지는 것이므로 엄격하게 나눠진다.

### 암묵적으로 할당된 원시 값
원시 값을 사용할 때 첫번째 케이스에 기본값을 줘서 그 다음 값들에 대해 암묵적으로 값을 할당할 수 있다.
```swift
enum Planet: Int {
    case mercury = 1, venus, earth, mars, jupiter, saturn, uranus, neptune
}
print(Planet(rawValue: 1)) // output: Optional(Planet.mercury) 
```
venus는 2, earth는 3의 rawValue를 자동으로 갖게 된다.

```swift
enum Planet: String {
    case mercury
    case venus
}
print(Planet.mercury.rawValue) // output: mercury
```
원시 값이 String이라면 rawValue로 접근했을 때 타입이름과 동일하게 갖게 된다. 

Reference
---

- [Swift Language Guide - Control Flow](https://docs.swift.org/swift-book/LanguageGuide/ControlFlow.html)
- [Swift Language Guide - Functions](https://docs.swift.org/swift-book/LanguageGuide/Functions.html)
- [Swift Language Guide - Enumerations](https://docs.swift.org/swift-book/LanguageGuide/Enumerations.html)


###### tags: `캡슐화`, `은닉화`, `Control Flow`, `FUnctions`, `Enumerations`

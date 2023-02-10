230210 클로저(closure), 메모리 누수 디버깅
===
TIL (Today I Learned)
===
학습내용
---
- 클로저 학습하기
- 메모리 누수 디버깅하기

## ✏️ 클로저(Closure)
클로저는 코드에서 전달되고 사용되는 기능 블럭이다. 정의된 컨텍스트에서 모든 상수와 변수에 대한 참조를 캡처하고 저장할 수 있다. 

전역과 중첩 함수는 클로저의 특별한 케이스다. 클로저는 다음의 세가지 형태중 하나를 띈다.
- 전역함수는 이름을 가지고 값을 캡처하지 않는 클로저다.
- 중첩함수는 이름을 가지고 둘러싼 함수로부터 값을 캡처할 수 있는 클로저다.
- 클로저 표현식은 주변 컨텍스트에서 값을 캡쳐할 수 있는 경량 구문으로 이름이 없는 클로저다.

스위프트의 클로저 표현은 깔끔하고 명료한 스타일을 가지고 있고 짧은 문법으로 최적화를 한다.
- 컨텍스트로부터 파리미터와 리턴타입 추론
- 단일표현 클로저로부터 명시적인 리턴
- 아규먼트 레이블 단축
- 트레일링 클로저 문법

### Sorted Method

Swift 기본 라이브러리는 `sorted(by:)`메서드를 지원한다. 이 메서드는 인자로 `(String, String) -> Bool`타입을 갖는 클로저를 갖는다. 

```swift
let names = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]

func backward(_ s1: String, _ s2: String) -> Bool {
    return s1 > s2
}
var reversedNames = names.sorted(by: backward)
```

만약 첫번째 문자열이 두번째 문자열보다 크다면 `backward`는 `true`를 리턴해서 첫번째 문자열이 두번째 문자열보다 앞에 있게 된다.
이렇게 함수자체를 인자로 넣는 것은 상당히 길기에 더 짧은 클로저 문법을 사용해서 개선할 수 있다.

### Closure Expression Syntax
클로저 표현 문법은 다음과 같다.

<img src="https://i.imgur.com/e9L4LZk.png" width="300"/>

클로저 표현의 파라미터는 `in-out`이 될 수 있지만 기본값을 가질 수 없다. 가변 파라미터의 이름을 지정하면 이를 사용할 수 있고 튜플은 파라미터 타입과 반환 타입으로 사용될 수 있다.

아래 예시는 backward의 클로저 표현식 버전이다.
```swift
reversedNames = names.sorted(by: { (s1:String, s2: String) -> Bool in
    return s1 > s2
})
```
기존의 `backward`와 동일하게 파라미터와 반환타입은 존재하지만 이들이 중괄호 안에 존재한다는 차이점이 있다. 
`in`을 기준으로 `in`앞은 클로저 헤드, `in`뒤는 클로저 바디로 불린다.

### Inferring Type From Context
정렬하는 클로저는 함수의 아규먼트로 전달되기 때문에 스위프트는 파라미터와 리턴타입을 추론할 수 있다. `sorted(by:)`메서드는 문자열 배열에서 호출이 되고 이는 타입이 `(String, String) -> Bool`이 되어야만 한다. 
위 사실은 `(String, String)`과 `Bool` 타입을 클로저 표현식 정의에 일부러 작성할 필요가 없음을 의미한다. 모든 타입이 추론이 가능하기 때문에 `->`와 `()`를 생략할 수 있다.

```swift
reversedNames = names.sorted(by: { s1, s2 in return s1 > s2 })
```

### Implicit Return from Single-Expression Closures
단일표현 클로저는 `return`을 생략하고 암시적으로 `return`을 할수 있다. 
```swift
reversedNames = names.sorted(by: { s1, s2 in s1 > s2 })
```
`s1 > s2`는 `Bool`타입을 리턴한다는 것에 모호성이 존재하지 않기 때문에 사용할 수 있다.

### Shorthand Argument Names
스위프트는 인라인 클로저의 아규먼트 이름에 대해 축약구문을 제공한다. 이는 `$0`, `$1`과 같은 이름으로 추론할 수 있다.

만약 이 축약구문을 사용한다면 클로저의 아규먼트 리스트를 생략해야 한다. 즉 위 `sorted`에서 `s1, s2`부분을 제거해야 한다. 가장 높은 숫자의 축약 아규먼트는 클로저가 갖는 아규먼트의 수를 결정한다. `in`키워드는 생략이 되면서 구문이 다음과 같이 변한다.
```swift
reversedNames = names.sorted(by: { $0 > $1 })
```
`$1`가 가장 큰 숫자를 받은 아규먼트 인자기 때문에 클로저는 두개의 아규먼트를 가지고 있다고 이해한다.

### Operator Methods
Swift의 `String`타입은 보다 큰 연산자`>`의 문자열 별 구현을 `sorted(by:)` 메서드에 필요한 메서드 타입과 정확하게 일치하기 떄문에 다음처럼 사용할 수 있다.
```swift
reversedNames = names.sorted(by: >)
```

### Trailing Closures
만약 클로저 표현식이 마지막 인자면서 너무 길다면 후행 클로저 문법을 사용할 수 있다. 후행 클로저 문법을 사용하면 클로저 인자 라벨을 작성하지 않아도 된다. 
```swift
func someFunctionTahatTakesAClosure(closure: () -> Void) {  
    
}
// 후행 클로저 전
someFunctionTahatTakesAClosure(closure: {
    
})
                                    
// 후행 클로저 후
someFunctionTahatTakesAClosure() {
    
}                                                                 
```

`sorted(by:)`를 후행 클로저 문법을 이용하면 다음과 같다.
```swift
reversedNames = names.sorted() { $0 > $1 }
```
만약 클로저 표현식이 함수 또는 메서드의 유일한 인자들이라면 소괄호를 지울 수 있다.

```swift
reversedNames = names.sorted { $0 > $1 }
```

`map`을 이용한 후행 클로저 문법 예시를 더 살펴보자.
```swift
let digitNames = [
    0: "Zero", 1: "One", 2: "Two",   3: "Three", 4: "Four",
    5: "Five", 6: "Six", 7: "Seven", 8: "Eight", 9: "Nine"
]
let numbers = [16, 58, 510]

let strings = number.map { (number) -> String in
  var number = number
  var output = ""
  repeat {
      output = digitNames[number % 10]! + ouput
      number /= 10
  } while number > 0
  return output
}
```
`map`메서드는 `array`의 각 요소마다 한번씩 클로저를 호출한다. 파라미터는 추론이 가능하기 때문에 `number`를 명시할 필요는 없다.

만약 함수가 여러개의 클로저를 가지고 있다면 첫번째 후행 클로저의 라벨을 생략하고 남아있는 후행 클로저에는 라벨을 붙인다. 

```swift
func loadPicture(from server: Server, completion: (Picture) -> Void, onFailure: () -> Void) {
    if let picture = download("photo.jpg", from: server) {
        completion(picture)
    } else {
        onFailure()
    }
}

loadPicture(from: someServer) { picture in 
    someView.currentPiture = pictrue
} onFailure: {
    print("Couldn't download the next picture.")
}
```

### Capturing Values
클로저는 특정 문맥의 상수나 변수를 캡처할 수 있다. 다시말해 원본 scope가 사라져도 클로저의 body안에서 그 값을 활용할 수 있다는 뜻이다. 

스위프트에서 값을 캡처하는 가장 간단한 형태는 중첩함수이다.
중첩함수는 외부의 함수 아규먼트들과 외부 함수에서 정의된 상수나 변수들을 캡처할 수 있다.

`makeIncrement`는 `increment`라는 중첩함수를 반환한다. 중첩함수에서는 `runningTotal`, `amount`라는 외부 컨텍스트의 값을 사용하고 있다.

```swift
func makeIncrementer(forIncrement amount: Int) -> () -> Int {
    var runningTotal = 0
    func incrementer() -> Int {
        runningTotal += amount
        return runningTotal
    }
    return incrementer
}
```

중첩된 함수만 따로 살펴보자.
```swift
func increment() -> Int {
    runningTotal += amount
    return runningTotal
}
```

`increment`함수는 파라미터를 가지지 않는다. 그러나 이 함수에서 `runningTotal`과 `amount`를 캡처하고 있기 때문에 참조할 수 있다. 참조로 캡처하는 것은 `makeIncrement`가 끝날때 `runningTotal`과 `amount`가 사라지지 않게 한다.그리고 다음 `increment`호출에서 `runningTotal`을 이용할 수 있게 한다.

### 👏 Note
최적화이유로 스위프트는 클로저에서 값이 사용되지 않는경우 값의 복사본을 저장하고 캡처한다.  그리고 더이상 사용되지 않는 변수를 처리하는 메모리관리를 다룬다.

`makeIncrementer`의 사용예시를 살펴보자.
`incrementByTen`은 호출할 때마다 `runningTotal`을 10씩 증가시키는 클로저를 가리키고, `incrementBySeven`은 `runningTotal`을 7씩 증가시키는 다른 클로저를 가리킨다.

```swift
let incrementByTen = makeIncrementer(forIncrement: 10)
incrementByTen()
// returns a value of 10
incrementByTen()
// returns a value of 20
incrementByTen()
// returns a value of 30

let incrementBySeven = makeIncrementer(forIncrement: 7)
incrementBySeven()
// returns a value of 7
```
<img src="https://i.imgur.com/V5v06c7.png" width="400"/><br/><br/>


클로저는 실행시점에서 힙 영역에 생기며 컨텍스트에 있는 변수나 상수들을 캡처한다. 이때 캡처는 `reference` 캡처이기 때문에 만약 위 예시 중첩함수 외부에서 값을 수정했다면 수정한 값을 캡처하게 된다.

```swift
func makeIncrementer(forIncrement amount: Int) -> () -> Int {
    var runningTotal = 0
    func incrementer() -> Int {
        runningTotal += amount
        return runningTotal
    }
    runningTotal = 100
    return incrementer
}

let incrementByTen = makeIncrementer(forIncrement: 10)

incrementByTen() // 110
```
### 👏 Note
만약 클로저를 클래스 인스턴스의 프로퍼티로 할당한다면 클로저는 인스턴스를 가리키게 된다. 그리고 클로저에서 인스턴스의 프로퍼티를 사용한다면 서로를 가리키는 상황이 발생해 강한순환참조가 일어날 수 있다. 이를 해결하기 위해 캡처리스트와 `weak` 또는 `unowned`를 이용해야 한다. 

### Closures Are Reference Types
앞의 예제에서 `incrementBySeven`과 `incrementByTen`은 상수지만 함수와 클로저는 참조 타입이기 때문에 `runningTotal`을 증가시킬 수 있다. 

클로저나 함수를 상수나 변수에 할당할 때마다 실제로 참조를 할당하는 것이지 실제 클로저의 내용을 담는 것이 아니다.(이는 값 타입)

이는 같은 클로저를 두 개의 상수나 변수에 할당했을 경우 둘 다 같은 클로저를 바라보고 있다는 의미이다.

```
let alsoIncrementByTen = incrementByTen
alsoIncrementByTen()
// returns a value of 50

incrementByTen()
// returns a value of 60
```

### Escaping Closures
클로저가 함수의 파라미터로 전달되고 그 함수가 종료된 이후에 사용을 하거나 변수나 상수에 저장하려고 한다면 클로저 타입 앞에 `@escaping` 을 추가해줘야 한다.

예를 들어 비동기 작업을 하는 많은 함수들은 `completionHandler`를 가지고 있다. 그리고 이를 변수에 담을 때는, 즉 함수가 종료된 이후에 사용하려고 변수에 담을 때는 `@escaping`을 붙여야 한다.

```swift
var completionHandlers: [() -> Void] = []
func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
    completionHandlers.append(completionHandler)
}
```

만약 클래스의 인스턴스를 self가 참조할 경우`escaping` 클로저는 특별히 신경을 써야 한다. `escaping` 클로저 안에서 `self`를 참조한다는 것은 클로저와 클래스 인스턴스간의 강한 순환참조를 발생시킬 수 있기 때문이다. (`non-escaping`은 클로저가 실행된 후 메모리에서 해제되기 때문에 고려하지 않는다)

보통 클로저는 클로저의 바디 안에서 값을 사용함으로써 암묵적으로 캡처를 한다. 하지만 `escaping`을 한 경우에는 `self`를 캡처리스트에 포함시키거나 사용하는 값 앞에 `self`를 붙여야 한다. (클로저가 함수 종료이후에도 지속적으로 살아있기 때문에 self를 참조하고 있어야 하기 때문). 

정리하자면 인자로 전달받은 클로저를 변수나 상수에 저장하거나 함수 종료이후에 실행시키고 싶다면 `@escaping`을 적어야 하고 탈출된 클로저에서는 아직 실행하지 않은 상태이기 때문에 메모리에 올라가지 않았고 다른 문맥에서 사용되기 때문에 `self`를 명시해줘야 값을 캡처 할 수 있다.

```swift
func someFunctionWithNonescapingClosure(closure: () -> Void) {
    closure()
}

class SomeClass {
    var x = 10
    func doSomething() {
        someFunctionWithEscapingClosure { self.x = 100 }
        someFunctionWithNonescapingClosure { x = 200 }
    }
}

let instance = SomeClass()
instance.doSomething()
print(instance.x) // "200"

completionHandlers.first?()
print(instance.x) // "100"
```

클로저 캡처리스트를 이용하는 방법도 있다.
```swift
class SomeOtherClass {
    var x = 10
    func doSomething() {
        someFunctionWithEscapingClosure { [self] in x = 100 }
        someFunctionWithNonescapingClosure { x = 200 }
    }
}
```

만약 `self`가 구조체나 열거형과같은 인스턴스라면 `self`를 항상 암묵적으로 참조할 수 있다. 하지만 `escaping` 클로저는 만약 `self`가 값타입인 경우 참조를 캡처할 수 없다.
```swift
struct SomeStruct {
    var x = 10
    mutating func doSomething() {
        someFunctionWithNonescapingClosure { x = 200 }  // Ok
        someFunctionWithEscapingClosure { x = 100 }     // Error
    }
}
```

`mutating` 키워드 내부에서는 `self`가 변할 수 있기 때문에 `escaping`클로저에서는 변할 수 있는 가능성을 가진 `self`를 참조할 수 없게 되는 것이다.

### Autoclosures
자동클로저는 인자 값이 없으며 특정 표현을 감싸서 다른 함수에 전달인자로 사용할 수 있는 클로저이다. 어떤 아규먼트도 가지지 않고 호출될 때 그 클로저안에 감싸져있는 표현식을 반환한다. 이 문법은 함수 파라미터의 소괄호를 생략해서 사용해 편리하다.

자동 클로저는 클로저를 호출할때까지 안쪽 코드를 실행시키지 않기 때문에 연산을 지연시킨다. 그렇기에 실제 계산이 필요할 때 호출되기 때문에 복잡한 연산을 하는데 유용하다.

```swift
var customersInLine = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]
print(customersInLine.count)
// Prints "5"

let customerProvider = { customersInLine.remove(at: 0) }
print(customersInLine.count)
// Prints "5"

print("Now serving \(customerProvider())!")
// Prints "Now serving Chris!"
print(customersInLine.count)
// Prints "4"
```

`customProvider`의 타입은 `String`이 아니라 `()->String`으로 문자열을 반환하는 매개변수가 없는 함수이다.

다음은 자동클로저를 함수의 인자 값으로 넣는 예시이다.
```swift
// customersInLine is ["Ewa", "Barry", "Daniella"]
func serve(customer customerProvider: @autoclosure () -> String) {
    print("Now serving \(customerProvider())!")
}
serve(customer: customersInLine.remove(at: 0))
// Prints "Now serving Ewa!"
```

함수의 클로저 인자에 `@autoclosure`를 붙임으로써 함수 호출시 자동으로 클로저로 변환하게 만든다. 즉 함수가 이미 클로저 인것을 알기 때문에 `String`형태의 값만 넣어도 되는 것이다.


## ✏️ 메모리 누수 디버깅하기
프로젝트를 진행하다가 문득 클로저에서 `self`를 참조하는 경우 꼭`weak`를 사용해야 하는가에 의문이 들었다. 결론부터 말하자면 현 상태에서는 참조가 일어나지않아도 추후 참조가 일어날 수 있는 가능성이 있기 때문에 `weak`를 붙여주는 게 좋다. 

하지만 학습을 하고 있는 상태에서 어떤 상태에서 순환참조가 발생하고 안하는지를 확실하게 알고 싶었다. 

### po CFGetRetainCount(someVariable)
`CFGetRetainCount`에 변수명을 사용하면 변수를 참조하고있는 참조카운트를 셀 수 있다. 하지만 이 경우에는 클로저의 레퍼런스 카운팅을 알지는 못했다.

### Memory Graph Debugger
Xcode의 Debug Navigator에서 Memory를 선택하면 현재 앱 메모리의 Footprint를 빠르게 볼 수 있다.

<img src="https://i.imgur.com/NmP7vib.png" width="600"/><br/>


그리고 Xcode의 이 버튼을 누르면

<img src="https://i.imgur.com/qpGKTnk.png" width="400"/>

현재 상태에서 메모리 snapshot을 찍은 뒤, 이 snapshot의 메모리 정보를 보여주는 Memory Graph Debugger가 나타난다. 

왼쪽 패널에는 현재 메모리에 적재되어 있는 객체들과 해당 클래스의 인스턴스 숫자들, 각 인스턴스들의 주소 목록을 보여준다.


<img src="https://i.imgur.com/EWjbwKn.png" width="200"/>

왼쪽 패널에서 객체들 중 하나를 선택하면 해당 객체를 메모리에 유지하도록 하는 레퍼런스들, 즉 해당 객체를 가리키고 있는 개체들이 그래프로 나타난다.

<img src="https://i.imgur.com/7ApkeFg.png" width="400"/><br/>

위 그림에서 진한 선은 strong 레퍼런스를 연한 선은 unknown(weak, strong)을 나타낸다. 

메모리 누수를 확인하고 싶은 객체를 클릭한 뒤 그 객체를 더블클릭하면 참조하고 있는 객체가 나타난다. 현 프로젝트에서는 클로저가 UILabel을 가리키고 UILabel또한 클로저를 가리키는지 확인을 하려고 디버깅을 했는데 객체들의 관계를 보니 강한순환참조가 일어나지는 않았다.

<img src="https://i.imgur.com/QdiW3tB.png" width="600"/><br/>






### 참고자료
- [Swift org - closure](https://docs.swift.org/swift-book/LanguageGuide/Closures.html)
- [메모리 릭 찾기](https://seizze.github.io/2019/12/20/iOS-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EB%9C%AF%EC%96%B4%EB%B3%B4%EA%B8%B0,-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EC%9D%B4%EC%8A%88-%EB%94%94%EB%B2%84%EA%B9%85%ED%95%98%EA%B8%B0,-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EB%A6%AD-%EC%B0%BE%EA%B8%B0.html)

230112 Navigation in Modal, 싱글톤 static let, ARC(Automatic Reference Counting)
===
TIL (Today I Learned)
===
학습내용
---
- Navigation in Modal
- 싱글톤의 shared는 왜 static let을 사용해야 할까?
- ARC 학습하기

## 고민한 점 / 해결 방법

## Navigation in Modal
- 네비게이션이 있는 모달을 띄워주려고 했는데 계속 네비게이션이 적용되지 않은 화면만 나왔다. 처음에는 iOS버전 문제인줄 알았지만 결국 네비게이션 컨트롤러도 뷰 컨트롤러라는 것을 깨닫고 네비게이션 컨트롤러 자체를 모달로 줘서 문제를 해결했다. 

## 싱글톤의 shared는 왜 static let을 사용해야 할까?
- 기본적으로 싱글톤은 앱 전역에서 접근할 수 있어야 하기 때문에 static을 사용한다. swift에서 static은 사용 시점에 초기화(lazy)되기 때문에 싱글톤 인스턴스가 최초 생성되기 전까지는 메모리에 올라가지 않는다. 이 때문에 Instance가 여러 개 생성되지 않는 Thread-Safe한 방법이 된다.
#### 그럼 let은 왜 필요하지?
- static은 "싱글톤 생성"에 한정해 Thread-Safe한 것이다. 즉 var로 되어있다면 다른 객체로 변환될 수 있는 가능성이 있기 때문에 유일하다고 볼 수 없다. 이 때문에 static let으로 shared를 선언해야 안전하게 싱글톤 객체를 만들 수 있다.

## ARC
> Swift의 ARC는 앱의 메모리를 자동으로 추적하고 관리한다. 하지만 특정상황에서 ARC는 코드 사이의 관계에 대한 정보가 필요하다. 그렇기 때문에 개발자는 ARC의 작동방식을 알아야 한다.
- ARC는 참조타입에 대해서만 지원한다.

### ARC는 어떻게 동작하는가?
클래스의 인스턴스를 만들면 ARC는 타입에 대한 정보와 값에 대한 정보를 담은 메모리 청크를 할당한다. 그리고 인스턴스가 더이상 필요로하지 않을 때 자동으로 메모리를 해제한다. 
하지만 만약 여전히 사용중인 인스턴스를 메모리 해제한다면 더이상 그 인스턴스의 속성과 메서드에 접근할 수 없게되고 만약 접근한다면 앱은 crash가 나게된다.

인스턴스가 필요한 경우 이 인스턴스가 사라지지 않게 하기위해 ARC는 active reference가 1개이상이라면 메모리해제를 하지 않도록 한다. 

클래스에 프로퍼티, 변수, 상수에 할당하게 된다면 강한 참조가 일어나게 된다. 이는 해당 인스턴스를 확실하게 유지하고 이런 참조가 유지되는 동안에는 할당해제를 허용하지 않는다.

### ARC Action
```swift
class Person {
    let name: String
    init(name: String) {
        self.name = name
        print("\(name) is being initialized")
    }
    deinit {
        print("\(name) is being deinitialized")
    }
}

var reference1: Person?
var reference2: Person?

reference1 = Person(name: "John Appleseed") // Person instance Reference Counting: 1
reference2 = reference1 // Person instance Reference Counting: 2

reference1 = nil // Person instance Reference Counting: 1
// reference2가 아직 강하게 참조중이여서 instance가 메모리에서 내려가지 않음
reference2 = nil // Person instance Reference Counting: 0, deallocate

```

### 클래스 인스턴스 간 강한참조사이클
```swift
class Person {
    let name: String
    init(name: String) { self.name = name }
    var apartment: Apartment?
    deinit { print("\(name) is being deinitialized") }
}

class Apartment {
    let unit: String
    init(unit: String) { self.unit = unit }
    var tenant: Person?
    deinit { print("Apartment \(unit) is being deinitialized") }
}
var john: Person?
var unit4A: Apartment?

john = Person(name: "John Applesssed")
unitA = Apartment(unit: "4A")
```
- 위 상황에서는 john과 unitA가 인스턴스 하나만 참조하고 있다. 구조를 살펴보면 아래와 같다.
<img src="https://i.imgur.com/2adCdey.png" width="500"/>
- 만약 john과 unitA변수에 nil을 할당한다면 ARC는 자동으로 deallocate할 것이다.
- 하지만 다음의 코드를 추가하게 되면 문제가 생긴다.


```swift
john!.apartment = unit4A
unit4A!.tenant = john
```
- john이 apartment를 참조하고 있기 때문에 Apartment 인스턴스를 reference counting이 2가 된다. 그리고 unit도 마찬가지로 reference counting이 2가 된다.
<img src="https://i.imgur.com/UjzwEVc.png" width="500"/>
- 만약 john과 unitA변수에 nil을 할당하게 된다면 인스턴스들이 서로를 가리키고 있기 때문에 메모리가 deallocate되지 않는다. 이 때문에 강한참조사이클을 해결하는 어떠한 방법이 필요하다.

### 강한참조 사이클을 해결하는 방법
> 강한참조 사이클을 해결하는 방법으로 weak, unowned키워드가 존재한다. 이들은 인스턴스를 강하게 참조하고 있지 않더라도 가리키도록 할 수 있다.

- 다른 인스턴스가 더 짧은 생명주기를 가질 때 약한참조를 사용한다. 위 Apartment 예시에서 apartment는 tenant을 가지지 않을 수 있기 때문에 weak을 사용하는 게 적절하다. 만약 다른 인스턴스가 동일하거나 더 긴 생명주기를 가진다면 unowned를 사용한다.

### 약한 참조
- 약한 참조는 인스턴스를 강하게 가리키고 있는 것이 아니라서 weak reference가 여전히 인스턴스를 가리키고 있더라도 deallocate될 수 있다. 그러므로 ARC는 deallocate될 때 weak reference를 nil로 바꿔준다. 그리고 nil로 바뀔 수 있기 때문에 var로 선언해서 바뀔 수 있도록 선언해야 한다.
> ARC에서 약한 참조에 nil을 할당하면 프로퍼티 옵저버는 실행되지 않는다.
```swift
class Apartment {
    let unit: String
    init(unit: String) { self.unit = unit }
    weak var tenant: Person?
    deinit { print("Apartment \(unit) is being deinitialized") }
}
```
- Person 클래스는 동일하게 하고 Apartment의 tanant를 weak참조로 바꿔보자.
<img src="https://i.imgur.com/ykhxA59.png" width="500"/>
- Apartment는 weak하게 참조하기 때문에 얇은 선으로 두었다.
- 이 상황에서 Person을 가리키는 변수에 nil을 할당하면 더 이상 강한참조로 Person인스턴스를 가리키는 것이 없어져 순환참조가 생기지 않는다.
<img src="https://i.imgur.com/bXVlwqh.png" width="500"/>

### 미소유 참조
> weak reference와 다르게 항상 값이 있음을 기대할 때 사용한다. ARC는 unowned reference의 값을 절대 nil로 바꾸지 않는다.
- reference는 항상 메모리가 해제되지않는 인스턴스에 대해서 가리킨다. 만약 메모리가 해제된 인스턴스에 접근할 경우 crash가 일어난다. 
```swift
class Customer {
    let name: String
    var card: CreditCard?
    init(name: String) {
        self.name = name
    }
    deinit { print("\(name) is being deinitialized") }
}

class CreditCard {
    let number: UInt64
    unowned let customer: Customer
    init(number: UInt64, customer: Customer) {
        self.number = number
        self.customer = customer
    }
    deinit { print("Card #\(number) is being deinitialized") }
}
```
- Customer은 신용카드를 갖고 있을 수도 없을 수도 있기 때문에 옵셔널으로 두었고 CreditCard는 반드시 customer가 존재해야 하므로 unowned로 두었다. unowned로 둔 이유는 CreditCard입장에서 Customer는 반드시 수명주기가 더 크기 때문이다. 

#### 음.. 라이프사이클이 더 긴 경우에 unowned를 사용하는 건 알겠는데 그냥 weak으로 대체하면 안될까? 
- 둘은 성능차이가 있다고 한다. weak 참조의 경우 어떤 인스턴스 A에 대해 새로운 weak참조가 생길 때마다, 인스턴스에 대한 모든 weak 참조들이 저장된 테이블에 등록되어야 한다. 만약 어떤 곳에서도 A를 강하게 참조하고 있는 것이 아니라면 이는 메모리에서 해제된다. 하지만 해제되기전에 table에 저장되어있는 weak reference들을 nil로 만들어줘야 한다. 이를 **zeroing weak**라고 한다. (더 깊게 들어가는 건 이해가 잘 안 된다..[weak vs unowned](https://velog.io/@hojonge/weak-vs-unowned-Swift)) 결론적으로 라이프사이클이 더 길다고 확신할 수 있는 경우에는 unowned로 참조하면 된다.

#### 미소유 옵셔널 참조
- Swift5이후부터 지원하는 미소유 옵셔널 참조.. 솔직히 왜 사용하는지 이해가 잘 가지 않는다. 메모리가 해제될 때 nil을 할당하는 weak와는 다르게 nil을 할당하지는 않지만 옵셔널의 형태를 가져야하고 확실한 시점임을 보장하는 상태에서 사용하는 unowned의 성격이라 보기에는 옵셔널로 되어있어 crash가 안나고... 언제 사용해야 할지 모르겠다. 생명주기가 더 길면서 weak table을 사용하지 않아 성능을 높이려는 애플의 노력일까
- 다음 예시를 통해 조금은 친숙해져보자.
```swift
class Department {
    var name: String
    var courses: [Course]
    init(name: String) {
        self.name = name
        self.courses = []
    }
}

class Course {
    var name: String
    unowned var department: Department
    unowned var nextCourse: Course?
    init(name: String, in department: Department) {
        self.name = name
        self.department = department
        self.nextCourse = nil
    }
}

let department = Department(name: "Horticulture")

let intro = Course(name: "Survey of Plants", in: department)
let intermediate = Course(name: "Growing Common Herbs", in: department)
let advanced = Course(name: "Caring for Tropical Plants", in: department)

intro.nextCourse = intermediate
intermediate.nextCourse = advanced
department.courses = [intro, intermediate, advanced]

department.courses[1] = nil
intro.nextCourse = nil
intermediate = Course(name: "Survey of Plants", in: department)
```

<img src="https://i.imgur.com/JLHLgfF.png" width="700"/><br>

- Course에서 department는 반드시 존재해야 하므로 unowned로 선언하는 건 오케이
- Course에서 nextCourse가 만약 strong참조를 하게 된다면(여기선 intro가 intermediate을 strong참조했다고 가정)Department intance의 course를 nil로 하고 Department의 해당 course를 nil로 하더라도 intermediate인스턴스는 메모리가 해제되지 않는다. 그러므로 strong은 부적절하다.
- 바로 위 상황에서 nextCourse에 weak reference를 사용하게 된다면 nextCourse에 직접 nil을 주지 않더라도 intermediate와 department.courses에만 nil을 주면 자동으로 내려간다. 하지만 현재 course가 nextCourse보다 반드시 생명주기가 많은 것은 아니기 때문에 weak이 아주 적절해보이지는 않는다.
- unowned optional이 아닌 unowned의 경우에는 intermediate가 nil이 되었을 때 
intro.nextCourse는 nil로 변환이 되지않아 접근했을 때 runtime error가 일어난다. 때문에 부적절
- unowned optional을 사용하면 현재 참조하고 있는 인스턴스가 사라졌을 때 nil을 명시적으로 줄 수 있어 런타임에러를 막을 수 있게 한다. (그 이후는 이해가 잘 안된다.. 나중에 다시 한 번 봐야겠다.[Unowned Optional reference](https://zeddios.tistory.com/1214))

### 클로저에서의 강한 참조 순환
- 강한 참조 순환은 변수 간 뿐만 아니라 클로저와 관계돼서 발생할 수도 있다. 왜냐하면 self를 참조하기 때문이다. 여기서 self를 참조한다는 것은 다음과 같다. 
```swift
class HTMLElement {
    let name: String
    lazy var asHTML: () -> String = {
        return self.name
    }
}
```
- 클로저 내부에서 HTMLElement의 name프로퍼티를 사용하기 위해 self가 필요로 하는데 이를 self를 참조한다고 한다.(클로저가 HTMLElement를 바라보고 있다.)
- 클로저는 이 강한 순환 참조를 해결하기 위해 클로저 캡쳐 리스트를 사용한다. 
```swift
class HTMLElement {
    let name: String
    let text: String?
    lazy var asHTML: () -> String = {
        if let text = self.text {
            return "<\(self.name)>\(text)</\(self.name)>"
        } else {
            return "<\(self.name) />"
        }
    }
    init(name: String, text: String? = nil) {
        self.name = name
        self.text = text
    }
    deinit {
        print("\(name) is being deinitialized")
    }
}

let heading = HTMLElement(name: "h1", text: "hi")
let defaultText = "some default text"

print(heading.asHTML())
// Prints "<h1>hi</h1>"
```
> asHTML 클로저는 지연 프로퍼티로 선언되었다. 왜냐하면 HTML을 렌더링 하기 위해 필요한 태그나 텍스트가 미리 준비되어있어야 하기 때문. 또 지연 프로퍼티이기 때문에 프로퍼티 안에서 self참조가 가능하다.
```swift
var paragraph: HTMLElement? = HTMLElement(name: "p", text: "hello, world")
print(paragraph!.asHTML())
// Prints "<p>hello, world</p>"
```
- paragraph를 HTMLElement?로 설정해서 nil할당이 가능하게끔 만든 후 paragraph!.asHTML()을 통해 클로저를 참조하게 만든다. 그리고 클로저는 self를 참조하고 있기 때문에 HTMLElement 인스턴스를 강하게 가리킨다.
<img src="https://i.imgur.com/gxsOtS2.png" width="500"/>
> self를 여러번 참조하더라도 실제로는 단 한 번만 강하게 참조한다.

### 클로저에서 강한 참조 순환 문제의 해결
- 클로저에서 강한 참조 순환 문제를 해결하기 위해 캡쳐 참조에 강한 참조 대신 약한 참조나 미소유 참조를 지정할 수 있다. 
> Swift에서는 클로저에서 특정 self의 메소드를 사용할 때 캡쳐 실수를 막기 위해 self를 명시하는 것을 필요로 한다.

### 캡쳐리스트 정의
- 클로저의 파라미터 앞에 소괄호([])를 넣고 그 안에 캡쳐 대상에 대한 참조 타입을 적어준다.
```swift
lazy var someClosure: (Int, String) -> String = {
    [unowned self, weak delegate = self.delegate!] (index: Int, stringToProcess: String) -> String in
    // closure body goes here
}
```
- ...?굉장히 어색한 문법
- 클로저의 파라미터가 없고 반환 값이 추론에 의해 생략이 가능하다면 캡쳐리스트 정의를 in 앞에 적어준다.
```swift
lazy var someClosure: () -> String = {
    [unowned self, weak delegate = self.delegate!] in
    // closure body goes here
}
```
> 만약 캡쳐리스트가 절대 nil이 될 수 없다면 그것은 반드시 약한 참조 리스트가 아니라 미소유 참조 리스트로 캡쳐돼야 합니다. (Apple 공식문서)
- 꼭 반드시일까요..? 참고하겠습니다 ㅎㅎ
```swift
class HTMLElement {
    let name: String
    let text: String?
    lazy var asHTML: () -> String = {
        [unowned self] in
        if let text = self.text {
            return "<\(self.name)>\(text)</\(self.name)>"
        } else {
            return "<\(self.name) />"
        }
    }
    init(name: String, text: String? = nil) {
        self.name = name
        self.text = text
    }
    deinit {
        print("\(name) is being deinitialized")
    }
}

var paragraph: HTMLElement? = HTMLElement(name: "p", text: "hello, world")
print(paragraph!.asHTML())
// Prints "<p>hello, world</p>"
```
- unowned로 캡쳐리스트를 참조하고 있기 때문에 reference counting이 증가하지 않는다.
<img src="https://i.imgur.com/LX3cUdG.png" width="500"/>



### Reference
- [Apple Developer Documentation - ARC](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html)
- [Swift Language Guide - ARC](https://jusung.gitbook.io/the-swift-language-guide/language-guide/23-automatic-reference-counting#unowned-references-and-implicitly-unwrapped-optional-properties)
- [싱글톤 static let](https://lietenant-k.tistory.com/42)

###### tag: `Navigation in Modal`, `싱글톤`, `static let`, `ARC`, `클로저`, `weak`, `unowned`

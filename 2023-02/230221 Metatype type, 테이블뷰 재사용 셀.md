230221 Metatype type, 테이블뷰 재사용 셀
TIL (Today I Learned)
===
학습내용
---
- Metatype type 학습하기
- 테이블뷰 재사용 셀


## Metatype type 학습하기

메타타입 타입은 클래스, 구조체, 열거형, 프로토콜타입과같은 어떤 타입의 타입이다. 

이러한 메타타입은 타입이름 뒤에 `.Type`이 붙는다. 프로토콜 메타타입은 런타임에 그 프로토콜을 따르는 구체적인 타입이 아니기 때문에 `.Protocol`이 붙는다. 

예를 들어 `SomeClass`의 메타타입은 `SomeClass.Type`이고 `SomeProtocol`의 메타타입은 `SomeProtocol.Protocol`인 셈이다.

뒤에 `self`를 붙여서 타입을 값으로 사용할 수도 있다. 예를 들어 `SomeClass.self`는 `SomeClass` 자체를 반환하고 `Someprotocol`은 런타임에 프로토콜을 채택하는 `SomeProtocol`의 인스턴스가 아닌 `SomeProtocol`그 자체를 반환한다. 
`type(of:)`메서드를 호출해서 런타임에 인스턴스의 타입에 접근할 수도 있다. 타입 자체에 접근하는 것이기 떄문에 `SomeSubClass`가 출력된다.

```swift
class SomeBaseClass {
    class func printClassName() {
        print("SomeBaseClass")
    }
}
class SomeSubClass: SomeBaseClass {
    override class func printClassName() {
        print("SomeSubClass")
    }
}
let someInstance: SomeBaseClass = SomeSubClass()
// The compile-time type of someInstance is SomeBaseClass,
// and the runtime type of someInstance is SomeSubClass
type(of: someInstance).printClassName()
// Prints "SomeSubClass"
```

메타타입 값으로부터 이니셜라이저 표현실을 사용해 인스턴스를 만든다.

```swift
class AnotherSubClass: SomeBaseClass {
    let string: String
    required init(string: String) {
        self.string = string
    }
    override class func printClassName() {
        print("AnotherSubClass")
    }
}
let metatype: AnotherSubClass.Type = AnotherSubClass.self
let anotherInstance = metatype.init(string: "some string")
```

### ✏️ 메타타입 추가정리

### 메타타입
`type(of:` 메서드는 해당 타입의 인스턴스가 아닌 해당 타입 자체를 저장할 수 있다. 이때 이 타입을 메타타입으라고 부른다. 

```swift
struct Human {
    static let name = "steven"
}
let stevenType = type(of: steven) // Human.Type
stevenType.name // steven
```

메타타입도 결국 타입이기 때문에 타입 어노테이션을할 때 명시할 방법이 필요하다. 이는 위의 주석에 표시한 것처럼 `Type`이라고 표현한다. 

즉 `Human.Type`은 메타타입이며 하나의 자료형이라고 볼 수 있는 것이다. 

### 언제 사용할까?
제네릭 함수와 사용할 때 타입에 따라 다른 작업을 하고싶을 때 사용한다.

```swift
protocol Human {
    var job: String { get set }
    init(_ job: String)
}
struct Teacher: Human {
    var job: String
    
    init(_ job: String) {
        self.job = job
    }
}
struct Student: Human {
    var job: String
    
    init(_ job: String) {
        self.job = job
    }
}

위와 같은 예시가 있을 때 메타타입에 따라서 다른 인스턴스를 생성시킬 수 있다. (보통 인스턴스를 생성시킬때 사용하는 것 같다.)

```swift
func create<T: Human>(type: T.Type) -> T {
    switch type {
        case is Teacher.Type:
            return T.init("teacher")
        case is Student.Type:
            return T.init('student')
        default:
            fatalError("x")
    }
}
```

### 메타타입의 값은 어떻게 얻어낼까?
`self`를 사용해서 얻어낼 수 있다. 인스턴스에서도 `self`를 사용하는데 이 `self`와는 다른 타입의 `self`이다. 

만약 위 `create`함수를 사용한다면 메타타입인 `create(type: Teacher.Type)`
을 넣어주는 것이 아닌 `create(type: Teacher.self)`를 넣어줘야 한다.

### static Metatype, Dynamic Metatype

`self`를 이용해서 만드는 메타타입은 Static Metatype이고, 
`type(of:)`를 이용해서 만드는 것은 Dynamic Metatype이다.


### 프로토콜의 메타타입

```swift
protocol Human { }
let human: Human.Type = Human.self // error
```

변수 `human`의 타입 `Human.Type`은 프로토콜의 타입을 의미하는 것이 아닌 프로토콜을 채택하는 타입을 의미한다. 하지만 `Human.self`는 실제 프로토콜의 타입을 지칭하기 떄문에 두 개의 타입이 맞지않는 것이다.

만약 `Human.Type`을 타입으로 사용했을때 다음과 같이 사용하면 에러가 나지 않는다.

```swift
struct Steven: Human { }
let stevenType: Human.Type = Steven.self // Steven.Type
```

변수 `stevenType`의 타입과 `Steven.self`가 지칭하는 것이 동일하기 때문에 위 코드는 에러가 나지않는다. `Human.Type`의 경우 해당 프로토콜을 따르는 Steven의 메타타입을 얘기하는 것이기 때문인 것이다. 이를 `existential metatype`이라고 부른다고 한다.

따라서 만약 진짜 `Human`이란 프로토콜의 메타타입을 얻고싶다면 `.Protocol`을 사용하면 도니다.

```swift
protocol Human { }
let humanType: Human.Protocol = Human.self // Human.Protocol
```

### self, .self, Self

self는 인스턴스에서 생성되고 해당 인스턴스를 나타낸다.
.self는 메타타입의 값을 나타낸다.
Self는 현재 속한 코드의 Type이 Self의 Type이 된다.

```swift
extension Int {
    static let zero: Self = 0
    func makeZero() -> Self {
        return Self()
    }
}
```

`Int`에 속해있으니 `Self`는 `Int`가 되는 것이다.

`Self`는 타입에 의존하지 않는 코드를 작성할 수 있다는 장점에서 사용된다.

## 테이블뷰 재사용 셀

### ✏️ reuseIdentifier
> 재사용가능한 셀을 식별하는 문자열
```swift
var reuseIdentifier: String? { get }
```

reuse Identifier는 테이블 뷰의 델리게이트가 만드는 `UITableViewCell`과 관련이 있다. 이는 셀 객체에 `init(frame:reuseIdentifier:)`에 할당이 되고 그 후에는 변경할 수 없다. `UITableView` 객체는 재사용가능한 셀들을 담은 큐를 가지고 있으며 이 셀들은 각각 reuse Identifier를 가지고 있다. 그리고 이들은 `dequeueReuseableCell(withIdentifier:)` 메서드에서 사용된다.

### 테이블 뷰 재사용에 따른 문제 해결방법.
위에서 테이블뷰는 셀을 재사용한다는 것을 학습했다. 이를 코드에서는 `dequeueReusableCellWithIdentifier`함수를 사용해서 재사용한다. 하지만 이 함수는 셀을 재사용하기 때문에 큐에 들어간 셀의 내용을 보여질 수가 있다.

즉 셀의 데이터소스의 내용은 다르지만 셀 자체는 재사용되기 때문에 첫번째 셀에만 체크박스를 했는데 스크롤하면서 재사용되는 셀에도 체크박스가 있게 되는 것이다.


### 해결방법
이 현상은 셀의 속성을 초기화시켜주지 않은 채 재사용되기 때문에 발생한다. 그렇기에 재사용되는 셀의 속성을 초기화시켜주는 것이 해결방법이다.

```swift
// tableViewCell.swift
override func prepareForReuse() {
    super.prepareForReuse()
    
    self.accessoryType = .none
}
```

테이블뷰 셀은 `prepareForReuse`함수를 제공하는데 이곳에서 셀의 뷰 측면의 속성들을 초기화시켜주면 된다. (Dequeue -> prepareForReuse -> cellForRowAt)

하지만 위 코드는 계속 체크표시를 없애기 때문에 데이터소스를 확인해서 선택된 셀은 체크박스를 다시 보이게 설정해야 한다.

```swift
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    guard let cell = tableView.dequeueReusableCell(withIdentifier: CountryTableViewCell.identifier, for: indexPath) as? CountryTableViewCell else { return UITableViewCell() }

    let currentCountry = countries[indexPath.row]

    if currentCountry.isSelected {
        cell.accessoryType = .checkmark
    }

    cell.configure(with: currentCountry)

    return cell
}
```

### 더 개선할 부분
일반적인 부분에서는 위와 같은 큐를 이용하고 상태를 초기화하는 것으로 충분하다. 하지만 인터넷 상황이 좋지 않은 상황에서 스크롤을 빠르게 한다면 이미지가 있어야할 공간에 있지않고 다른 공간에 있을 수 있다. 이는 셀에서 이미지가 로드되지 않은 상태에서 재사용되기 때문에 일어나는 현상이다. 

이런 현상 역시 `prepareForReuse` 메서드를 사용해서 개선할 수 있다.
```swift
override func prepareForReuse() {
    mainImageView.af_cancelImageRequest()
    mainImageView.image = nil
}
```

위 코드처럼 셀이 dequeue되기 직전 이미지 요청이 있다면 취소하고 imageView의 image를 nil로 바꾸는 것만으로도 이상한 위치에 있는 이미지가 나오는 것을 방지할 수 있다.(대신에 아무것도 안 들어 있을 것이다.)



### 참고자료
- [Apple Developer Documentation - metatype](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/types/)
- [소들이님 블로그 - 메타타입 1](https://babbab2.tistory.com/151)
- [소들이님 블로그 - 메타타입 2](https://babbab2.tistory.com/152)
- [Apple Developer Documentation - reuseIdentifier](https://developer.apple.com/documentation/uikit/uitableviewcell/1623246-reuseidentifier)
- [유셩장님 블로그 - dequeueReusableCellWithIdentifier 과정과 사용이유](https://sihyungyou.github.io/iOS-dequeueReusableCell/)

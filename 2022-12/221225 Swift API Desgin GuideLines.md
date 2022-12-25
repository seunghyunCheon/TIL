
221225 Swift API Desgin GuideLines
===
TIL (Today I Learned)
===
> API Design GuideLines의 모든 내용을 포함하고 있지는 않으며 의역으로 인한 해석 오류가 있을 수 있습니다. 참고해서 봐주시면 감사하겠습니다.

학습 내용
---
- Swift API Design GuideLines 문서 번역

목차
---
- [기초](#기초)
- [네이밍](#네이밍)
    - [명확한 사용을 촉진](#명확한-사용을-촉진)
    - [유창한 사용을 위한 노력](#유창한-사용을-위한-노력)
    - [전문용어를 잘 사용](#전문용어를-잘-사용)
- [컨벤션](#컨벤션)
    - [일반적인 컨벤션](#일반적인-컨벤션)
    - [파라미터](#파라미터)
    - [아규먼트 레이블](#아규먼트-레이블)
- [특별한 지시](#특별한-지시)

## 기초
- 사용 시점이 명확한 것이 가장 중요한 목표. 메서드나 프로퍼티들은 오직 한 번만 선언되고 반복적으로 사용이 되고, API를 간결하고 명확하게 설계하는 것이 중요하다.선언문을 읽는 것만으로는 충분하지 않기에 항상 사용사례를 검토해서 문맥상 명확하도록 한다.
- 명확성은 간결성보다 중요하다. 비록 Swift 코드는 compact하지만 가장 적은 문자들로 가장 작은 코드를 만드는 것은 목표가 아니다.
- swift의 간결성은 자연스럽게 보일러 플레이트를 줄여주는 강력한 타입 시스템의 side-effect이다.
- 모든 선언에 문서주석을 쓰자. 문서 주석으로 얻은 통찰력은 설계에 큰 영향을 준다. 그러니 미루지 말자. 
> 만약 API기능을 간단한 용어로 설명하지 못한다면 아마 wrong API를 작성하고 있을 것이다.
### Documentation
1. swift의 마크다운을 활용해라
2. 엔티티가 선언되는 것을 설명하는 요약으로 시작해라. API는 그것의 선언과 요약으로 완전히 이해할 수 있다.
3. 가능하면 단일 문장으로 끝내라.
4. 함수가 리턴하는 것을 설명한다. 단 null effect와 Void는 제외한다.
5. 초기화가 생성하는 것을 설명한다.
6. 선언된 엔티티가 무엇이고 어떤 기능을 하는지 설명한다.

## 네이밍
### 명확한 사용을 촉진
- 이름이 사용되는 코드를 읽는 사람에게 헷갈리지 않게 하기위해 모든 단어를 포함해라.
```swift
// good
extension List {
    public mutating func remove(at position: Index) -> Element
}
employees.remove(at: x)


// bad
employees.remove(x) // unclear: x를 지우는 건가?
```
- 불필요한 단어는 생략한다. 이름의 모든 글자는 사용할때 중요한 의미를 전달해야한다. 많은 단어들은 명확한 의미를 가져야하지만, 독자가 이미 보유하고 있는 정보로 중복되는 정보는 생략해야 한다. 특히 타입 유형만 반복하는 단어들은 생략해야 한다. 아래 예시로는 Element가 타입 유형이다.
```swift
// bad
public mutating func removeElement(_ member: Element) -> Element?

allViews.removeElement(cancelButton)

// good
public mutating func remove(_ member: Element) -> Element?

allViews.remove(cancelButton) // clearer
```
- 변수, 파라미터, 연관타입의 이름은 그들의 타입제약조건보다는 역할에 따라서 갖도록 한다.
```swift
// bad
var string = "Hello"
protocol ViewController {
  associatedtype ViewType : View
}
class ProductionLine {
  func restock(from widgetFactory: WidgetFactory)
}
```
위의 방식으로 타입 이름을 정하면 명확성과 표현성이 최적화되지 않는다. 엔티티의 역할을 표현하는 이름을 선택하도록 노력해라.
```swift
// good
var greeting = "Hello"
protocol ViewController {
  associatedtype ContentView : View
}
class ProductionLine {
  func restock(from supplier: WidgetFactory)
}
```
만약 연관값이 프로토콜 제약조건에 밀접하게 연결되어 프로토콜의 이름이 곧 역할이라면, Protocal이름에 Protocol을 추가함으로써 충돌을 피해라.
```swift
protocol Sequence {
  associatedtype Iterator : IteratorProtocol
}
protocol IteratorProtocol { ... }
```
- 취약한 타입 정보를 보안해서 파라미터의 역할을 명시해라.
특히 파라미터 타입이 NSObject, Any, AnyObject나 Int, String같은 기초타입이라면 사용시점에서의 타입정보와 문맥이 완전히 의미를 전달하지 못할지도 모른다. 이런 경우 선언시점은 명확하지만 사용시점이 막연하다.
```swift
// bad
func add(_ observer: NSObject, for keyPath: String)

grid.add(self, for: graphics) // 무엇을 추가? for는 무엇을 가르키지?
```
명료함을 복원하려면, 각 약식 매개변수( weakly typed parameter) 앞에 그 역할을 설명하는 명사를 붙인다.

```swift
func addObserver(_ observer: NSObject, forKeyPath path: String)
grid.addObserver(self, forKeyPath: graphics) // clear
```

### 유창한 사용을 위한 노력
- 메서드나 함수이름을 영어 문법에 맞게 짓는 것을 선호한다.
```swift
// good
x.insert(y, at: z)          “x, insert y at z”
x.subViews(havingColor: y)  “x's subviews having color y”
x.capitalizingNouns()       “x, capitalizing nouns”

// bad
x.insert(y, position: z)
x.subViews(color: y)
x.nounCapitalize()
```
- 첫번째 인수나 두번째 인수 이후에 유창성이 저하되는 것은 허용된다. (그러한 매개변수들이 중심이 아닐 때)
```swift
AudioUnit.instantiate(
  with: description, 
  options: [.inProcess], completionHandler: stopProgressBar)
```
- 팩토리 메서드 이름은 "make"로 시작하라.
- 초기화 및 팩토리 메서드 호출에 대한 첫번째 인자는 기존의 문법 이름에 설명을 추가하지 않는다.
```swift
// good 
let foreground = Color(red: 32, green: 64, blue: 128)
let newPart = factory.makeWidget(gears: 42, spindles: 14)
let ref = Link(target: destination)

// bad
let foreground = Color(havingRGBValuesRed: 32, green: 64, andBlue: 128)
let newPart = factory.makeWidget(havingGearCount: 42, andSpindleCount: 14)
let ref = Link(to: destination)
```
- 함수와 메서드 이름을 side-effect에 따라 짓는다.
    - side-effect가 없는 것들은 명사구로 읽혀져야 한다. e.g. x.distance(to: y), i.successor()
    - side-effect가 있는 것들은 명령형 동사 구로 읽혀져야한다. e.g. print(x), x.sort(), x.append(y)
    - Mutating/nonmutating 메서드 이름을 일관되게 짓는다. Mutating 메서드는 종종 유사한 의미를 갖는 nonmutating 메서드를 가지지만 그 위치에서 인스턴스를 업데이트 하기보다는 새로운 값을 리턴해라.
        - 연산이 자연스럽게 동사에 의해 설명된다면 mutating 메서드에 동사의 명령을 사용하고, nonmutating메서드에는 "ed"나 "ing"를 이름뒤에 붙여서 사용해라.
        - nonmutating 메서드에는 과거 분사 "ed"를 사용한다.
        - 직접목적어가 존재할 때는 현재분사 "ing"를 사용한다.
        ```swift
        mutating func stripNewlines()
        func strippingNewlines() -> String

        s.stripNewlines() // 바로 새로운 라인이 추가가 된다.
        let oneLine = t.strippingNewlines() // 새로운 라인을 리턴을 한다.
        ```
        - 연산이 자연스럽게 명사로 설명된다면 nonmutating 메서드에서 명사로 사용한다.
        - 하지만 mutating 메서드라면 앞에 form을 붙인다.
- Bool타입의 속성과 메서드는 nonmutating일 때 수신자에 대한 주장으로 읽혀져야 한다. e.g. x.isEmpty, line1.intersects(line2)
- 무엇이 무엇인지 설명하는 프로토콜은 명사로 읽어야 한다. e.g. Collection
- 능력을 설명하는 프로토콜은 뒤에 able, ing를 붙인다. e.g. Equatable, ProgressReporting
- 타입, 속성, 변수, 상수는 명사로 읽어야 한다.

### 전문용어를 잘 사용
- 만약 일반적인 용어로 의미가 잘 전달된다면 모호한 용어는 사용하지 마라.
- 만약 전문용어를 사용하고자 한다면 확립된 의미를 고수해라.
- 약어를 피해라. 
- 선례를 받아들여라

간단한 용어를 사용하기 보다는 근간이 되는 용어를 사용해라. 

## 컨벤션
### 일반적인 컨벤션
- 시간복잡도가 O(1)이 아닌 계산 프로퍼티에는 문서화를 한다. 사람들은 프로퍼티에 대해 중요한 연산이 없다고 생각한다. 왜냐하면 저장 프로퍼티는 멘탈모델로써 저장된다고 생각하기 때문이다. 
- free 함수보다 메서드와 프로퍼티를 선호해라. (free함수의 정확한 뜻은 모르겠지만 인수의 순서가 중요하지않은 min과 max함수와 같은 것들로 추정된다)
```swift
    min(x, y, z)
    print(x)
    sin(x)
```
free function: 명확한 self가 없거나 제약사항이 없는 generic이거나 확립된 도메인 표기법의 일부인경우
- 타입과 프로토콜은 UpperCamelCase를, 다른 모든 것들은 lowerCamelCase를 한다.
    - 미국 영어에서 흔히 대문자로 표시되는 약어와 이니셜은 전부 upper case로 나타나거나 전부 down-case로 나타낸다.  
    <br>
    ```swift
        var utf8Bytes: [UTF8.CodeUnit]
        var isRepresentableAsASCII = true
        var userSMTPServer: SecureSMTPServert
    ```
    - 다른 약어는 기존의 단어를 따른다.  
    <br>
    ```swift
        var radarDetector: RadarScanner
        var enjoyScubaDiving = true
    ```
- 메서드들은 그것들이 같은 기초적인 의미를 공유하고 있거나 분명한 도메인에서 동작할 때 이름을 공유할 수 있다.

    ```swift
    extension Shape {
      /// Returns `true` if `other` is within the area of `self`;
      /// otherwise, `false`.
      func contains(_ other: Point) -> Bool { ... }

      /// Returns `true` if `other` is entirely within the area of `self`;
      /// otherwise, `false`.
      func contains(_ other: Shape) -> Bool { ... }

      /// Returns `true` if `other` is within the area of `self`;
      /// otherwise, `false`.
      func contains(_ other: LineSegment) -> Bool { ... }
    }
    ```

    - 기하 타입과 Collection타입은 분리되어있기 때문에 아래와 같은 방식도 괜찮다.
<br>
    ```swift
    extension Collection where Element : Equatable {
      /// Returns `true` if `self` contains an element equal to
      /// `sought`; otherwise, `false`.
      func contains(_ sought: Element) -> Bool { ... }
    }
    ```
    - 그러나 아래 index메서드들은 다른 의미를 갖고 있기 때문에 이름을 다르게 지어야한다.  
    <br>
    ```swift
    extension Database {
      /// Rebuilds the database's search index
      func index() { ... }

      /// Returns the `n`th row in the given table.
      func index(_ n: Int, inTable: TableID) -> TableRow { ... }
    }
    ```
    - 마지막으로 리턴타입에 대한 오버로딩을 피해라. 타입추론에 있어 모호성을 주기 때문이다.  
    <br>
    ```swift
    extension Box {
      /// Returns the `Int` stored in `self`, if any, and
      /// `nil` otherwise.
      func value() -> Int? { ... }

      /// Returns the `String` stored in `self`, if any, and
      /// `nil` otherwise.
      func value() -> String? { ... }
    }
    ```

## 파라미터
- 문서를 제공에 쓰일 파라미터 이름을 사용해라. 비록 파라미터가 함수나 메서드의 사용시점에서 보여지진 않지만 중요한 역할을 한다.

    문서에서 읽기 쉬운 이름을 선택해라. 다음의 예시는 자연스럽게 읽힌다.
    
    ```swift
    /// Return an `Array` containing the elements of `self`
    /// that satisfy `predicate`.
    func filter(_ predicate: (Element) -> Bool) -> [Generator.Element]

    /// Replace the given `subRange` of elements with `newElements`.
    mutating func replaceRange(_ subRange: Range, with newElements: [E])
    ```
    그러나 다음의 예시는 잘 읽히지 않는다.<br>

    
    ```swift
    /// Return an `Array` containing the elements of `self`
    /// that satisfy `includedInResult`.
    func filter(_ includedInResult: (Element) -> Bool) -> [Generator.Element]

    /// Replace the range of elements indicated by `r` with
    /// the contents of `with`.
    mutating func replaceRange(_ r: Range, with: [E])
    ```
- 기본값이 일반적인 사용을 단순화할 때 기본값 파라미터를 사용한다. 
    기본값 아규먼트는 관련없는 정보를 숨김으로써 가독성을 향상시킨다.
    ```swift
    let order = lastName.compare(
      royalFamilyName, options: [], range: nil, locale: nil)
    ```
    다음과 같이 더욱 간단해 진다.
    ```swift
    let order = lastName.compare(royalFamilyName)
    ```
    API를 사용하려는 사람들에게 더 낮은 인지부담을 주기 때문에 메서드 패밀리에서 사용하기보다는 기본 아규먼트를 일반적으로 사용한다.
    ```swift
    extension String {
      /// ...description...
      public func compare(
         _ other: String, options: CompareOptions = [],
         range: Range? = nil, locale: Locale? = nil
      ) -> Ordering
    }
    ```
    메서드 패밀리의 모든 멤버들은 별도로 분리되고 이해되어야 하지만 기본값을 사용하는 단일 메서드를 사용하면 이들에 대해 지루한 차이를 찾는 노력을 할 필요가 없어진다.(모든 기본값을 가지는 매개변수를 갖고 있는 함수가 정의되어있기 때문에)
- 기본값이 있는 파라미터는 뒤쪽에 위치시킨다. 기본값이 없는 파라미터는 대게 중요하기 때문이다.


## 아규먼트 레이블
- 아규먼트들이 유용하게 구별될 수 없는 경우 모든 레이블을 생략한다. e.g. min(number1, number2), zip(sequence1, sequence2)
- 값 보존 타입 전환을 수행하는 이니셜라이저는 첫번째 아규먼트레이블을 생략한다. e.g. Int64(someUInt32) 하지만 "narrowing" 타입 전환을 하는 경우에는 "narrowing"을 설명하는 레이블을 사용하길 권장합니다.
```swift
extension UInt32 {
  /// Creates an instance having the specified `value`.
  init(_ value: Int16)            ← Widening, so no label
  /// Creates an instance having the lowest 32 bits of `source`.
  init(truncating source: UInt64)
  /// Creates an instance having the nearest representable
  /// approximation of `valueToApproximate`.
  init(saturating valueToApproximate: UInt64)
}
```
- 첫번째 아규먼트가 전치사 구문의 일부를 형성할 때 아규먼트 레이블을 작성한다. e.g. x.removeBoxes(havingLength: 12)
- 처음 두 전달인자가 단일 추상화의 경우 예외사항이 발생한다.
```swift
// bad
a.move(toX: b, y: c)
a.fade(fromRd: b, green: c, blue: d)

// good
a.moveTo(x: b, y: c)
a.fadeFrom(red: b, green: c, blue: d)
```
전치사 구에서 인자가 여러개가 들어올 때는 전치사를 함수이름에 적고 인수들을 나열하는 것으로 이해했다.(확실하지 않음)
- 문법적 구문이 맞지 않을 때는 인수 레이블을 작성하는 것이 중요하다.
```swift
// good
view.dismiss(animated: false)
let text = words.split(maxSplits: 12)
let studentsByName = students.sorted(isOrderedBefore: Student.namePrecedes)

// bad 
view.dismiss(false)
words.split(12)
```
dismiss의 경우 무엇을 false하는지 모르니까 인수레이블로 animated를 줌으로써 확실하게 표현. 

## 특별한 지시
- 튜플 멤버와 클로저 파라미터 이름을 지어라. 이러한 이름들은 문서 주석에 참조될 수 있고 튜플 멤버접근에 용이해진다.
```swift
/// Ensure that we hold uniquely-referenced storage for at least
/// `requestedCapacity` elements.
///
/// If more storage is needed, `allocate` is called with
/// `byteCount` equal to the number of maximally-aligned
/// bytes to allocate.
///
/// - Returns:
///   - reallocated: `true` if a new block of memory
///     was allocated; otherwise, `false`.
///   - capacityChanged: `true` if `capacity` was updated;
///     otherwise, `false`.
mutating func ensureUniqueStorage(
  minimumCapacity requestedCapacity: Int, 
  allocate: (_ byteCount: Int) -> UnsafePointer<Void>
) -> (reallocated: Bool, capacityChanged: Bool)
```
클로저 아규먼트는 지원되지 않기 때문에 클로저 파라미터에 사용되는 이름은 top-level 함수로 설정되야 한다.
- Any, AnyObject와 같은 제약없는 다형성을 사용할때는 overload에서의 모호성을 피하기 위해 신경을 더 써라
```swift
struct Array {
  /// Inserts `newElement` at `self.endIndex`.
  public mutating func append(_ newElement: Element)

  /// Inserts the contents of `newElements`, in order, at
  /// `self.endIndex`.
  public mutating func append(_ newElements: S)
    where S.Generator.Element == Element
}
```
위 예시는 values.append([2, 3, 4])를 했을 때 2, 3, 4를 추가하는 건지 [2, 3, 4]를 추가하는 건지 명확하지 않아 헷갈리게 된다. 이를 해결하기 위해선 아규먼트 레이블을 유저입장에서 확실하게 알 수 있도록 설정해주는 것이 좋다.
```swift
struct Array {
  /// Inserts `newElement` at `self.endIndex`.
  public mutating func append(_ newElement: Element)

  /// Inserts the contents of `newElements`, in order, at
  /// `self.endIndex`.
  public mutating func append(contentsOf newElements: S)
    where S.Generator.Element == Element
}
```

Reference
---
- [Swift API design GuideLines](https://www.swift.org/documentation/api-design-guidelines/)


###### tags: `naming`, `convention`, `parameter`, `argument`

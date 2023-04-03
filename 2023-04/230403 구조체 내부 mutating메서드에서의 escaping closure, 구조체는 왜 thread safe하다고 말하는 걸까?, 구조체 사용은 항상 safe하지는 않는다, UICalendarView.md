# 230403 구조체 내부 mutating메서드에서의 escaping closure, 구조체는 왜 thread safe하다고 말하는 걸까?, 구조체 사용은 항상 safe하지는 않다, UICalendarView
```
틀린내용이 많을 수 있습니다. 의견 내주시면 감사하겠습니다😄
```
===
학습내용
- 구조체 내부 mutating메서드에서의 escaping closure
- 구조체는 왜 thread safe하다고 말하는 걸까?
- 구조체 내부 mutating메서드에서의 escaping closure, 구조체 사용은 항상 safe하지는 않는다.
- UICalendarView
## 구조체 내부 mutating메서드에서의 escaping closure
프로젝트 진행 중 구조체의 mutating메서드에서 escaping closure에서 self를 참조했을때 컴파일 에러가 발생하는 문제가 발생했다.
<img src="https://i.imgur.com/XQ7fHv0.png" width="500"/><br/>

escaping closure는 현재 context를 빠져나간 뒤에 실행되는 메서드이기 때문에 `self`를 참조하고 있어야 한다. 하지만 구조체는 인스턴스를 참조하는 개념이 아니라 복사하는 개념이기 때문에 escaping closure에서 self를 참조하는 것이 값타입의 개념과 충돌하는 것이다. 

반면에 non-escaping 클로저에서 self를 참조하는 것은 클로저가 호출되는 동안만 유효하기 때문에 self를 사용할 수 있다. 이때는 참조하는 것이 아니라 복사하는 것과 같다. (복사해서 인스턴스의 프로퍼티를 변경하고 대체시켜주는 것이다)

그래서 대안으로 [self] in을 사용할 수도 있는데 이는 클로저의 생성시점에 `self`를 복사해놓는 것이기 때문에 생성시점의 `self`만 사용하기 때문에 컴파일에러가 보이지 않는다. 하지만 실제 인스턴스의 프로퍼티가 변경되더라도 클로저에서 캡처하고 있는 `self`는 생성시점만 캡처하고 있기 때문에 이 `self`의 프로퍼티는 변하지 않는다. 즉 escaping closure를 통해 다른 컨텍스트에서 실행중인 `self`내의 프로퍼티는 고정된 값이기 때문에 현재`self`의 프로퍼티가 변경되더라도 반영할 수 없다. 그래서 이 상황에서 클래스를 사용해 프로퍼티의 변화를 추적했다. 
하지만 이 경우 주의할 점은 escaping closure에서 `self`의 프로퍼티를 변경하는 코드가 있을 때 `self`가 thread-safety하지 않기 때문에 직렬 큐를 사용하거나 세마포어를 사용해서 race condition이 나지않게 해야 한다.
<img src="https://i.imgur.com/HKmOUq5.png" width="400"/>

## 구조체는 왜 thread safe하다고 말하는 걸까?
구조체는 값이 복사되는 형태인 값타입이기 때문에 여러 스레드에서 구조체에 접근할 때 인스턴스를 복사한 후 접근하게 된다. 즉 원본데이터에는 영향을 주지 않기 때문에 여러 스레드에서 접근하더라도 결과가 같기에 thread-safe한 것이다. 
그런데 여러 스레드에서 구조체에 접근한다는 의미는 DispatchQueue를 이용한 escaping closure에서 접근한다는 의미일텐데 escaping closure에서는 값타입을 참조를 하지 못하지 않지 못한다(이 자체가 thread-safe하다고 하는 건가 싶었다)
그리고 참조가 아닌 클로저 캡처리스트를 이용해 `[self] in`으로 값을 복사해서 인스턴스의 프로퍼티를 변경하더라도 원본 인스턴스는 변화되지 않기 때문에  참조가 불가능한 형태로 사용한다는 것이기 때문에 이 자체로 thread-safe하다고 말을 하는걸까? 생각이 들었다.
이 점이 궁금해서 구조체의 주소를 통해 실제 인스턴스의 값이 변경될 수 있게 만드는 `inout`을 사용해서 다음의 코드를 작성했다.
```swift
import Foundation

struct Point {
    var x = 0
    var y = 0

    mutating func update() {
        x += 1
        y += 1
    }

    func printPoint() {
        print("(\(x), \(y))")
    }
}

func modifyPoint(point: inout Point) {
    point.update()
    point.printPoint()
}

let queue = DispatchQueue(label: "thread-test", attributes: .concurrent)

let point = Point(x: 1, y: 1)

for _ in 0..<10 {
    queue.async {
        modifyPoint(point: &point)
    }
}

sleep(1)
```
위 코드에서 `&point`는 `point`의 주소를 넣는 것이기 때문에 `point`의 값이 `thread-safe`하지 못하게 동시에 증가가 되는 경우가 생긴다. 
이를 아래로 바꿔주면 `thread-safe`한 코드로 만들 수 있게 된다.
```swift
for _ in 0..<10 {
    queue.async {
        var copy = point
        modifyPoint(point: &copy)
    }
}
```
`copy`는 `point`의 값을 복사한 것이기 때문에 원본을 수정하지 않고 copy를 수정하기 때문에 thread-safe하다. 
즉 구조체도 thread-safe하게 사용하려면 lock을 걸어놓는다거나 값을 복사해놓고 사용하는 등(위 코드처럼 `var copy = point`로 하거나 [self in]을 사용하는 등)의 로직이 필요하다고 느꼈다.

## 구조체 내부 mutating메서드에서의 escaping closure, 구조체 사용은 항상 safe하지는 않는다.

### 사람들은 왜 클래스보다 구조체를 사용하도록 말할까?
스위프트 개발자들의 일반적인 조언은 기본적으로 구조체를 사용하라고 하는 것이다. 그리고 클래스는 꼭 사용해야하는 경우에 사용하라고 한다. 왜 그런지 이유를 살펴보고 모든 상황에서 이게 옳은 것인지를 확인해보자.

### 구조체는 값타입이다.
값타입인 구조체는 각각 고유한 복사본이기 때문에 메모리 측면에서 관리하기가 더 쉽다. 반면에 참조타입은 메모리의 일부지점에 대한 참조이다.

### 구조체는 thread-safe하다
구조체는 고유한 복사본이기 때문에 여러 상황에서 한번에 하나의 스레드만 관리할 수 있다. 이는 여러개의 스레드에서 동시에 접근해서 버그를 발생시키는 것을 예방한다.

반대로 참조타입은 거의 모든 스레드에서 접근할 수 있다. 만약 두 개의 스레드에서 동시에 객체에 접근한다면 side-effect가 생길것이다. 이는 이상한 결과를 보여줄것이고 크래시를 만들 수 있다.

### 구조체는 변경가능성을 쉽게 제어한다.

구조체는 var로 선언하지 않으면 인스턴스를 변경시킬 수 없게하는 특별한 규칙이 있다. 나아가 구조체의 함수에서 프로퍼티를 변경시키려면 `mutating` 키워드를 붙여야 한다.

반면에 참조 타입은 mutating키워드 없이 프로퍼티를 변경할 수 있다.

### 사실 구조체의 변경가능성과 thread-safe는 좋은 규칙을 필요로한다.
이론적으로 위의 내용들은 모두 사실이지만 현실이 이를 어떻게 쉽게 구조체를 unsafe하게 방해하는지 살펴보자. 구조체 안에 참조타입을 넣게되면 구조체 내부에서 변경가능성을 제공하게 된다. 

```swift
struct ValueType {
    let name: String
    let referenceType: ReferenceType
}

class ReferenceType {
    var name: String
    
    init(name: String) {
        self.name = name
    }
}

let valueType = ValueType(name: "Kenny", referenceType: ReferenceType(name: "Test"))
valueType.referenceType.name = "Tom"
```

위 코드를 통해 구조체는 반드시 thread-safe하지는 않다는 것을 알 수 있다. 만약에 여러개의 스레드에서 구조체 내부의 프로퍼티에 접근한다면 실행할때마다 값이 변경될 수 있는 race condition상황에 놓일 수 있게 되는 것이다. 그렇기에 구조체를 사용하더라도 안전하게 만들기 위해 규율있는 접근법이 필요하다. 

### 언제 값타입을 사용하고 언제 참조타입을 사용하지?
가장 기본적으로 Model data를 구성할 때 struct를 사용하고 행동을 캡슐화할 때 class를 사용한다.

예를들어 e-commerce 앱에서 아이템을 struct로, 아이템을 소유하고 있고 total을 만들어주는 기능을 담는 곳은 class로 정의한다. 

```swift
struct ShoppintItem {
    let itemName: String
    let price: Double
    let aisle: Int
}

class Order {
    var shoppingItems: [ShoppintItem] = []
    
    func totalOrder() -> Double {
        let prices = shoppingItems.map { $0.price }
        let price = prices.reduce(0.0, +)
        return price
    }
}
```

위 코드에서 `ShoppingItem`은 내부 프로퍼티들이 값타입에서 let으로 설정되어있고 값타입만 받고있기 때문에 변경가능성이 존재하지 않는다. 그리고 `Order`는 전역에서 값이 복사된 채로 동작한다면 하나의 앱에서 서로다른 Item들이 shoppingItem이 담겨있을 수 있기 때문에 하나의 쇼핑목록을 공유하는 차원에서 Class로 정의하는 것이 좋다. 

## UICalendarView
날짜 모양의 캘린더를 보여주는 뷰. 사용자가 단일, 다중 날짜를 선택할 수 있도록 한다.

```swift
@MainActor class UICalendasrView: UIView
```

### Overview
캘린더 뷰를 사용하여 모양을 커스텀하여 사용자에게 정보를 갖고있는 날짜를 제공한다. 
사용자는 하나의 날짜, 다중날짜를 선택할 수 있고 선택하지 않을 수도 있다.

캘린더 뷰를 interface에 적용하기 위해 다음을 한다.
- 보여질 캘린더 뷰를 위해 Calendar와 Locale을 설정한다
- 초기에 보여지게 하기 위해 캘린더 뷰에 날짜를 설정한다.
- 원한다면 구체적인 날짜에 모양을 제공하기 위해 delegate를 사용한다
- 날짜 선택을 관리하기 위해 selection메서드와 델리게이트를 설정한다.
- calendar view를 오토레이아웃 설정한다

캘린더는 날짜 선택에만 사용이된다. 만약 날짜와 시간 선택을 다루고싶다면 `UIDatePicker`를 이용한다.

### Configure a calendar view
사용자의 위치, 문화, 언어에 맞게 캘린더를 구성한다. 
기본적으로 캘린더 뷰는 사용자의 현재 `Calendar`와 `Locale`을 선택한다. 만약 사용자가 앱에서 다른 캘린더나 locale을 선택할 수 있다면 그 선택한 것에 맞게 캘린더뷰를 구성해야 한다.

```swift
let calendarView = UICalendarView()
let gregorianCalendar = Calendar(identifier: .gregorian)
calendarView.calendar = gregorianCalendar
calendarView.locale = Locale(identifier: "zh_TW")
calendarView.fontDesign = .rounded
```

그 다음 캘린더 뷰에 보여질 초기값을 설정한다.
```swift
calendarView.visibleDataComponents = DateComponents(calendar: Calendar(identifier: .gregorian), year: 2022, month: 6, day: 6)
```
만약 사용자가 볼 수 있고 선택할 수 있는 날짜를 최신의 날짜로 제한하려면 `availableDateRange`를 사용한다.

```swift
let fromDataComponents = DateComponents(calendar: Calendar(identifier: .gregorian), year: 2022, month: 1, day: 1)
let toDateComponents = DataComponents(calendar: Calendar(identifier: .gregorian), year: 2022, month: 12, day: 31)

guard let fromDate = fromDateComponents.date, let toDate = toDateComponents.date else {
    return
}

let calendarViewDateRange = DateInterval(start: fromDate, end: toDate)
calendarView.availableDateRange = calendarViewDateRange
```

### Display decorations for specific dates
사용자에게 특정 날짜에 decoration을 통해 구체적인 날짜정보를 보여줄 수 있다. color, size, image, custom view를 특정 날짜에 적용함으로써 강조할 수 있다. 캘린더 뷰에 decoration을 보여주기 위해 custom `UICalendarViewDelegate` 객체를 만들고 `calendarView(_:decorationFor:)`를 구현한다. 그리고 커스텀 객체를 calendar view의 delegate로 놓는다. 

```swift
class MyEventDatabase: NSObject, UICalendarViewDelegate {
    // Your database implementation goes here.
    func calendarView(_ calendarView: UICalendarView, decorationFor: DateComponents) -> UICalendarView.Decoration? {
        // Create and return calendar decorations here.
    }
}

let database = MyEventDatabase()
calendarView.delegate = database
```

`UICalendarView.Decoration`의 어떤 타입이 가장 잘맞는지 결정한다
- 기본 decoration은 색과 상대적인 사이즈를 커스텀할 수 있는 색이채워진 원을 보여준다.
- 이미지 decoration은 system symbol, color, 상대적인 사이즈와 함께 커스터마이즈 할 수 있다.
- custom decoration은 커스텀 뷰를 반환하는 클로저와함께 커스터마이즈 할 수 있다.

데이터가 변경되었을 때 캘린더 뷰에게 decoration view를 reload하라고 말해준다.

```swift
database.observeEventChanges { changedDates in
    calendarView.reloadDecorations(for: changedDates, animated: true)
}
```

### Handle date selection

원하는 날짜 유형선택을 결정한다. 그런다음 그 유형에 맞는 `selection` 객체와 `delegate`를 만든다. 그리고 캘린더 뷰의 `selectionBehavior`에 할당한다.

```swift
let dateSelection = UICalendarSelectionSingleDate(delegate: self)
calendarView.selectionBehavior = dateSelection
```
사용자는 캘린더뷰로부터 날짜를 고를 수 있다. 다음과 같이 코드로 선택된날자를 설정할 수도 있다.

`dateSelection.selectedDate(DataComponents(calendar: Calendar(identifier: .gregorian), year: 2022, month: 6, day: 6))`

다음으로 date selection을 처리하기 위해 델리게이트 메서드를 구현한다.
```swift
func dateSelection(_ selection: UICalendarSectionSingleDate, canSelectDate dateComponents: DateComponents?) -> Bool {
    return dateComponents != nil
}
```

### 참고문서
- [Using Structs over Classes in Swift Isn’t as Safe as You Think It is](https://medium.com/devgauge/using-structs-over-classes-in-swift-isnt-as-safe-as-you-think-it-is-a59794d10c31)
- [클로저 캡쳐](https://velog.io/@kimdo2297/%ED%81%B4%EB%A1%9C%EC%A0%B8-%EC%BA%A1%EC%B3%90%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C-about-closure-capture)
- [Apple Docs - UICalendarView](https://developer.apple.com/documentation/uikit/uicalendarview)

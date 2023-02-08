230206 NSNumber, UIStackView, UIScrollView
===
TIL (Today I Learned)
===
학습내용
---
- NSNumber 학습하기

## NSNumber
> 원시적인 숫자값을 감싸고있는 객체
 
`NSNumber`는 `NSValue`의 서브클래스로 부호가 있거나 없는 `char, short int, int, long, int, long long int, float, double, bool`에 접근하거나 설정하는 메서드의 집합을 제공한다.

`NSNumber`는 특정 `Boolean, integer, unsigned ingeger, floating point`타입으로 변환된 객체의 저장된 값을 반환하는 읽기전용 프로퍼티를 제공한다. 
`numeric type`들은 다른 저장 수용능력을 갖고있기 때문에 어떤 타입으로 초기화를 시도한 뒤 다른 타입으로 접근하면 에러가 날 수 있다. 
예시로는 `double`로 초기화를 하고 `floatValue`로 접근하는 것이 있다.

`NSNumber`는 다양한 유형의 숫자를 저장하도록 설계된 Objective-C 클래스로, Objective-C에서는 `Int, dobule`등의 원시적인 숫자 유형을 `NSNumber`로 감싸지 않고는 대부분의 애플 API에서 사용할 수 없기 때문에 중요하지만 스위프트에서는 대부분 자동으로 `NSNumber`로 변환하는 기능을 한다.

하지만 직접 NSNumber로 변환해줘야하는 경우가 있다. 예시로 `NumberFormatter`가 존재한다.
```swift
let number = 1000
let formatter = NumberFormatter()
formatter.numberStyle = .spellOut

let string1 = formatter.string(form: number) ?? "" // error
```

스위프트에서는 자동으로 `int`를 `NSNumber`로 브릿징하지 않기 때문에 여기서 타입캐스팅을 해줘야 한다.
```swift
let string2 = formatter.string(from: number as NSNumber) ?? ""
```
## UIStackView
> 행이나 열로 뷰의 컬렉션형태를 그리는 streamlined된 인터페이스
```swift
@MainActor class UIStackView : UIView
```
스택뷰는 디바이스 방향이나 스크린 사이즈에 따라 동적으로 적응시키는 유저 인터페이스이다. 
`arrangedSubviews` 프로퍼티를 이용해 모든 뷰들을 관리한다. 이러한 서브뷰들은 스택 뷰의 `axis`, `distribution`, `alignment`, `spacing`에 따라 레이아웃이 그려진다.

### arrangedSubvies
> 스택뷰에 의해 정렬된 뷰 리스트
```swift
var arrangedSubviews: [UIView] { get }
```

#### subView 추가
`addArrangedSubView()`, `insertArrangedSubview(_:at)`을 이용해 추가할 수 있다.
`addArrangedSubview`는 스택 뷰의 `subView`가 아닌 경우 스택 뷰의 `subView`에 자동으로 추가한다. 하지만 이미 스택 뷰의 `subView`라면 추가되지 않는다.
`insertArrangedSubview(_:at:)`은 입력받은 인덱스에 뷰를 추가한다.

#### subView 제거
`removeArrangedSubview(_:)`는 배열에서 입력받은 `view`를 제거하는 메서드이다. 이를 통해 제거된 뷰는 더 이상 스택 뷰에서 관리되지 않는다. 즉 `arrangedSubviews` 배열에서 뷰를 제거할 수는 있지만 View 계층에서는 제거하지 못한다.

뷰 계층에서 제거하려면 `removeFromSupeview()`를 호출해야 한다.

## UIScrollView

<img src="https://i.imgur.com/3Ov2B0V.png" width="500"/>

### ContentOffset
> 스크롤 뷰의 origin으로부터 컨텐츠 뷰의 origin까지 얼마나 떨어졌는지를 나타내는 포인트
```swift
var contentOffset: CGPoint { get set }
```
- default value: `CGPointZero`


### ContentInset
> content view가 safe area 혹은 스크롤뷰 모서리에 삽입되는 사용자 지정 거리
```swift
var contentInset: UIEdgeInsets { get set }
```
- 즉 content와 가장자리와의 간격이다.

### ContentSize
> 컨텐츠 뷰의 사이즈
```swift
var contentSize: CGSize { get set }
```



- [Apple Developer Documentation - NSNumber](https://developer.apple.com/documentation/foundation/nsnumber)
- [What is NSNumber?](https://www.hackingwithswift.com/example-code/language/what-is-nsnumber)
- [stack.addArrangedSubView](https://velog.io/@wonhee010/stack.addArrangedSubview)
- [Apple Delveloper Documentation - UIStackView](https://developer.apple.com/documentation/uikit/uistackview)
- [Apple Delveloper Documentation - arrangedSubviews](https://developer.apple.com/documentation/uikit/uistackview/1616232-arrangedsubviews)
- [Apple Delveloper Documentation - contentOffset](https://developer.apple.com/documentation/uikit/uiscrollview/1619404-contentoffset)
- [Apple Delveloper Documentation - contentInset](https://developer.apple.com/documentation/uikit/uiscrollview/1619406-contentinset)

230103 KVO(Key Value Observing), Notification, Access Control
===
TIL (Today I Learned)
===
학습내용
---
- KVO 학습하기
- Notification 학습하기
- Access Control 학습하기

## 고민한 점 / 해결 방법

### [KVO]
> KVO는 어떤 객체의 속성이 변했을 때 바로 다른 객체가 알아차리도록 하는 것을 말한다. 이는 애플리케이션의 응집성에 중요한 요소가 될 수 있다.
- 일반적으로 controller객체는 모델을 observe하고, view객체는 controller와 model객체를 observe한다.
- UIKit 프레임워크의 클래스들이 일반적으로 KVO를 지원하지 않더라도 NSObject를 상속해서 구현할 수 있다.
- Notification는 모든 객체로 알림을 보내는 중심객체가 있는 반면에 KVO 알림은 값이 변할 때 직접적으로 객체에 변화를 준다.

#### 사용방법 01
```swift
class MyObjectToObserve: NSObject {
    @objc dynamic var myDate = NSDate(timeIntervalSince1970: 0) // 1970
    func updateDate() {
        myDate = myDate.addingTimeInterval(Double(2 << 30)) // Adds about 68 years.
    }
}
```
- NSObject를 상속받고 @objc dynamic키워드를 통해 Observing될 수 있는 값으로 만든다.

#### 사용방법 02
```swift
class MyObserver: NSObject {
    @objc var objectToObserve: MyObjectToObserve
    var observation: NSKeyValueObservation?

    init(object: MyObjectToObserve) {
        objectToObserve = object
        super.init()

        observation = observe(
            \.objectToObserve.myDate,
            options: [.old, .new]
        ) { object, change in
            print("myDate changed from: \(change.oldValue!), updated to: \(change.newValue!)")
        }
    }
}
```
- 관찰하는 곳도 NSObject를 상속받고 관찰하는 타입을 안에 프로퍼티로 만들어준다.
- NSKeyValueObservation타입도 정의해서 이곳에 observe함수의 실행을 담는다. 
- 관찰하고자 하는 프로퍼티를 첫번째 인자에, 이 값에 대해 새로운 값과 변경되기전 값을 가져오고 싶다면 .old, .new를 추가해준다.
- myDate가 변경된다면 print문이 실행된다.

### [Notification]
> NotificationCenter를 통해 등록된 모든 옵저버들에게 broadcaste되는 container
#### Notification property
- struct NSNotification.Name: notification의 이름을 정의하는 구조체(이곳에서 알림의 이름이자 식별자를 설정)
- Notification의 Name은 보통 extension해서 사용한다.
```swift
extension Notification.Name {
    static let play = Notification.Name("play")
}
```

#### NotificationCenter
<img src="https://i.imgur.com/Mw9ulkY.png" width="500"/><br>
등록된 옵저버들에게 정보를 broadcast할 수 있는 알림전송 메커니즘
- NotificationCenter.default: NotificationCenter의 싱글톤으로 어느 곳에서나 접근이 가능하다.

#### Example
```swift
import Foundation

extension Notification.Name {
    static let warning = Notification.Name("warning")
}

class GangNamRegion {
    let warningMessage = "[강남구] 코로나 확진환자 120명 발생"
    func sendWarningMessage() {
        NotificationCenter.default.post(name: Notification.Name.warning,
                                        object: nil,
                                        userInfo: ["Message": warningMessage])
    }
}


class Human {
    let name: String
    
    init(name: String) {
        self.name = name
    }
    
    func belong(to region: GangNamRegion) {
        NotificationCenter.default.addObserver(self,
                                               selector: #selector(receivedMessage(notification:)),
                                               name: Notification.Name.warning,
                                               object: nil)
    }
    
    @objc func receivedMessage(notification: Notification) {
        guard let warningMessage = notification.userInfo?["Message"] as? String else {
            return
        }
        print("\(warningMessage)")
    }
}

let rowan = Human(name: "rowan")
let goatt = Human(name: "goatt")
let gangNam = GangNamRegion()

rowan.belong(to: gangNam)
goatt.belong(to: gangNam)
gangNam.sendWarningMessage()

// [강남구] 코로나 확진환자 120명 발생
// [강남구] 코로나 확진환자 120명 발생
```
- post는 옵저버들에게 신호를 알리는 메서드. object는 주로 발신자를 넣는데, nil이 아닌 self를 넣게 된다면, self에 해당하는 타입에 대해서만 알림을 보내게 된다. 만약 이때 받는 곳에서 object가 nil이라면 어떤 타입이든 상관없다는 뜻이기 때문에 알림이 받아진다.
- 하지만 post하는 곳에서 object에 nil을 주고 observe하는 곳에서 특정타입을 object로 선언하게된다면 특정타입에 해당하는 것에 대해서만 observe하는 것이기 때문에 관찰되지 않는다.
- userInfo는 추가로 전달할 데이터를 넣는다.
```swift
NotificationCenter.default.post(name: Notification.Name.warning,
                                        object: nil,
                                        userInfo: ["Message": warningMessage])
```

### [Access Control]
#### 모듈
> - 코드를 배포하는 단위로 프레임워크나 앱이 이 단위로 배포된다.
> - 다른 모듈에서 import키워드를 사용해 접근할 수 있다.
> - Xcode의 각 빌드 타겟은 분리된 단일 모듈로 취급된다.

#### 접근레벨 가이드 원칙
> 더 낮은 레벨을 갖고 있는 다른 엔티티를 특정 엔티티에 선언해서 사용할 수 없다.
- public은 private, file-private에서 사용할 수 없다.
- 함수는 그 함수의 파라미터 타입이나 리턴 값 타입보다 더 높은 접근 레벨을 가질 수 없다.
> 만약 프레임워크의 구현을 감추고 싶다면 internal로 선언하면 된다.

#### 커스텀 타입
- public 타입은 기본적으로 internal 멤버를 갖고 public은 갖지 않는다. public API를 만들시 노출되지 말아야 할 API가 노출되는 경우를 방지하기 위한 이유이다.

#### 열거형 타입
- 열겨형의 타입과 case의 타입이 동일해야 한다.
- 고유값과 연관값은 반드시 그 타입보다 높은 접근 레벨을 가져야 한다.

#### 상수, 변수, 프로퍼티, 서브스크립트
- 타입보다 더 높은 접근레벨을 가질 수 없다.
```swift
private var privateInstance = SomePrivateClass()
```

#### 게터와 세터
```swift
public private(set) var number = 0
```
- get은 public, set은 private의 접근제어를 갖는다.

#### 프로토콜
- 프로토콜의 접근레벨과 그 안의 요구사항의 접근레벨은 동일하다.

### Reference
- [Swift Lanugate Guide - Access Control](https://docs.swift.org/swift-book/LanguageGuide/AccessControl.html)
- [Apple Document Archive - KVO](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/KVO.html)
- [Apple Developer Documentation - KVO](https://developer.apple.com/documentation/swift/using-key-value-observing-in-swift)
- [Notification Center, Delegate, KVO??](https://velog.io/@leeyoungwoozz/iOSSwift-Notification-Center-KVO)
- [Notification](https://kioo.tistory.com/entry/TIL-210315)
- [NotificationCenter](https://jeonyeohun.tistory.com/223)



###### tags: `KVO`, `Notification`, `Access control`, `접근제어`

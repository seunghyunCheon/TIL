230302 Accessibility Programming Guide for iOS 읽기, GCD(QoS, async, DispatchGroup, thread Safe, DispatchSemaphore, mainThread RunLoop)
===
학습내용
---
- Accessibility Programming Guide for iOS 읽기
- GCD(QoS, async, DispatchGroup, thread Safe, DispatchSemaphore, mainThread RunLoop)
## Accessibility Programming Guide for iOS - Understanding Accessibility on iOS

시작부터 iOS기반의 디바이스들은 voiceMail, large font, 웹페이지 zooming, photo, map등 모두가 사용하기 쉽도록 하는 기능들을 포함하고 있었다.

다음의 특징을 통해 디바이스 사용이 불편한 사람들이 쉽게 사용하도록 만든다.

- Zoom: 전체화면을 확대한다.
- White on Black: 장치의 색을 화이트와 블랙만 지원.
- Mono Audio: 왼쪽과 오른쪽 채널의 사운드를 모노 신호로 결합한다.
- Speak Auto-text. 사용자가 입력하는동안 텍스트 수정 및 제안을 말한다.
- Voice Control: 사용자가 음성명령을 이용해 전화를 걸도록 만든다.

### ✏️ Accessbility and VoiceOver

VoiceOver는 유저가 화면을 보지않고도 제어할 수 있는 혁신적인 스크린을 읽는 기술이다. 이는 인터페이스와 사용자의 터치 사이에서 중재자 역할을 하여 요소 및 작업에 대한 음성 설명을 제공한다. 
VoiceOver가 활성화되어 있다면 실수로 전화를 걸거나 연락처를 삭제하는 행위를 막을 수 있다. 어떤 위치에 요소가 있고, 어떤 행동을 하며 어떤 결과를 초래하는지를 다 알려주기 때문이다.

하지만 접근가능한 유저 인터페이스 요소는 화면에서의 위치, 이름, 행동, 값, 타입에 대한 도움이 되고 정확한 정보를 제공해야 한다. 이를통해 말을 해주기 때문이다. 이는 Accessbility Inspector나 Interface Builder pane을 이용해서 디테일하게 설정할 수 있다.

### ✏️ Why You Should Make Your App Accessible

다음의 이유로 VoiceOver를 적용시켜야 한다.
- 유저 폭을 증가시킨다. 좋은 앱을 위해 열심히 만들었는데 많은 유저가 사용할 기회를 놓치지 않는다.
- 화면을 보지않아도 기능을 이용하게 만든다. 시각 장애가 있는 사용자들은 이를 통해 앱을 사용한다.
- 접근성 지침을 다루는데 도움이 된다. 다양한 관리 기관이 접근성을 위한 지침을 작성하고, VoiceOver사용자가 이 지침을 충족하는데 도움이 될 수 있다.
- 옳은 일이다.

접근성을 지원하는 것은 앱을 혁신적이고 멋있는 앱으로 만들지는 않는다. UI Accessiblity programming 인터페이스는 앱의 모양을 변경하지 않고 메인 로직을 방해하지 않는 기능을 추가할 수 있다.

### ✏️ iOS Accessibility API and Tools

iOS 3.0이후로 UI Accessibility programming interface는 앱이 VoiceOver를 이용하여 장애가 있는 사람들이 사용할 수 있도록 정보를 제공하는 lightweight API이다.

이는 UIKit의 부분으로 기본적으로 표준 UIKit control과 view에 구현되어져 있다. 즉 표준 컨트롤과 view를 사용하면 앱에 접근할 수 있도록 하는 많은 작업이 수행된다. 앱의 customization 수준에 따라 접근 가능한 사용자 인터페이스 요소에 대한 정확하고 유용한 설명을 제공하는 것만큼 간단하게 접근가능하게 만들 수 있다.

다음의 iOS SDK를 통해 앱이 접근가능하게 만든다.

- Interface Builder inspector pane. nib file을 디자인하는 동안 접근성 설명을 제공하는 쉬운 방법. [Interface Builder inspector pane](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/iPhoneAccessibility/Making_Application_Accessible/Making_Application_Accessible.html#//apple_ref/doc/uid/TP40008785-CH102-SW1)를 참고
- Accessibility Inspector. 앱의 사용자 인터페이스안에 삽입된 정보를 보여주고, 정보를 검증할 수 있게 만든다. 

### ✏️ The UI Accessibility Programming Interface

UI Accessibility Programming interface는 두개의 비공식 프로토콜, 하나의 클래스, 하나의 함수로 구성되어있다.

- `UIAccessibility` 비공식 프로토콜: 이 프로토콜을 구현한 객체들은 접근성 상태를 보고하고 설명이 있는 정보를 제공한다. 표준 UIKit 컨트롤과 뷰들은 기본적으로 이를 구현한다.
- `UIAccessibilityContainer` 비공식 프로토콜: 이 프로토콜은 `UIView`의 subclass가 객체의 일부 또는 전체를 별도의 요소로 엑세스 할도록 할 수 있게 만든다. 이러한 View에 포함된 객체 자체가 UIView의 하위클래스가 아니기에 자동으로 접근할 수 없는 경우 유용하다.
- `UIAccessibilityElement` 클래스: 이 클래스는 `UIAccessibilityContainer` 프로토콜을 통해 반환될 수 있는 객체를 정의한다. UIAccessibilityElement의 인스턴스를 만들어 UIView를 상속하지 않는 객체 또는 존재하지 않는 개체와 같이 자동으로 접근할 수 없는 항목을 나타낸다.
- `UIAccessibilityConstatns.h` 헤더파일: 이 파일은 접근성 요소가 나타낼 수 있는 특성과 앱에서 게시할 수 있는 알림을 설명하는 상수를 정의한다. 


### Accessibility Attributes

접근가능한 유저 인터페이스 요소를 설명하는 속성은 UI Accessibility API의 핵심으로 구성되어있다. VoiceOver는 컨트롤 또는 뷰에 접근하거나 상호작용할 때 사용자에게 속성 정보를 제공한다.

속성들은 가장 많이 사용하는 인터페이스의 구성요소기도 하다. 이는 속성들이 하나의 뷰나 하나의 컨트롤들을 다른 것이라고 차별화하는 정보를 캡슐화하고 있기 때문이다. 표준 UIKit 컨트롤과 view을 위해 기본 속성정보가 앱에 적합한지 확인해야 할 수도 있다.

UI Accessibility programming interface는 다음의 속성들을 정의한다.
- Label: 컨트롤 또는 뷰를 설명하는 간결한 현지화된 단어, 구절이다. 하지만 요소의 타입을 식별하지는 않는다.
- Traits: 하나혹은 여러개의 특성 결합. 동작 또는 사용법의 단일 측면을 설명한다. 예를 들어 키보드의 키처럼 행동하면서 현재 선택되었다는 것을 행동하는 요소는 키보드의 키와 선택되었다는 특성을 결합해서 있는 것이다. 즉 두개의 trait을 사용하고 있는 것.
- Hint: 요소의 action에 대한 결과를 설명하는 짧은 현지화된 구절. 예를 들어 "Adds a title", "Opens the shopping list"가 있다.
- Frame: 요소의 화면 위치와 크기를 지정하는 CGRect구조체에 의해 주어진 화면 좌표의 요소 프레임이다.
- Value: 요소의 현재 값. (레이블에 값이 표현되지 않을때). 예를 들어 slider는 "Speed"이지만 현재 값은 "50%"일 수도 있는 경우 value에 적을 수 있다.

접근성 요소들은 기본적으로 내용을 제공하는지에 관계없이 속성들을 제공한다. 접근성 요소는 항상 frame과 label 속성을 갖는다. frame 속성은 접근성 요소가 위치를 보고해야 하기 때문에 필수이고(UIView를 상속받으면 기본적으로 frame이 있다.) 레이블 속성은 이름과 설명을 VoiceOver가 읽기 때문에 필요하다.

접근성 요소에서 hint와 traits를 제공하는 것은 필수가 아니다. 어떤 요소가 action을 하지않는다면 hint를 사용할 필요가 없다.

value 속성은 요소의 값이 변할 때 사용한다. 예를 들어 이메일 주소를 포함하는 텍스트필드는 "Email address"라고 레이블을 짓지만 내용은 사용자 입력에 따라 달라진다. 

<img src="https://i.imgur.com/JhXAjPO.png" width="600"/>


## GCD(QoS, async, DispatchGroup, thread Safe, DispatchSemaphore)

### ✏️ DispatchQueue의 초기화

DispatchQueue를 커스텀하여 사용할 수 있다.

label을 제외한 파라미터들에는 기본값이 설정되어있기에 커스텀이 필요한 파라미터에 대해서만 초기화를 해줄 수 있다.

```swift
convenience init(label: String,
                 qos: DispatchQos = .unspecified,
                 attributes: DispatchQueue.Attributes = [],
                 autoreleaseFrequency: DispatchQueue.AutoreleaseFrequency = .inherit,
                 target: DispatchQueue? = nil)
```

### label

DispatchQueue의 이름을 설정해주는 파라미터이다. 이는 디버깅 환경에서 추적하기 위해 사용하는 String으로 식별자와 같다고 볼 수 있다.

### qos
QoS는 `Quality of Service`의 약자로 실행 될 Task들의 우선순위를 정해주는 값이다.

### attributes
DispatchQueue의 속성을 정해주는 값이다. `.concurrent`로 초기화하면 다중 스레드 환경에서 코드를 처리하는 큐가 될 것이고 빈 값이라면 `Serial`큐가 될 것이다.

`.initiallyInactive`는 후에 async, sync 코드블록을 호출한 직후에 큐에게 작업을 시키는 것이 아닌 `active()`를 호출할 때 작업을 처리한다.

### autoreleaseFrequency
DispatchQueue가 자동으로 객체를 해제하는 빈도의 값을 결정하는 파라미터이다. 기본값은 inherit이다.
- inherit: target과 같은 빈도
- workItem: workItem이 실행될때마다 객체들을 해지
- never: autorelease를 하지 않는다.

### target
코드 블록을 실행할 큐를 target으로 설정할 수 있다.

### ✏️ Qos
 QoS에서 우선순위는 *무엇을 먼저 처리할까? 무엇을 먼저 끝낼까?*와 같은 우선순위가 아니다. 여기서 말하는 우선순위는 무엇에 더 많은 에너지를 쏟을까?와 같은 맥락이다.
 즉 QoS가 높은게 절대적으로 작업의 순서를 결정짓는 것이 아니다. 하지만 어느정도는 영향을 미칠 것이다. 에너지를 많이 쏟는다는 것은 더 많은 스레드를 할당한다는 것이다. 
 
 큐를 초기화한 뒤에 async와 sync의 파라미터로 DispatchQoS의 우선순위를 할당할 수도 있다. 그리고 시스템은 QoS 정보를 통해 스케쥴링, CPU 및 I/O 처리량, 타이머 대기 시간 등의 우선순위를 조정한다. 
 
 DispatchQoS는 열거형 타입으로 총 6개의 QoS 클래스가 있으며 4개의 주요 유형, 2개의 특수유형으로 구분한다. 
 
 우선순위가 높을수록 더 많은 전력을 소모하기에 수행 작업에 적절한 QoS를 할당하면 앱이 더 반응적이고 보다 효율적인 에너지 사용이 가능해진다.
 
 ### User-interactive
 main thread에서 작업하며 사용자 인터페이스 새로고침, 애니메이션 등 사용자와 상호작용하는 작업에 할당. 작업이 빠르게 수행되지 않으면 유저인터페이스는 멈추게된다.
 
 ### User-initiated
 문서를 열거나 버튼을 클릭해 액션을 수행하는 것처럼 빠른 결과를 요구하는 유저와의 상호작용 작업에 할당한다. 몇 초 이내의 짧은 시간이내에 수행할 작업
 
 ### Default
 QoS를 할당하지 않을때 기본값으로 사용되며 중간레벨
 
 ### Utility
 데이터를 읽거나 다운로드처럼 작업이 완료되는데에 어느정도 시간이 걸리는 작업에 할당
 
 ### Background
 index 생성, 동기화, 백업 등 사용자가 볼 수 없는 백그랑누드 작업에 할당. 에너지 효율에 중점을 둔다.
 
 ### Unspecified
 Qos에 정보가 없음을 나타내며 시스템이 QoS를 추론해야 한다.
 
 ### ✏️ async
 
 async 메서드에는 4가지 파라미터가 존재한다.
 
 ```swift
func async(group: DispatchGroup? = nil, qos: DispatchQos = .unspecified, flags: DispatchWorkItemFlags = [], execute work: @escaping () -> Void)
```

### Group
DispatchQueue의 async 코드 블록을 묶어서 관리해주는 `DispatchGroup`이다. 여러 스레드에서 비동기로 작업을 처리하다보면 여러 개의 작업을 함께 관리해줘야 할 때가 있다.

### qos
적절한 케이스를 넣어주면 시스템이 알아서 관리한다.

### flags
코드 블록을 실행할 때 적용될 추가 속성을 결정한다. 기본 값으로는 빈 배열이며 여러 가지 속성을 한번에 부여할 수도 있다.

### ✏️ DispatchGroup
```swift
class DispatchGroup: DispatchObject
```

`DispatchGroup`은 비동기적으로 처리되는 작업들을 그룹으로 묶어, 그룹 단위로 작업 상태를 추적할 수 있는 기능이다. 이때 묶어줄 Async작업들이 꼭 같은 큐, 스레드에 있지 않더라도 묶어줄 수 있다.

DispatchGroup은 async에서만 사용할 수 있다. (sync는 작업종료시점을 알 수 있어 추적할 필요가없음.)

### ✏️ group에 등록하기: enter, leave

DispatchGroup을 사용하는 방법은 2가지가 있다.
- async를 호출하면서 파라미터로 group을 지정한다.
- enter, leave를 코드의 앞뒤로 호출하여 group을 지정해준다.

```swift
let group = DispatchGroup()

// enter, leave를 사용하지 않는 경우
DispatchQueue.main.async(group: group) {}
DispatchQueue.global().async(group: group) {}

// enter, leave를 사용하는 경우
group.enter()
DispatchQueue.main.async {}
DispatchQueue.global().async {}
group.leave()
```

### ✏️ notify
`notify`는 DispatchGroup의 업무 처리가 끝나는 시점에 원하는 동작을 수행하는 메서드이다.

```swift
let yellow = DispatchWorkItem {
    for _ in 1...5 {
        print("😀😀😀😀😀")
        sleep(1)
    }
}

let blue = DispatchWorkItem {
    for _ in 1...5 {
        print("🥶🥶🥶🥶🥶")
        sleep(2)
    }
}

let group = DispatchGroup()

DispatchQueue.global().async(group: group, execute: blue)
DispatchQueue.global().async(group: group, execute: red)

// 두 개의 async코드가 끝나고 난 후에 main큐에서 출력을 한다.
group.notify(queue: .main) {
    print("모든 작업이 끝났습니다.")
}
```

### wait

wait는 DispatchGroup의 수행이 끝나기를 기다리기만 하는 메서드이다. notify와 달리 별도의 코드를 실행하지 않아 코드블록을 실행시킬 queue를 지정할 필요가 없다.

```swift
let group = DispatchGroup()

DispatchQueue.global().async(group: group, execute: blue)
DispatchQueue.global().async(group: group, execute: red)

group.wait()
print("모든 작업이 끝났습니다.")
```

timeout 파라미터를 설정할 수도 있다. timeout에 10을 전달하면 group을 10초간 기다린다. 그 이후에는 다음 코드를 실행한다.

### Race Condition

동시에 작업을 처리하다보면 문제점이 생길 수 있다.

```swift
import Foundation

let arr = [1, 2, 3]

DispatchQueue.global().async {
    let element = cards.removeFirst()
    print("\(element)")
}


DispatchQueue.global().async {
    let element = cards.removeFirst()
    print("\(element)")
}


DispatchQueue.global().async {
    let element = cards.removeFirst()
    print("\(element)")
}
// output
// 1
// 1
// 1
```

위 코드는 하나의 배열에 여러 스레드가 동시에 접근하고 있기 때문에 일어나는 일로 이를 `Race Condition`이라고 한다. 

Swift의 대부분의 타입들은 `Thread Unsafe`하기 때문에 위와 같은 일이 일어나기 쉽다. 이 문제를 해결하기 위해 `DispatchSemaphore`가 존재한다.


### DispatchSemaphore

DispatchSemaphore는 공유 자원에 접근할 수 있는 스레드의 수를 제어하는 역할을 한다. 이는 semaphore count를 카운트하는 식으로 동작한다. 예를 들어 스레드가 접근하면 count에 -1을 접근이 끝나면 count에 +1을 해준다.

만약 허용된 스레드의 수만큼 접근된 상태라면 다른 스레드는 접근하지 못하고 줄을 서서 기다리게 된다.

```swift
let semaphore = DispatchSemaphore(value: 1) // count = 1

DispatchQueue.global().async {
    semaphore.wait() // count -= 1
    
    semaphore.signal() // count += 1
}
```

wait()는 값에 접근했다고 알리는 메서드, signal()을 볼 일 다봤다는 메서드이다.(이 메서드 둘은 반드시 짝지어서 호출해야한다.)

이를 사용해 위 Race Condition이 일어난 코드를 개선할 수 있다.
```swift
import Foundation

let arr = [1, 2, 3]
let semaphore = DispatchSemaphore(value: 1)

DispatchQueue.global().async {
    semaphore.wait()
    let element = cards.removeFirst()
    print("\(element)")
    semaphore.signal()
}


DispatchQueue.global().async {
    semaphore.wait()
    let element = cards.removeFirst()
    print("\(element)")
    semaphore.signal()
}


DispatchQueue.global().async {
    semaphore.wait()
    let element = cards.removeFirst()
    print("\(element)")
    semaphore.signal()
}
```

DispatchSemaphore는 스레드의 접근을 제어하기 위한 수단이며 Race Condition을 해결하기 위한 수단만은 아니다. 즉 스레드 접근 제어의 수가 2개라면 2개가 동시에 접근할 수 있기에 스레드 접근제어가 더 포괄적인 개념이다.

DispatchSemaphore사용없이 Race Condition을 해결하는 방법이 또 존재한다.

직렬 큐를 이용하는 것이다.
직렬 큐는 단일 스레드에서 동작하기에 들어온 순서대로 일을 처리한다. 

```swift
var arr = [1, 2, 3, 4, 5, 6, 7, 8, 9]
let serialQueue = DispatchQueue(label: "serial")

DispatchQueue.global().async {
    for i in 1...3 {
        serialQueue.async {
            let number = arr.removeFirst()
            print(number)
        }
    }
}

DispatchQueue.global().async {
    for i in 1...3 {
        serialQueue.async {
            let number = arr.removeFirst()
            print(number)
        }
    }
}

DispatchQueue.global().async {
    for i in 1...3 {
        serialQueue.async {
            let number = arr.removeFirst()
            print(number)
        }
    }
}
// 1부터 9까지 정상적으로 출력
```

<img src="https://i.imgur.com/MeGx2lY.png" width="500"/><br/>

### ✏️ UI 작업은 왜 Main Thread에서 해야할까?

UIKit 공식문서에는 다음과 같은 내용의 문구가 있다.
> UIKit 클래스는 main thread에서만 사용하세요. 이러한 제한 사항은 UIResponder에서 파생된 클래스 혹은 사용자가 UI를 통해 조작하는 것과 관련된 클래스에 적용되는 사항입니다.

### UIKit은 Thread Safe하지 않다.
즉 여러 스레드에서 제어할 수 있어 Race Condition이 일어날 수가 있다. 사용자가 버튼의 상태를 바꾸거나 특정행위에 의해 화면이 바뀌는 일이 여러 스레드에서 제어하게 된다면 의도하지 않은 화면을 보게 될 것이다. 이 때문에 직렬큐인 메인 큐를 통해 메인스레드에서만 동작하도록 하는 것이다.

### 그럼 왜 꼭 Main Thread인가? 다른 직렬 큐는 안되나?

메인 스레드에는 `Main RunLoop`가 존재하기 때문이다. 메인 스레드에서는 RunLoop가 일정한 주기를 유지하며 계속 동작하고 있다. 이 주기에 맞추어서 사용자의 입력을 받아 UI를 그리게 한다. 이러한 주기를 `View Drawing cycle`이라고 한다. 다른 스레드 고유의 RunLoop에서 UI를 그리게 된다면 UI가 그려지는 시점이 모두 제각각이기 때문에 통일한 Main Thread에서 동작하게 하는 것이다.




### 참고자료
- [Apple Archive - Accessibility Programming Guide for iOS](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/iPhoneAccessibility/Accessibility_on_iPhone/Accessibility_on_iPhone.html#//apple_ref/doc/uid/TP40008785-CH100-SW1)
- [야곰닷넷 - 동시성 프로그래밍](https://yagom.net/courses/%eb%8f%99%ec%8b%9c%ec%84%b1-%ed%94%84%eb%a1%9c%ea%b7%b8%eb%9e%98%eb%b0%8d-concurrency-programming/lessons/%eb%8f%99%ec%8b%9c%ec%84%b1-%ed%94%84%eb%a1%9c%ea%b7%b8%eb%9e%98%eb%b0%8d/)

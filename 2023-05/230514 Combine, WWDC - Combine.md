# 230514 Combine, WWDC - Combine

### 학습내용
- Combine
- WWDC Combine

## Combine
> 이벤트 처리 연산자를 결합함으로써 비동기 이벤트를 처리를 커스터마이즈 한다.

### Overview
Combine 프레임워크는 시간에 따라 값을 처리하는 선언적인 Swift API다. 이러한 값들은 많은 종류의 비동기 이벤트들을 나타낼 수 있다. Combine은 `publisher`를 선언하여 값이 시간에 따라 변할 수 있도록 드러낸다. 그리고 `subscriber`를 통해서 그러한 값들을 `publisher`로부터 받는다.

- `Publisher`프로토콜은 시간에 따라 값의 순서를 전달할 수 있는 타입을 선언한다. `publisher`는 upstream publisher로부터 값을 받고 그것들을 다시 publish할 수 있는 operator를 가지고 있다.
- `publisher`의 체인 끝에 `Subscriber`가 요소를 받는 대로 행동한다. `Publisher`는 명시적으로 `Subscriber`에서 요청한 경우에만 값을 내보낸다. 이는 `subscriber`코드에 존재하여 얼마나 `publisher`로부터 빠르게 이벤트를 받을지 제어한다.

몇개의 Foundation 타입은 그것의 기능을 `publisher`를 통해 드러낸다. `Timer`, `NotificationCenter`, `URLSession`이 그 예이다. Combine은 또한 `KVO`에 일치하는 프로퍼티를 위해 내장된 `publisher`를 제공한다.

여러개의 `publisher`의 출력을 결합할 수도 있고, 그것들의 상호작용을 조정할 수도 있다. 예를 들어 텍스트 필드의 `publisher`로부터 업데이트를 구독할 수 있다. 그리고 그 텍스트를 URL request를 수행하는데 사용할 수 있다. 그 후 또다른 `publisher`를 사용하여 응답을 처리하고 그것들을 앱 업데이트하는데 사용할 수 있다.

Combine을 사용해 이벤트 처리코드를 중앙화시키고 중첩 클로저나 컨벤션기반의 콜백과 같은 위험이 될만한 부분을 제거함으로써 코드읽기와 유지보수하기 쉽게 만들 수 있다.

## WWDC - Combine
기존에는 비동기 인터페이스들을 함께 구성하여 비동기 처리를 할 때 복잡성이 증가했다.
그래서 이 모든 것들을 대체하는 것이 아니라 그것들 사이에 공통적인 것들을 찾기 시작했다. 그 결과 값을 처리하기 위한 통합적 선언적 API인 Combine이 나왔다. 

컴바인은 Swift로 작성되어있기에 Generics와 같은 기능을 사용할 수 있다.
그리고 컴바인은 type safe하므로 런타임이 아닌 컴파일 시간에 오류를 포착할 수 있다. 

마지막으로 컴바인은 요청 기반이므로 앱의 메모리 사용량과 성능을 보다 신중하게 관리할 수 있다. 이를 가능하게 하는 핵심개념인 3가지 publisher, subscriber, operator가 존재한다.

### Publisher
게시자는 컴바인의 선언적 부분이며 값과 오류가 생성되는 방식을 설명하고 구독자 등록을 허용한다. 이는 시간이 지남에 따라 값들을 받는 것이다. 

```swift
protocol Publisher {
    associatedtype Output
    associatedtype Failure: Error
    
    func subscribe<S: Subscriber>(_ subscriber: S)
        where S.Input == Output, S.Failure == Failure
}
```
publisher에는 구독이라는 핵심기능이 존재한다. 위 함수에 대한 제약 조건처럼 Subscriber는 구독자의 입력이 게시자의 출력과 일치하고 구독자의 실패가 게시자의 실패와 일치해야 한다. 

publisher의 예시는 다음과 같다.

```swift
extension NotificationCenter {
    struct Publisher: Combine.Publisher {
        typealias Output = Notification
        typealias Failure = Never
        init(center: NotificationCenter, name: Notification.Name, object: Any? = nil)
    }
}
```
`Notification`을 대체하는 것이 아닌 컴바인에 적응시킨다는 것이 핵심이다. 

### Subscriber

구독자는 일반적으로 값을 수신하면 상태를 변경하고 작동하기 때문에 참조유형을 사용한다. 즉 클래스이다. 

구독자 프로토콜은 다음과 같다.

```swift
protocol Subscriber {
    associatedtype Input
    associatedtype Failure: Error
    
    func receive(subscription: Subscription)
    func receive(_ input: Input) -> Subscriber.Demand
    func receive(completion: Subscribers.Completion<Failure>)
}
```
구독은 구독자가 게시자에서 구독자로의 데이터 흐름을 제어하는 방법이다.
입력도 받을 수 있고 연결된 게시가자 유한한 경우 완료 또는 실패로 이어질 수도 있다.

```swift
extension Subscribers {
    class Assign<Root, Input>: Subscriber, Cancellable {
        typealias Faiure = Never
        init(object: Root, keyPath: ReferenceWritableKeyPath<Root, Input>)
    }
}
```

할당은 클래스이며 클래스의 인스턴스, 개체의 인스턴스 및 해당 개체에 대한 형식 안전 키 경로로 초기화된다. 이것이 하는 일은 입력을 받으면 해당 개체의 해당 속성에 입력을 쓰는 것이다.

구독자는 게시자의 반대편이다. 
게시자가 유한한 경우 완료를 포함하여 값을 받는다. 

### 사용예시

`Wizard`라는 모델 개체가 있고 마법사의 등급에만 관심이 있다고 가정해보자.
```swift
// Using Publisher and Subscriber
class Wizard {
    var grade: Int
}
let merlin = Wizard(grade: 5)
let graduationPublisher = NotificationCenter.Publisher(center: .default, name: .graduated, object: merlin)

let gradeSubscriber = Subscribers.Assign(object: merlin, keyPath: \.grade)

graduationPublisher.subscribe(gradeSubscriber)
```

학생들이 졸업한다는 알림을 듣고 학생들이 졸업하면 모델 객체의 값을 업데이트하고 싶다고 가정해보자.

하지만 위 코드는 컴파일에러가 나온다.
그 이유는 유형이 일치하기 않기 때문이다. 

<img src="https://hackmd.io/_uploads/S1BaeD0Vh.png" width="500"/><br/>

`NotificationCenter`는 알림을 생성하지만 정수 속성에 쓰도록 구성된 `Assign`은 정수를 예상한다. 따라서 필요한 것은 알림과 정수 사이를 변환하는 중간객체이다.
이를 오퍼레이터라고 한다. 
오퍼레이터는 게시자 프로토콜을 채택할 때까지 게시자다. 또한 선언적이므로 값 유형이기도 하다. 오퍼레이터의 주 역할은 값 변경, 값 추가, 값 제거 등 다양한 종류의 동작을 도와준다.

그리고 업스트림이라고 하는 다른 게시자를 구독하고 그 결과를 다운스트림이라고 하는 구독자에게 보낸다. 

다음은 연산자의 예이다.
```
extension Publishers {
    struct Map<Upstream: Publisher, Output>: Publisher {
        typealias Failure = Upstream.Failure
        
        let upstream: Upstream
        let transform: (Upstram.Output) -> Output
    }
}
```
Map은 연결하는 업스트림과 업스트림의 출력을 자체 출력으로 변환하는 방법으로 초기화되는 구조체이다. 

Map은 자체적으로 실패를 생성하지 않기 때문에 단순히 업스트림의 실패 유형을 미러링하고 그냥 통과한다.

```swift

let graduationPublisher = NotificationCenter.Publisher(center: .default, name: .graduated, object: merlin)

// ana
let gradeSubscriber = Subscribers.Assign(object: merlin, keyPath: \.grade)
let converter = Publishers.Map(
    upstream: graduationPublisher
) { note in
  return note.userInfo?["NewGrade"] as? Int ?? 0 
}

converter.subscribe(gradeSubscriber)
```
해당 클로저는 알림을 받고 NewGrade라는 사용자 정보 키를 찾는다. 이 키가 존재하면 정수이고 이 클로저에서 반환한다. 

이는 클로저의 결과가 정수이므로 구독자에 연결할 수 있다는 것을 의미한다.

```swift
extension Publisher {
    func map<T>(_ transform: @escaping (Output) -> T) -> Publishers.Map<Self.T> {
        return Publisher.Map(upstream: self, transform: transform)
    }
}
```

다음의 확장을 통해 더 편하게 값을 변경할 수 있다.

```swift
let cancellable = NotificationCenter.default.publisher(
    for: .graduated,
    object: merlin
).map { note in
          return note.userInfo?["NewGrade"] as? Int ?? 0
      }
 .assign(to: \.grade, on: merlin)
```

할당은 취소기능도 결합되어있다. 취소가 필요하면 게시자와 구독자 순서를 조기에 해제할 수 있다.
위 구문은 컴바인을 사용하는 방법의 핵심이다. 

이와 같은 연산자들을 Declarative Operator API라고 한다.
Filter, Reduce, Publisher의 첫 번째, 다섯 번째 요소를 가져오는 것과 같은 목록작업이 존재한다. 그리고 스레드나 큐 이동과 같은 백그라운드 스레드로 이동하거나 기본스레드로 이동하는 역할도 한다. 

연산자들은 너무 많이존재하기 때문에 컴바인의 핵심 디자인 원칙인 컴포지션을 항상 참고해야 한다. 많은 작업을 수행하는 몇 가지 연산자를 사용하는 대신 조금씩 많은 연산자를 제공해야 이해하기 쉽다.

<img src="https://hackmd.io/_uploads/BJZCuPRVn.png" width="500"/><br/>

단일값은 비동기식으로 나타내야 하는 경우 나중에 제공되므로 미래가 있다. 따라서 `Future`를 사용한다. 
많은 값을 비동기적으로 나타내야 하는 경우 게시자를 이용한다.

위 코드에서 `note.userInfo?["NewGrade]" as? Int ?? 0` 대신 잘못된 값이 진행되지 않도록 하고 모델 객체에 기록되는 것을 방지하는 것이 더 나을 것이다.

이를 `compactMap`으로 변경하여 스트림 아래로 더이상 내려가지 않도록 할 수 있다.
```swift
let cancellable = NotificationCenter.default.publisher(
    for: .graduated,
    object: merlin)
    .compactMap { note in 
        return note.userInto?["NewGrade"] as? Int            
    }
    .assign(to: \.grade, on: merlin)
)
```

.prefix나 .filter도 사용이 가능하다.

위 내용들은 동기이고
컴바인은 비동기 세계에서 작업할 때 빛을 발하기 시작한다.

### Zip, CombineLatest.

Zip은 여러 업스트림 입력을 단일 튜플로 변환한다. 
예를 들어 첫 번째 게시자가 A를 생성하고 두 번째 게시자가 1을 생성하면 이제 튜플을 생성하고 해당ㅇ 값 다운스트림을 구독자로 보낼 수 있다.

이는 여러개의 비동기 작업 결과를 기다리기 위해 3개의 업스트림을 사용하는 Zip버전을 사용한다. 
```swift
Zip3(organizing, decomposing, arranging)
    .map { $0 && $1 && $2 }
    .assign(to: \.isEnabled, on: continueButton)
```

CombineLatest는 Zip과 마찬가지로 여러 업스트림 입력을 단일 값으로 변환한다. 
이는 Zip과 달리 진행하려면 업스트림의 입력이 필요하므로 일종의 when/or 작업이 된다. 이를 지원하기 위해 각 업스트림에서 받은 마지막 값을 저장한다. 

첫 번째 게시자의 값과 두 번째 게시자의 값을 결합하여 새 값을 보낼 수 있다. 그리고 나중에 두 번째 게시자가 새 값을 생성하면 첫 번째 게시자의 이전 값과 결합하여 새 값을 보낼 수 있다. 
즉 업스트림 변경 사항에 따라 새 이벤트를 받는다.

CombineLatest는 값이 계속해서 방출될 수 있고 최근 값들을 조합해서 만든다. 
반면에 Zip은 짧은 게시자의 시퀀스를 따른다. 

### 참고문서
- [Apple Docs - Combine](https://developer.apple.com/documentation/combine)
- [WWDC - Combine](https://developer.apple.com/videos/play/wwdc2019/722/)

# 230516 Combine - Publisher, Subscriber, Subject

### 학습내용
- Publisher, Subscriber
- Subject

## Publisher, Subscriber

### Publisher, Subscriber 사이의 값 전달

#### Publisher
타입이 시간에 따라 일련의 값을 전송할 수 있음을 선언하는 프로토콜
하나 이상의 `Subscriber`인스턴스에게 element를 제공.

`Publisher`는 `receive(subscriber)`를 구현해 `subscriber`를 `accept`한다.

`Publisher`를 만드는 방법으로 `Future`, `Just`, `Deferred`, `Empty`, `Fail`, `Record`가 존재.

가장 간단한 `Just`를 통해 `Publisher`를 만들어보자.

`Just`는 subscriber에게 output은 한번만 출력한 다음 완료하는 publisher로  `Publisher`라는 프로토콜을 채택한 구조체.

```swift
let publisher = Just("Combine")
```

`Combine`이라는 `String`을 발행했는데 이 발행된 걸 받아서 쓰는 곳이 필요하다. 그게바로 `Subscriber`.

#### Subscriber
`Subsriber`도 프로토콜이고 정의는 `Publisher`로부터 input을 받을 수 있는 타입을 선언하는 것이다.

`Subscriber`는 `Publisher`에서 `sink`메서드를 통해 인스턴스를 만들 수 있다. 

`sink`메서드는 subscriber를 만들고 subscriber를 리턴하기 전에 즉시 무제한의 값을 요청한다. 

```swift
let publisher = Just("Combine")

let subscriber = publisher.sink { (value) in 
    print(value) // Combine                                
}
```

`sink`메서드에 `Completion`함수도 가능하다.

그리고 `sink`로 subscriber를 만들면 이 타입은 `AnyCancellable`이 된다.

그리고 이 `AnyCancellable`은 `Cancellable`프로토콜을 채택하고 있다.
`Cancellable`은 프로토콜로 작업이 취소될 수 있음을 지시하는 역할을 한다.

```swift
protocol Cancellable
```

`cancel()`메서드를 호출하면 할당된 자원이 해제된다. (타이머, 네트워크 접근, disk I/O.)

아무래도 subscriber가 구독을 해제하고 싶은 시점이 존재하기 때문에 해당 프로토콜을 따르는 것 같다. 그리고 이 취소는 `cancel`메서드를 호출함으로써 취소를 할 수 있다.

`AnyCancellable`문서를 확인해보면 
다음과 같이 정의되어 있다.

> 취소 시 제공된 클로저를 실행하는 타입을 지우는 취소가능한 객체.

subscriber를 만들때 취소할 때 동작하는 클로저를 제공하고 `cancel`을 호출 시 그 클로저를 호출하는 것으로 추측된다.

### subscriber를 생성하는 방법

잠시 Subscriber 공식문서의 개요를 훑고가보자.

`Subscriber`인스턴스는 `Publisher`로부터 요소들의 스트림을 받는다. 이 스트림은 이들간의 관계에 변화를 설명하는 라이프 사이클 이벤트와 함께 온다. 연관타입인 subscriber의 `Input`과 `Failure`는 publisher의 `Output`과 `Failure`가 일치해야 한다. 

publisher의 `subscribe(_:)`메서드를 호출함으로써 subscriber를 연결시킬 수 있다. 이 호출이 일어난 후 publisher는 subscriber의 `receive(subscription:)`메서드를 호출한다. 이는 subscriber에게 `Subscription` 인스턴스를 제공하는데 이 인스턴스는 publisher로부터 요구한 요소들을 사용하고 선택적으로 구독을 취소한다. subscriber가 초기 요구사항을 한 뒤에는 publisher가 새롭게 발행한 요소들을 전달하기 위해 `receive(_:)`를 비동기적으로 호출한다. 만약 publisher가 발행을 멈춘다면 발행이 정상적으로 완료되었는지 아니면 오류로 완료되었는지를 나타내는 Subscribers.Completion 유형의 매개 변수를 사용하여 receive(completion:)를 호출한다. 

Combine은 publisher의 연산자로 다음의 subscriber들을 제공한다.

- `sink(receiveCompletion:receiveValue:)`는 완료 신호를 수신할 때와 새 요소를 수신할 때마다 임의의 클로저를 실행한다.
- `assign(to:on:)`은 주어진 인스턴스의 key Path에 의해 식별되는 프로퍼티에 각각의 새롭게 받은 값을 쓴다.

요약하면
- publisher의 값을 subscribe하기 위해 subscribe, sink, assign 세 개중 하나를 선택한다.
- `subscribe`함수 호출 시 초기 구독이 생성되고 난 후의 `receive`는 비동기적으로 호출된다.

`subscribe`메서드를 사용해서 구독자를 만들어보자.

```swift
class MagazineSubscriber: Subscriber {
    // 이 연관값은 Publisher의 값과 동일해야 함.
    typealias Input = String
    typealias Failure = Never
    
    // publisher에 subscribe로 부착되는 순간 호출되는 메서드
    // 성공적으로 구독했음을 알리고 item 요청.
    func receive(subscription: Subscription) {
        print("잡지구독 시작")
        subscription.request(.unlimited)
    }
    
    // publisher로부터 요소를 받고 subscription을 통해
    // 앞으로 publisher에게 얼마나 요소를 요청할 지를 반환한다.
    func receive(_ input: String) -> Subscribers.Demand {
        print("\(input)")
        return .none
    }
    
    // publisher가 subscriber에게 성공적으로 모든 것을 발행 혹은 오류로 완료되었음을 알림.
    func receive(completion: Subscribers.Completion<Never>) {
        print("완료.", completion)
    }
}

let publisher = ["GQ스타일", "싱글즈", "마리끌레르", "뷰티쁠"].publisher
publisher.subscribe(MagazineSubscriber())
```

그리고 일일이 `print`를 넣어주지 않고 다음과 같이 `publisher`에서 `print`를 제공하고 있다.
```swift
publisher.print().subscribe(MagazineSubscriber())
// receive subscription: (["GQ스타일", "싱글즈", "마리끌레르", "뷰티쁠"])
// request unlimited
// receive value: (GQ스타일)
// receive value: (싱글즈)
// receive value: (마리끌레르)
// receive value: (뷰티쁠)
// receive finished
// 완료. finished
```
이렇게 `Subscriber`를 커스텀하게 만들 수 있지만... sink가 있으니 언제 사용할 지는 감이 잘 오지않는다.
상황에 따라 몇개만 발행받고 싶을때 사용할지도 모르겠다.

## Subject
> 외부 호출자가 요소를 publish할 수 있는 메서드를 드러내는 publisher

```swift
protocol Subject<Output: Failure> : AnyOjbect, Publisher
```

Subject는 `send(_:)`` 메서드를 호출하여 스트림에 값을 "주입"하는 데 사용할 수 있는 publisher이다. 이는 기존 명령형 코드에 컴바인 모델을 적용시킬 때 유용하다. 

컴바인에는 `Subject`프로토콜을 채택한 구체 클래스가 2개가 존재한다.
- CurrentValueSubject
- PassthroughSubject

### CurrentValueSubject
> 단일 값을 래핑하고 값이 변경될 때마다 새로운 값을 발행한다.

```swift
final class CurrentValueSubject<Output, Failure> where Failure: Error
```

`PassthroughSubject`와는 다르게 `CurrentValueSubject`는 가장 최근에 발행된 요소의 버퍼를 유지한다. 

`send(_:)`를 호출함으로써 현재 값을 업데이트한다.

```swift
let currentValueSubject = CurrentValueSubject<String, Never>("Brody")
let subscriber = currentValueSubject.sink(receiveValue: {
    print($0)
})
currentValueSubject.value = "Hi"
currentValueSubject.send("하이")
```

`CurrentValueSubject`는 초기값을 주어야 한다. 안그러면 컴파일 에러가 나온다.

### PassthroughSubject
> downstream 구독자들에게 요소를 broadcast하는 subject

```swift
final class PassthroughSubject<Output, Failure> where Failure : Error
```

`Subject`의 구체 클래스인 `PassthroughSubject`는 존재하는 명령형 코드를 컴바일 모델에 적용시키는데 편리한 방법을 제공한다. 

`CurrentValueSubject`와는 다르게 초기값과 가장 최근발행된 요소의 버퍼를 가지지 않는다. 
PassthroughSubject는 구독자가 없거나 현재 수요가 0인 경우 값을 삭제한다.

```swift
let passthroughSubject = PassthroughSubject<String, Never>()
let subscriber = passthroughSubject.sink(receiveValue: {
    print($0)
})
passthroughSubject.send("hi")
passthroughSubject.send("하이")
```

`PassThroughObject`는 최근 버퍼를 가지지 않기 떄문에 `value`속성이 존재하지 않는다.

`Subject`는 `send`라는 메서드를 통해 값을 스트림에 주입할 수 있는 `Publisher`이다.

이는 나중에 값이 패치될 수 있는 경우에 사용될 수 있을 것 같다. 예를 들어 다음의 코드가 가능해진다.

```swift
enum SomeError: Error {
    case unknown
}

let passthroughSubject = PassthroughSubject<String, Error>()
let subscriber = passthroughSubject.sink { result in
    switch result {
    case .finished:
        print("finished")
    case .failure(let error):
        print(error)
    }
} receiveValue: { value in
    print(value)
}

passthroughSubject.send("안녕")
passthroughSubject.send("brody")

passthroughSubject.send(completion: .failure(SomeError.unknown))

passthroughSubject.send("hi")
```

기존에는 Error를 Never로 처리하고 있었는데 이를 Error로 변경함으로써 에러에 대한 처리도 구현할 수 있게 되었다. 

또한 완료 혹은 실패를 Subject에 전달하게 되면 해당 Subject가 완전히 종료되기 때문에 이후 send를 하더라도 값을 발행하지 않는다.



### 참고문서
- [Zedd - Publisher, Subscriber](https://zeddios.tistory.com/925)
- [Zedd - Subject](https://zeddios.tistory.com/965)
- [Apple Docs - Cancellable](https://developer.apple.com/documentation/combine/cancellable)
- [Apple Docs - AnyCancellable](https://developer.apple.com/documentation/combine/anycancellable)
- [Apple Docs - Subscriber](https://developer.apple.com/documentation/combine/subscriber)
- [Apple Docs - Subject](https://developer.apple.com/documentation/combine/subject)
- [Apple Docs - CurrentValueSubject](https://developer.apple.com/documentation/combine/currentvaluesubject)
- [Apple Docs - PassthroughSubject](https://developer.apple.com/documentation/combine/passthroughsubject)

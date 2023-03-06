230306 WWDC - Concurrency Programming Guide GCD in Swift3
===
학습내용
---
- WWDC - Concurrency Programming Guide GCD in Swift3

## WWDC - Concurrency Programming Guide GCD in Swift3

애플리케이션의 메인 스레드에서 대규모 작업(이미지 처리, 데이터 로드)를 하게 된다면 spinning wheel appearing같이 UI가 느려지거나 중지될 수 있다. 

이는 하나의 스레드에서 모든 걸 처리하기 위해 나타나는 현상이기에 동시성을 도입하면서 접근할 수 있다.

thread를 만들어 concurrent한 환경을 만들 수는 있지만 이는 코드 불변성을 초래할 수 있다. 

GCD는 다중스레드 코드를 작성할 수 있게 도와주고 스레드 위에 추상적인 개념을 도입한다.


### Controlling concurrency

응용 프로그램에서는 동시성을 제어하는 비용이 든다.
스레드 풀은 장치의 모든 호출을 사용하기 위해 달성한 동시성을 제한한다. 
이는 코드를 실행하는데 사용할 발송 대기열의 올바른 개수를 선택하는 것이 매우 중요하다는 것을 의미한다. 자세한 건 thread explosion을 알아본다.

### Structuring Your Application

애플리케이션 영역을 독립적인 데이터 흐름으로 식별하는 것이 좋다.
이는 이미지 변환이나 데이터베이스가 있을 수 있다. 이러한 영역을 서로 다른 하위 시스템으로 분할한 다음 각 전송 대기열을 사용하여 하위 시스템을 백업한다.
이렇게 하면 너무많은 대기열과 너무 많은 스레드의 문제를 겪지 않고 각 하위 시스템에서 작업을 독립적으로 실행할 수 있는 대기열이 제공된다.

### Grouping Work
하나의 블록을 다른 블록과 비동기 대기열로 비동기화 한 다음 기본 대기열로 다시 동기화 할 수 있다. 이는 그룹화 작업이고 그 작업이 끝나기를 기다리는 것이다.

여러 개의 서로 다른 작업항목을 생성하려는 단일 작업학목이 있고 해당 작업이 완려되었을 때만 작업을 진행하려는 경우 해당 작업을 수행할 수 있다.

<img src="https://i.imgur.com/AOUaEKy.png" width="300"/>


### DispatchGroup
Work라는 작업을 Dispatch에 Submit하면 비동기 호출에 그룹을 선택적 매개 변수로 추가할 수 있다.

해당 그룹에 많은 작업을 추가하고 다른 큐에 작업을 수행할 수 있으면서도 동일한 그룹과 연관시킬 수 있다.
그리고 DispatchGroup에 Work를 Submit할 때마다 Group이 완료될 것으로 예상되는 항목의 카운터를 증가시킨다.

<img src="https://i.imgur.com/XwyWncP.png" width="700"/>

마지막으로 모든 작업이 끝난 후에는 `(notify(queue:))`를 통해 선택한 대기열에서 작업을 지시할 수 있다.

### Using Quality of Service Classes
QoS 품질에 따라 우선순위를 제어할 수 있다. 
dispatch queue에서 비동기(async)가 실행되면 실행 컨텍스트가 캡쳐된다. 즉 디스패치 큐에 제출하는 하는 시간의 QoS가 사용된다.


<img src="https://i.imgur.com/5HrOYv3.png" width="400"/>


#### DispatchWorkItem

DispatchWorkItem의 `allocateCurrentContext`옵션을 사용하면 DispatchQueue에 Submit하는 시간이 아닌 실행 컨텍스트의 QoS가 사용된다.

예를 들어 assignCurrentContext를 사용해 만든 작업항목은 작업항목을 작성할 때의 QoS를 사용하기에 이를 저장하고 있다가 실행할 때 갖고있는 속성값을 함께 디스패치에 제출한다.

<img src="https://i.imgur.com/gGTYZVs.png" width="600"/>

DispatchWorkItem의 wait을 이용하여 신호를 보낼 수 있다. 

### Swift 3 and Synchoronization

글로벌 변수는 Atomic(원자적)으로 초기화된다.
- 프로그램이 실행되면서 메모리에 할당되기 때문에 당연히 원자적

클래스, lazy 속성은 Atomic(원자적)이지 않다.
- 초기화 컨텍스트를 실행하는 2개 이상의 다른 명령에서 동시에 두 번 생성할 수 있다.

### Lock

전통적으로 동기화를 하기위해 C Lock이나 Foundation.Lock을 사용하지만 다음의 이유로 DispatchQueue를 사용하는 것이 좋다.
- 오용하기 훨씬 쉽다
- 디버깅 도구에서 Xcode의 실행 시간과 훨씬 더 잘 통합되어 있다.

### Use Explict Synchronization

동기화를 이용해 내부 상태를 반환하면 상호배제를 할 수 있다.

#### Getter

```swift
class MyObject {
    private let internalState: Int
    private let internalQueue: DispatchQueue
    var state: Int {
        get {
            return InternalQueue.sync { internalState }
        }
    }
}
```

#### Setter
```swift
class MyObject {
    private let internalState: Int
    private let internalQueue: DispatchQueue
    var state: Int {
        get {
            return internalQueue.sync { internalState }
        }
        set (newState) {
            internalQueue.sync { internalState = newState}
        }
    }
}
```

클래스의 프로퍼티에 접근하는 동시적인 작업이 있는 경우에는 위처럼 하나의 큐에 sync를 이용해서 동기화를 한다. 


### 참고자료
- [WWDC - Concurrent Programming With GCD in Swift 3](https://developer.apple.com/videos/play/wwdc2016/720/)

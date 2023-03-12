230311 DispatchGroup과 DispatchSemaphore의 사용, Operation
===
학습내용
---
- DispatchGroup과 DispatchSemaphore의 사용
- Operation

## DispatchGroup과 DispatchSemaphore의 사용

비동기적으로 데이터를 불러와 콜백함수의 인자로 전달하는 예시를 살펴보자.

```swift
import Foundation

func fetchData(_ data: Int, delay: Double, completionHandler: @escaping (String)->()) {
    print("fetchData - 진입")
    
    DispatchQueue.main.asyncAfter(deadline: .now() + delay) {
        print("fetchData - data \(data)")
        
        completionHandler("\(data)")
    }
    
    print("fetchData - 종료")
}

func combineAsyncCalls(completionHandler: @escaping (String) -> ()) {
    print("combineAsyncCalls - 진입.")
    
    var text = ""
    fetchData(0, delay: 0.4) { text += $0}
    fetchData(1, delay: 0.2) { text += $0}
    print("combineAsyncCalls - text \(text)")
    
    completionHandler(text)
    
    print("combineAsyncCalls - 종료")
}

combineAsyncCalls {
    print("combineAsyncCalls - 클로저 메소드 진입 ")

    print($0) // 빈공백

    print("combineAsyncCalls - 클로저 메소드 종료 ")
}
```

### 출력문
<img src="https://i.imgur.com/yk12Unb.png" width="400"/><br/>



`combineAsyncCalls`함수 내에서 `fetchData`를 호출하지만 `fetchData`내부에는 `DispatchQueue.main.asyncAfter`라는 비동기 함수가 있기 때문에 기다리지 않고 다음 코드로 이동하게 된다. 만약 이 코드 대신에 `DispatchQueue.global().sync`가 있다면 `fetchData`가 끝난 후에 다음의 `fetchData`를 호출할 것이다.

하지만 목적은 그게 아니다. 순서에 종속되지않는 100개의 작업이 있다고 할 때 100개의 작업을 전부 기다리는 것은 성능적으로 좋지않다. 

#### DispatchGroup으로 해결

그래서 순서가 중요하지않는 여러개의 작업들을 그룹으로 묶어 이 그룹의 동작을 동기화할 수 있는 객체가 바로 `DispatchGroup`이다. 
`DispatchGroup`은 `wait()`을 통해 내부의 비동기작업들이 끝날때까지 기다릴 수도 있고, `notify`를 통해 실행블록을 등록만하고 비동기작업들이 끝난 후에 호출하게 만들 수 있다.

`notify`를 이용해 `combineAsyncCalls`함수를 다음과 같이 수정하면 해결이 된다.

```swift
func combineAsyncCalls(completionHandler: @escaping (String) -> ()) {
    let dispatchGroup = DispatchGroup()
    print("combineAsyncCalls - 진입.")
    
    var text = ""
    dispatchGroup.enter()
    fetchData(0, delay: 0.4) {
        text += $0
        dispatchGroup.leave()
    }
    dispatchGroup.enter()
    fetchData(1, delay: 0.2) {
        text += $0
        dispatchGroup.leave()
    }
    dispatchGroup.notify(queue: .main) {
        completionHandler(text)
    }
    
    print("combineAsyncCalls - 종료")
}
```

처음에는 `dispatchGroup.leave`를 `fetchData의` 후행 클로저 내부에 사용하는 것이 아닌 `fetchData`호출 이후에 적었다. 하지만 시간을 기다리지않고 다음 줄을 실행하기 때문에 제대로 동작하지 않았다. 정확히는 특정시간이 지난 이후에 실행되는 후행클로저 내부에 쓰는 것이 맞는 것이다.

그리고 위 코드는 콘솔앱에서 사용할 때 주의점이 있다. 콘솔앱은 UI앱과는 다르게 메인스레드가 항상 동작하고 있지 않는다. 따라서 `notify`를 하는 순간 다음 코드로 실행이 되고 함수가 종료되면 메인 스레드에 작업이 존재하지않아 종료될 수 있다. 
`completionHandler`가 존재하는지 존재하지않는지는 이 부분에서 중요하지 않다. 호출부의 후행 클로저로 적힌 코드들이 실행이 되지 않기 때문이다.

#### DispatchSemaphore로 해결

`DispatchSemaphore`를 통해 스레드의 수를 제어해 동기적으로 실행시킬 수 있다. 스레드의 수를 여러개로 설정해 동시에 작업할 수는 있지만 그렇게 하게되면 맨 처음 예시와 동일하게 빈 텍스트를 전달하게 될 것이다. 

아래의 경우 처음 세마포어의 수를 0으로 설정해서 동기적으로 만들었는데 `DispatchGroup`과는 동작이 다르다. 
아래 코드는 sync로 작업하는 것과 동일하게 동작한다. 왜냐하면 `signal()`을 하는 순간 세마포어가 1증가하는데 그전까지는 `wait()`을 통해 동작을 막았기 때문이다. (`wait`이 있으면 세마포어가 0이하일 때 동작하지않는다. 1이상이라면 세마포어를 1감소하고 다음블록을 실행)

```swift
func combineAsyncCalls(completionHandler: @escaping (String) -> ()) {
    let dispatchSemaphore = DispatchSemaphore(value: 0)
    print("combineAsyncCalls - 진입.")
    
    var text = ""
    // 메인스레드에서 세마포어를 사용할 수 없기에 global큐에서 작업.
    DispatchQueue.global().async {
        fetchData(0, delay: 0.4) {
            text += $0
            dispatchSemaphore.signal()
        }
        dispatchSemaphore.wait()
        fetchData(1, delay: 0.2) {
            text += $0
            dispatchSemaphore.signal()
        }
        dispatchSemaphore.wait()
        
        completionHandler(text)
    }
}
```

### 결론
비동기 기능을 동기화하기위한 두 가지 도구로 `DispatchGroup`과 `DispatchSemaphore`가 존재하는데 순서에 관계없이 모든 작업이 완료될 때까지 기다려야한다면 `DispatchGroup`을, 순서대로 실행하고 완료하려면 `DispatchSemaphore`를 사용한다.

## Operation

Operation은 GCD위에서 동작하며 GCD를 객체지향적으로 새롭게 재탄생시킨 고수준 API이다. 

이는 GCD에서 Task라고 불렀던 코드블록들을 캡슐화, 객체화(타입화)한 것으로 이를 이용하면 다음과 같은 장점이 있다.
- 재사용 용이
- 타입 간의 관계 설정
- 다양한 프로퍼티 활용
- 스케쥴링 용이

이렇게 만들어진 Operation객체들을 코드레벨에서는 `BlockOperation`으로 만들고 `OperationQueue`라는 Queue에 넣어서 실행 및 관리를 한다.

```swift
let order1 = BlockOperation {
    print("알리오올리오")
}

let order2 = BlockOperation {
    print("바닐라 아이스크림")   
}

let orderBar = OperationQueue()
orderBar.maxConcurrentOperationCount = 2 // 같은시간에 동작하는 작업들의 최대 수

orderBar.addOperation(order1)
orderBar.addOperation(order2)
```

### ✏️ Operation 기본

GCD의 `DispatchWorkItem`과 유사하지만 더 많은 기능을 가지고 있는 것이 `Operation`이다. `Operation`과 `OperationQueue`에서는 객체지향적으로 설계를 했기 때문에 좀 더 세부적인 스케쥴링이 가능하다.

```swift
class Operation: NSObject
```

`Operation`은 추상클래스이기에 항상 이를 상속받는 타입을 사용해야 한다. 하위 클래스는 커스텀 클래스로 만드는 방법과 기존에 구현되어있는 `BlockOperation`을 사용하는 방법이 존재한다.

```swift
let operation = BlockOperation { }

operation.addExecutionBlock { }

operation.completionBlock = { }
```

`addExecutionBlock`함수는 `Operation`의 동작이 끝난 후 실행된다. 
`completionBlock`은 `Operation`의 동작과 `addExecutionBlock`의 동작이 끝난 후 실행된다.

### Note
기본적으로 `start()`로 실행하면 현재 스레드에서 `Operation`을 실행한다. 구체적으로는 `Operation`이 동기라면 현재 스레드, 비동기라면 새로운 스레드를 만들어서 `Operation`을 처리한다. 그리고 `OperationQueue`에 넣는 경우는 `Operation`을 각각 새로운 스레드를 만들어 작업한다.

#### Operation의 프로퍼티
```swift
var isCanclled: Bool
var isExecuting: Bool
var isFinished: Bool
var isConcurrent: Bool
var isAsynchronous: Bool
var isReady: Bool
var name: String?
```

상태를 추적하는 프로퍼티들은 `Operation` 상태에 따라 자동으로 값을 변경해준다. 

공식문서에서는 `isConcurrent`대신 `isAsynchronous` 값을 사용하길 권장한다.

<img src="https://i.imgur.com/mqhTRN8.png" width="600"/>

### ✏️ Operation 심화

### Cancel
Operation의 상태에는 `isCancelled`라는 프로퍼티가 있다. `operation.cancel()`을 통해 이 프로퍼티를 `true`로 변경할 수는 있지만 실제로 `Operation`의 동작을 직접적으로 취소하진 않는다.

`cancel()`을 호출한 후 `isCanceled`프로퍼티를 추적하는 것을 통해 작업을 관리해줘야 한다.

### isAsynchronous

이 프로퍼티는 `Operation`이 `Asynchronous`로 동작할 것인가, `Synchoronous`로 동작할 것인가에 대한 프로퍼티이다.
이는 `Operation`을 `start()`해서 직접 실행하는 것과 `OperationQueue`를 사용할 때 두가지로 나누어서 볼 수 있다.

먼저 직접 실행할 경우, 동기라면 `Operation`을 실행한 현재의 스레드에서 작업을 처리하고, 비동기일 때는 새로운 스레드를 만들어서 작업을 처리한다.

그런데 만약 `OperationQueue`에 넣어서 작업을 처리하면 `isAsynchronous`의 값과는 상관없이 모두 새로운 스레드를 만들어 처리한다.

`Operation`의 `isAsynchoronus`프로퍼티 기본값은 `false`로 동기로 동작한다. 만약 비동기로 동작하는 `Operation`이 필요하면 Custom해서 Operation을 만들어줘야 한다.

### Note
수동으로 Operation을 실행시켜주는 건 왜 권장하지 않을까?

수동으로 하는 경우 비동기로 처리하고 싶을 때 `isAsynchronous` 상태 값을 신경써야 한다. 하지만 `OperationQueue`를 이용하면 동기/비동기 상태와는 상관없이 모두 새로운 스레드에서 작업을 하기 때문에 효율적으로 `Operation`을 관리할 수 있게된다.


### Dependency
종속성은 Operation 인스턴스들 사이에 종속적인 관계를 만들어서 실행의 순서를 관리하도록 해준다.
기본적으로 `Operation`은 직접 실행한다면 실행한 순서대로, `OperationQueue`에 넣으면 넣은 순서대로 작업을 처리하는데 `addDependency(_:)`메서드로 종속적인 관계를 만들어주면 해당 `Operation`은 자신이 종속된 `Operation` 작업이 끝날 떄까지 실행할 수 없게 된다.

```swift
import Foundation

let yagom = BlockOperation {
    print("🐻🐻🐻🐻🐻")
}

let noroo = BlockOperation {
    print("🦌🦌🦌🦌🦌")
}

let odong = BlockOperation {
    print("🌳🌳🌳🌳🌳")
}

odong.addDependency(yagom)
odong.start()
// error!
```

위 코드에서 odong은 yagom의 실행이 끝나야 비로소 `isReady`상태가 되기에 에러가 난다. 

종속성은 `Operation`이 서로 다른 `OperationQueue`에 있더라도 유효하며, 종속성 제거는 `removeDependency()`로 제거한다. 

### waitUntilFinshed()
`Operation`이 동작하고 있는 스레드에서 해당 `Operation`이 끝날 때까지 기다리는 메서드이다. 이 메서드를 사용할 때에는 교착 상태에 빠지지 않도록 주의해야 한다.

### OperationQueue

sync/async에 관한 정보는 `Operation`이 가지고 있고, 스레드 관리는 `OperationQueue`가 하는 식으로 역할이 분리되어있다. 

`operation`을 큐에 추가하면 실행될 때까지 `OperationQueue`에 남아있으며 직접 삭제할 수는 없다.

#### main, current
```swift
class var main: OperationQueue
class var current: OperationQueue?
```
`current`는 현재 작업을 시작한 `OperationQueue`를 반환한다. 대기열을 확인할 수 없는 경우에는 nil이 될 수 있기 때문에 옵셔널 타입으로 정의되어 있다. `main`은 메인스레드에서 동작하는 `OperationQueue`를 반환한다.

#### 사용예시
```swift
let queue = OperationQueue()
let mainQueue = OperationQueue.main
let currentQueue = OperationQueue.current
```
#### addOperation
```swift
func addOperation(_ op: Operation)
func addOperations(_ ops: [Operation], waitUntilFinshed: Bool)
func addOperation(_ block: @escaping () -> Void)
```

#### cancelAllOperations()
대기열에 있는 모든 Operation의 `cancel()`메서드를 호출한다.



### 참고자료
- [DispatchGroup과 DispatchSemaphore](https://qteveryday.tistory.com/308)
- [야곰닷넷 - 동시성프로그래밍](https://yagom.net/courses/%eb%8f%99%ec%8b%9c%ec%84%b1-%ed%94%84%eb%a1%9c%ea%b7%b8%eb%9e%98%eb%b0%8d-concurrency-programming/)

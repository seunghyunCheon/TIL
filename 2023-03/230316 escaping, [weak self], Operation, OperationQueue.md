230316 escaping, [weak self], Operation, OperationQueue
===
학습내용
---
- escaping, [weak self]
- Operation, OperationQueue
- Custom Operation

## escaping, [weak self]

escaping클로저는 non-escaping 클로저와 다르게 아래와 같은 동작이 가능하다.
- 파라미터로 전달된 클로저를 외부 변수/상수에 저장 가능
- 함수가 종료된 뒤 실행 가능

escaping클로저의 대표적인 예로 비동기 처리가 있다.
함수의 실행 순서를 정할 수 있기 때문에 실행 순서를 보장받을 수 있다.

```swift
class Server {
  static var values: [Value] = []

  static getValues(completion: @escaping (Bool, [Value]) -> Void) {
      //2. 서버로 request(요청)
      Alamofire.request(urlRequest).responseJSON { response in
          //3. response 전달 받음
          persons.append("value")
          DispatchQueue.main.async {
              completion(true, values)
          }

      }
  }
}

//1. getValues 실행 -> values를 요청
Server.getValues { (isSuccess, values) in
  // 4. response에 따른 처리
  if isSuccess {
      
  }
}
```

`getvalues`메서드의 클로저 인자는 escaping이기 때문에 함수 안에서 꼭 바로 클로저를 시킬 필요가 없다. 

즉 `Almofire.request`로 값을 보내고 바로 종료하고, 특정 시간 후에 response가 도착했을 때 escaping 클로저에 인자를 넣어 실행시키고 이를 호출한 부분에서 인자들을 받아서 처리할 수 있는 것이다.

### weak self는 꼭 사용해야 할까?

기본적으로 non-escaping 클로저에서는 순환 참조가 발생할 수 없다. 
함수를 호출하고 종료할 때까지 클로저가 실행이 되어야하기 때문이다. 

`map`함수와 같은 고차함수들은 non-escaping 클로저이다.

`asyncAfter`메서드는 `escaping` 클로저를 받고는 있지만 특정 시간이후에는 클로저를 실행해 클로저가 메모리에서 제거된다. 그렇기 때문에 [weak self]를 사용하지 않아도 된다.

하지만 [weak self]를 사용할 때와 사용하지않을때 결과가 다를 수 있기 때문에 이에 주의해야 한다.

```swift
class Class2 {

    func format(_ value: Int) -> String { return String(value) }

    func test() {
        DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
            let formatted = self.format(10)
            print("formatted: \(formatted as Any)")
        }
    }
    
    deinit {
        print("Class2 deinit")
    }
}

class Class1 {
    let class2: Class2

    init() {
        class2 = Class2()
    }

    func someEvent() {
        class2.test()
    }

    deinit {
        print("Class1 deinit")
    }
}

var class1: Class1? = Class1() // 1
class1?.someEvent() // 2
class1 = nil // 3
// Class1 deinit
// formatted: 10
// Class2 deinit
```

위 예시는 class2인스턴스의 test함수 내에서 self참조를 강하게 하고 있기 때문에 reference counting이 2이다. 
그래서 Class1이 nil이 된다고 해도 Class2가 1이 계속 남아있게 된다. 
즉 살아있는 인스턴스를 계속 참조하고 있기 때문에 formatted에 10이 되는 것이다.

만약 위 예시에서 class2인스턴스의 test 함수 내에서 [weak self]를 사용한다면 결과는 달라진다.

class2의 reference counting이 1이기 때문에 class1이 nil이 된다면 바로 class2가 메모리에서 해제된다. -> 결국 클로저에서 가리키는 self가 nil이 되어 값도 nil이 된다.
### weak self 시 출력
```swift
// Class1 deinit
// Class2 deinit
// formatted: nil
```

따라서 값을 참조하고 있는 부분이 존재할 수 있기 때문에 무조건적으로 [weak self]를 사용하는 것을 지양해야 한다.

## Operation, OperationQueue

GCD는 멀티코어 하드웨어 환경에서 동시성을 지원하기 위해 사용하는 기술이다. 

`Operation`과 `OperationQueue`는 GCD위에서 동작한다. 애플은 먼저 고수준의 추상화레벨인 이를 고려하고 GCD가 필요하다면 GCD를 사용하도록 권장한다.

다음의 특징을 통해 두 가지 중 하나를 선택할 수 있다.
- GCD: 동시에 실행되는 작업단위를 보여주는 가벼운 방법. 이 작업들을 스케쥴하지 않는다면 사용한다. 즉 시스템이 직접 스케쥴링하는 것. 때문에 의존성을 추가하는 것은 성가시다. 
- Operation: GCD에 비해 오버헤드는 약간 더 많지만 의존성을 쉽게 추가할 수 있고 재사용, 취소, 중지하는 등의 다양한 작업이 가능하다.

테이블 뷰에 이미지들을 불러오는 경우에는 성능과 전력소비 문제를 고려해, `Operation`이 좋은 선택일 수 있다. 만약 유저가 스크롤해서 이미지를 화면 밖으로 내보냈을때 구체적인 이미지에 대한 작업을 취소할 필요가 있기 때문이다. 작업이 백그라운드 스레드에 있고 대기 중인 작업이 수십개가 존재한다면 성능이 계속 저하된다.


## Custom Operation

### main()과 start()

<img src="https://i.imgur.com/eQHyK5J.png" width="500"/><br/>

`OperationQueue`에서 `Operation`을 실행시킬 때에는 `start()`메서드를 호출해준다. `main`을 private으로 설정하지 않은 이뉴는 상속을 통한 재정의가 가능해야하기 때문

Operation을 custom할 경우 Operation이 동기인지 비동기인지에 따라 재정의해야하는 메서드가 다르다. 
`main()`은 non-concurrent(Sync), `start()`는 concurrent(Async) Operation에 대해 실행된다고 한다.

### Custom Synchronous Operation
동기 Operation을 Custom하는 방법은 `main()`메서드만 재정의해주면 된다.

```swift
class MySyncOperation: Operation {
    let str: String
    let count: UInt
    
    init(str: String, count: UInt) {
        self.str = str
        self.count = count
    }
    
    override func main() {
        guard count > 1 else {
            print("count에는 1보다 큰 숫자를 입력해주세요")
            return
        }
        
        for _ in 0...count-1 {
            print(str)
        }
    }
}

let syncOperation1 = MySyncOperation(str: "야곰", count:3)
syncOperation1.start()
```

Operation의 실질적인 동작을 담당하는 `main()`만 재정의하면 Custom Operation을 만들 수 있다.
동기적으로 코드를 실행하는 Operation이기 때문에 `main` 내의 작업이 끝난 뒤에 상태가 finished가 된다. 
 
### Custom Asynchronous Operation

비동기 Operation을 Custom하는 경우에는 상태변화를 담당하는 `start()`함수를 재정의해야 한다. 이는 비동기가 다른스레드에 맡겨지고 메인스레드에서는 실행을 끝내는 특징때문에 생긴다. 이는 Operation이 `finished`상태로 변하는 시점이 부정확해지는 상황이 발생한다. 
즉 상태를 변경하는 코드를 전체적으로 재정의해주어야 한다.

먼저 `main`을 재정의했을 때 일어나는 코드를 살펴보자.
```swift
class StrangeAsyncOperation: Operation {
    let str: String
    let count: UInt
    
    init(str: String, count: UInt) {
        self.str = str
        self.count = count
    }
    
    override func main() {
        DispatchQueue.global().async { [str, count] in
            guard count > 1 else {
                print("count에는 1보다 큰 숫자를 입력해주세요.")
                return
            }

            for _ in 0 ... count - 1 {
                print(str)
            }            
        }
    }
}

let strangeOperation = StrangeAsyncOperation(str: "야곰", count: 5)
strangeOperation.completionBlock = {
    if strangeOperation.isFinished {
        print("Finished!!")
    }
}

/* 출력
Finished!
야곰
야곰
야곰
야곰
야곰
*/

```
`global.async()`에서 비동기로 작업을 처리하니 실제 내부 작업이 끝나기도 전에 Operation의 상태가 Finished로 변경된 것이다. 그렇기에 상태 변경에 대한 코드도 비동기 작업에 맞게 재정의되어야 한다.

```swift
class AsyncOperation: Operation {
    // 1
    enum State: String {
        case ready, executing, finished

        // 2
        var keyPath: String {
            "is\(rawValue.capitalized)"
        }
    }

    // 3
    var state = State.ready {
        // 4
        willSet {
            willChangeValue(forKey: state.keyPath)
            willChangeValue(forKey: newValue.keyPath)
        }
        didSet {
            didChangeValue(forKey: oldValue.keyPath)
            didChangeValue(forKey: state.keyPath)
        }
    }

    // 5
    override var isExecuting: Bool {
        state == .executing
    }
    override var isFinished: Bool {
        state == .finished
    }
    override var isAsynchronous: Bool {
        true
    }
    // 6
    override func start() {
        if isCancelled {
            state = .finished
            return
        }
        state = .executing
        main()
    }

    override func cancel() {
        state = .finished
    }
    
    let str: String
        let count: UInt

        init(str: String, count: UInt) {
            self.str = str
            self.count = count
        }

        override func main() {
            DispatchQueue.global().async { [str, count] in
                guard count > 1 else {
                    print("count에는 1보다 큰 숫자를 입력해주세요.")
                    return
                }

                for _ in 0 ... count - 1 {
                    sleep(1)
                    print(str)
                }

                // 7
                self.state = .finished
            }
        }
}



let strangeOperation = AsyncOperation(str: "야곰", count: 5)
strangeOperation.completionBlock = {
    if strangeOperation.isFinished {
        print("Finished!")
    }
}
strangeOperation.start()
strangeOperation.cancel()
```

KVO를 이용해 Operation 프로퍼티들을 업데이트하고, 커스텀한 State에 따라 상태를 조작하기 위해 `isExecuting`, `isFinshed`, `isAsynchronous`를 override한다. 

핵심은 Operation이 가지고있는 고유의 `isExecuting`, `isFinshed` 값들이 재정의해서 state라는 커스텀으로 정의한 상태값에 기반하여 변경된다는 것이다.

이 때문에 `start`메서드를 재정의하여 원하는대로 상태를 조작할 수 있는 것이다.


### Note
`start`메서드를 재정의할 때는 super를 호출하면 안된다. Operation의 상태를 변경하는 동작이 의도와 다를 수 있기 때문이다. 그리고 `start`메서드를 재정의하면서 `isCancelled`프로퍼티를 확인하는 작업도 놓치지 않아야 한다.


### 참고자료
- [escaping Closure](https://jeong9216.tistory.com/471)
- [kodeco - Operation](https://www.kodeco.com/5293-operation-and-operationqueue-tutorial-in-swift)
- [야곰닷넷 - 동시성프로그래밍](https://yagom.net/courses/%eb%8f%99%ec%8b%9c%ec%84%b1-%ed%94%84%eb%a1%9c%ea%b7%b8%eb%9e%98%eb%b0%8d-concurrency-programming/lessons/%eb%8f%99%ec%8b%9c%ec%84%b1-%ed%94%84%eb%a1%9c%ea%b7%b8%eb%9e%98%eb%b0%8d/)

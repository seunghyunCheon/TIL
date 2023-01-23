230123 비동기 메서드 유닛 테스트, URLSession 톺아보기, TDD
===
TIL (Today I Learned)
===
학습내용
---
- 비동기 메서드 테스트하기
- 테스트릴 위한 객체 만들기
- TDD 학습하기

## 고민한 점 / 해결 방법
## 비동기 메서드 테스트
> 일반적인 테스트는 비동기 메서드가 실행중인 스레드를 기다려주지 않고 실행을 종료한다. 비동기 메서드를 기다려주기 위한 방법으로 expectation(description:), fulfill(), wait(for:timeout:)이 존재한다.
- `expectation(description:): 어떤 것이 수행되어야 하는지를 description으로 정해준다.`
- `fulfill()`: 정의한 expectation이 충족되는 시점에 호출하여 동작을 수행했음을 알림
- `wait(for:timeout:)`: expectation을 배열로 담아 전달해 배열 속의 expectation이 모두 fulfill될 때까지 기다린다. timeout을 통해 시간제한을 걸 수 있다.
```swift
func test_makeRandomValue호출시_randomValue를_잘설정해주는지() {
    // given
    let promise = expectation(description: "It makes random value")
    sut.randomValue = 50

    // when
    sut.makeRandomValue {
        // then
        XCTAssertGreaterThanOrEqual(self.sut.randomValue, 0)
        XCTAssertLessThanOrEqual(self.sut.randomValue, 30)
        promise.fulfill()
    }

    wait(for: [promise], timeout: 10) // wait
}

func test_reset호출시_tryCount가0이되는지() {
    // given
    let promise = expectation(description: "It makes tryCount zero")
    sut.tryCount = 4
    // when
    sut.reset {
        // then
        XCTAssertEqual(self.sut.tryCount, 0)
        promise.fulfill()
    }

    wait(for: [promise], timeout: 10)
}
```

## 테스트를 위한 객체 만들기
> 네트워킹 환경이 구축된 상태에서만 테스트가 가능한 것은 First 원칙에서 Repeatable 에 위반한다.(모든 환경을 통제하여 매번 예상한 결과대로 테스트가 진행되어야 한다)
- 테스트를 위한 객체를 만들어 URLSession을 대신해 테스트에 참여하게 만드는 것

### 테스트 더블
> 테스트가 진행하기 어려운 경우 이를 대신하여 테스틀 진행할 수 있도록 만들어주는 객체
- 테스트 더블의 역할
    - 1. 테스트 대상 코드를 격리한다.
    - 2. 테스트 속도를 개선한다.
    - 3. 예측 불가능한 실행 요소를 제거한다.
    - 4. 특수한 상황을 시뮬레이션한다.
    - 5. 감춰진 정보를 얻어낸다.
- 테스트 더블의 종류
    - 1. Dummy(모조의, 가짜의): 가장 기본적인 테스트 더블로 어떤 기능도 구현되어있지않고 객체를 전달하기 위한 목적으로 주로 사용된다.
    - 2. Stub(쓰다 남은): Dummy가 실제로 동작하는 것처럼 만들어 실제 코드를 대신해서 동작해 주는 객체. 테스트가 곤란한 부분의 객체를 도려내어 그 역할을 최소한으로 대신해 줄 만큼만 구현이 되어있다.
    - 3. Fake: Stub보다 구체적으로 동작하지만 실제 앱의 동작에서는 적합하지 않은 객체. 로직은 실제와 비슷하지만 동작을 단순화해서 구현했기 때문에 Fake객체라고 불린다.
    - 4. Spy: Stub의 역할을 가지면서 호출된 내용에 대한 방법, 과정 등의 정보를 기록하는 객체.
    - 5. Mock: 실제 객체와 가장 비슷하게 구현된 객체. Stub은 상태 기반 테스트, Mock은 행동 기반 테스트. 상태 기반 테스트가 메서드를 호출하고 결과값과 예상 값을 비교하는 것이라면 행동 기반 테스트는 시나리오들을 만들어 놓고 시나리오대로 동작했는지에 대한 여부를 확인하는 것

### 의존성 주입
> 하나의 객체가 다른 객체의 의존성을 제공하는 기술 DI(Dependency Injection)
- 의존성이란?
    - 어떤 객체가 내부에서 생성하여 가지고 있는 객체를 의존성이라고 한다.
    ```swift
    // Car클래스는 wheel에 의존한다.
    class Car {
        var wheel: Wheel = Wheel()
    }

    class Wheel {
        var weight = 10
    }
    ```
- 의존성 주입이란?
    - 의존성을 주입시키는 것. 즉 내부에서 초기화가 이루어지는 것이 아닌 외부에서 객체를 생성하여 주입시키는 것
- 의존성 주입은 왜 사용하나요?
    - 객체간의 결합도를 낮추기 위해서. 객체 간 결합도가 낮아지면 리팩토링이 쉽고, 테스트 코드 작성이 쉬워진다. 
```swift
class UpDownGame {
    var randomValue: Int = 0
    var tryCount: Int = 0
    var urlSession: URLSessionProtocol

    init(urlSession: URLSessionProtocol = URLSession.shared) {
        self.urlSession = urlSession
    }
    ...
}
```
- 내부에서 urlSession을 초기화하는 것이 아닌 외부에서 주입된 것을 토대로 초기화가 됙 때문에 리팩토링이 쉬워진다. 현재는 urlSession이 외부에서 주입되지만 이를 테스트 더블로 바꿔 주입함으로써 결합도가 낮은 장점을 얻을 수 있다.

### 테스트를 위한 객체를 이용해서 테스트 작성하기
- 현재는 URLSession이 URLSessionDataTask를 만들어 네트워킹을 통해 얻고자 하는 값을 얻어와서 completionHandler를 실행해주고 있다. 
- 테스트 더블은 Networking을 통해 직접 데이터를 받아오는 부분이 아닌 직접 데이터를 만들어서 CompletionHandler로 전달하면 된다. 그리고 URLSession와 URLSessionDataTask대신 DummyData를 전달할 수 있는 Stub 객체를 만들어서 바꾸면 된다.
- 이 부분이 조금 어려워서 URLSession에 대해 정보를 더 찾아봤다.
```swift
// URLSession.swift
open class URLSession : NSObject {
    // property
    internal let workQueue: DispatchQueue 
    internal let taskRegistry = URLSession._TaskRegistry()
    ...
    
    // method
    open func dataTask(with url: URL, completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void) -> URLSessionDataTask {
        return dataTask(with: _Request(url), behaviour: .dataCompletionHandler(completionHandler))
    }

    ...

    func dataTask(with request: _Request, behaviour: _TaskRegistry._Behaviour) -> URLSessionDataTask {
        guard !self.invalidated else { fatalError("Session invalidated") }
        let r = createConfiguredRequest(from: request)
        let i = createNextTaskIdentifier()
        let task = URLSessionDataTask(session: self, request: r, taskIdentifier: i)
        workQueue.async {
            self.taskRegistry.add(task, behaviour: behaviour)
        }
        return task
    }
}

// URLSessionTask.swift
open func resume() {
    workQueue.sync {
        guard self.state != .canceling && self.state != .completed else { return }
        if self.suspendCount > 0 { self.suspendCount -= 1 }
        self.updateTaskState()
        if self.suspendCount == 0 {
            self.hasTriggeredResume = true
            self._getProtocol { (urlProtocol) in
                self.workQueue.async {
                    if let _protocol = urlProtocol {
                        _protocol.startLoading()
                    }
                    ...
            }
        }
    }
}
```
> 이해한 내용이 틀릴 수도 있습니다...😅
- 01 Main에서 Session객체를 만들어 dataTask안에 url과 후에 전달받을 completionHandler: (Data, Response, Error) handler를 인자로 넣는다.
- 02 호출된 dataTask안에서 Sesstion클래스 내의 두 번째인자에 behaviour를 넣는 또다른 dataTask를 호출한다.
- 03 호출된 두 번째 dataTask안에서 Session이 invalidate한 경우 탈출하고 전달받은 url을 받아 요청서를 만들고 identifier를 설정해 task를 만든다. 그리고 workQueue에 비동기적으로? taskRegistry에 해당 task를 completionHandler와 함께 추가한다. 이후 바로 만들었던 URLSessionDataTask를 리턴한다.
- 04 Main에서 리턴받은 URLSessionDataTask의 작업을 실행하기 위해 resume()메서드를 호출한다.
- 05 SessionTask가 취소되거나 완료된 것이라면 작업을 중단한다. suspendCount가 0이라면 running상태로 Resume을 trigger한 뒤 self._getProtocol을 통해 urlProtocol을 받아 urlProtocol이 nil이 아니라면 startLoading을 호출해 비동기 작업을 시작한다. 
- 5번 부분을 정확히 알고 싶어서 NSLock, URLprotocol 등 키워드를 찾아봤는데 코드만 보고 이해하기는 어려웠다. 추후 queue에 대한 이해와 urlProtocol에 대한 이해가 쌓이면 다시 봐야겠다 (5번 부분은 무시해주세요!)

#### 결론적으로
- URLSession과 URLSessionTask는 밀접하게 연관이 되어있고 이들은 workQueue에서 비동기적으로 add되고 실행된다는 사실이 중요하다. 
- 그리고 SessionTask에서 resume을 해줘야 큐에 있던 작업이 시작되기 때문에 꼭 resume을 해줘야 한다.

#### 다시 원점으로.. 테스트 더블
- 더미데이터에는 직접 정의한 data, response, error, completion이 존재한다.
```swift
struct DummyData {
    let data: Data?
    let response: URLResponse?
    let error: Error?
    var completionHandler: DataTaskCompletionHandler? = nil
    
    func completion() {
        completionHandler?(data, response, error)
    }
}
```
- StubURLSession에는 dummyData와 URLSessionDataTask를 리턴하는 dataTask함수가 존재한다.
```swift
class StubURLSession: URLSessionProtocol {
    var dummyData: DummyData?
    init(dummy: DummyData) {
        self.dummyData = dummy
    }

    func dataTask(with url: URL, completionHandler: @escaping DataTaskCompletionHandler) -> URLSessionDataTask {
        return StubURLSessionDataTask(dummy: dummyData, completionHandler: completionHandler)
    }
}
```
- StubURLSessionDataTask에는 외부에서 들어온 파라미터를 통해 dummyData와completionHandler를 설정한다. 그리고 resume이 실행되면 설정해놓은 dummyData에서 completion을 호출하도록 만든다.
```swift
class StubURLSessionDataTask: URLSessionDataTask {
    var dummyData: DummyData?
    
    init(dummy: DummyData?, completionHandler: DataTaskCompletionHandler?) {
        self.dummyData = dummy
        self.dummyData?.completionHandler = completionHandler
    }
    
    override func resume() {
        // data, response, error가 생성이 되고 전달받은 completion에 data, response, error를 인자로 넣어 실행
        dummyData?.completion()
    }
}
```
- 테스트 케이스 작성.
```swift
func test_randomValue로_3이라는값을_받을때() {
    // given
    let promise = expectation(description: "")
    let url = URL(string: "http://www.randomnumberapi.com/api/v1.0/random?min=1&max=30&count=1")!
    let data = "[3]".data(using: .utf8)
    let response = HTTPURLResponse(url: url, statusCode: 200, httpVersion: nil, headerFields: nil)
    let dummy = DummyData(data: data, response: response, error: nil)
    let stubURLSession = StubURLSession(dummy: dummy)

    sut.urlSession = stubURLSession // 의존성 주입

    // when
    sut.reset {
        // then
        XCTAssertEqual(self.sut.randomValue, 3)
        promise.fulfill()
    }
    wait(for: [promise], timeout: 10)
}
```

## TDD
> 테스트 주도 개발(Test-driven development TDD)의 약자로 매우 짧은 개발 사이클을 반복하는 소프트웨어 개발 프로세스 중 하나이다. 이는 먼저 실패하는 테스트 케이스를 작성하고(Red) 이를 해결하는 코드를 작성하고(Green) 작성한 코드를 더 나은 방향으로 개선해 나가는 작업을 하는 것이다.


<img src="https://i.imgur.com/qImrEnC.png" width="500"/>

- TDD는 테스트를 통과하는 코드를 작성하기 위해 재사용성과 의존성에 대해서 고민하여 보다 의존성이 낮은 코드를 작성할 수 있다. 이외에도 유지 보수가 용이하다는 장점도 있다.
- 하지만 많은 부분에 대해서 테스트 케이스를 작성하게 되면 개발 속도가 느려질 수 있다. 



### Reference
- [야곰닷넷 - Unit Test](https://yagom.net/courses/unit-test-%ec%9e%91%ec%84%b1%ed%95%98%ea%b8%b0/lessons/unit-test%ea%b0%80-%eb%ac%b4%ec%97%87%ec%9d%b8%ea%b0%80%ec%9a%94/)
- [Apple Developer Documentation - URL Loading System](https://developer.apple.com/documentation/foundation/url_loading_system)
- [Apple Developer Documentation - URLSessionConfiguration](https://developer.apple.com/documentation/foundation/urlsessionconfiguration)
- [Apple Developer Documentation - URLSession](https://developer.apple.com/documentation/foundation/urlsession)

###### tag: `Unit Test`, `URLSession`, `URLSessionDataTask`, `TDD`

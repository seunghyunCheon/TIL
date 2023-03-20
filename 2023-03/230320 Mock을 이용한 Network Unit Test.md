230320 Mock을 이용한 Network Unit Test
===
학습내용
---
- Mock을 이용한 Network Unit Test

## Mock을 이용한 Network Unit Test

테스트 더블은 테스트 코드에서 의존하는 객체를 대신해 사용하는 것이다. 실제객체를 테스트하기 불편한 상황에 모의객체를 만들어 의존성 주입을 해서 원하는 대로 객체의 동작을 제어할 수 잇다.

테스트 원칙 중에는 테스트 대상은 외부환경에 의존하면 안된다는 원칙이 존재한다.

이는 어느 상황이든지 테스트 코드는 일관되게 동작해야 한다는 말과도 같다. 외부환경으로 네트워크가 대표적인 예시인데, 이는 환경에 따라 통신이 실패할 수도 있고, 테스트를 보내는 요청이 post일 때 서버의 정보를 변환시킬 수 있기 때문에 여러 상황에서 적합하지 않을 수 있다.

만약 network를 담당하는 `NetworkManager`객체가 `viewModel` 내부에 존재하고`viewModel`을 테스트한다면 메서드들이 네트워크에 의존하기 때문에 일관적이지 않은 결과가 나오게 될 것이다. 
`Network`를 테스트하고 싶다면 `Network Test`를 작성하면 되는 것이다. 

중요한 건 `ViewModel` 테스트는 `ViewModel`에 집중하는 것이다. 이러한 경우 `NetWorkStub`이나 `NetWorkMock`을 만들어서 `resume` 시 바로 정해진 결과를 반환하도록 만들 수 있다.

**즉 값이 제대로 오는지를 테스트하는 것이 아니라 값이 온 상태에서 값의 유형에 따라 어떠한 흐름으로 처리할지를 테스트하는 것이다.**

보통 Stub은 상태기반테스트, Mock은 행동기반테스트라고 부른다. 
상태 기반 테스트는 메서드를 호출하고 결과값과 예상값을 비교하는 것, 행위 기반 테스트는 예상되는 행위들에 대한 시나리오를 만들고 시나리오대로 동작했는지에 대한 여부를 판단하는 것이다.

글로 봤을때와 예시들을 살펴봐도 잘 와닿지 않아서 Mock은 메서드 호출에 의미를 두고, Stub은 메서드 호출에 따른 반환 값에 초점을 둔 것이라고 이해했다.

#### Mock의 예시
```swift
protocol FileSaver {
  func save(file: File)
}

class MockFileSaver: FileSaver {
  var didCallSave = false
  var savedFile: File?

  func save(file: File) {
    didCallSave = true
    savedFile = file
  }
}
```

이후 내용은 naljin블로그를 참고했습니다.

### Mock 객체 만들기

네트워크에 의존하지 않는 답이 정해진 객체를 만들기 위해 `Mock`객체를 만들어보자. 

기존의 `URLSession`은 `URLSessionDataTask`를 만들어내고 `resume`메서드를 호출하면 `URLSessionDataTask`로부터 비동기 통신이 일어나고 완료가 되면 `completionHandler`로 이동이 된다.

Mock 객체를 만드는 이유는 비동기 통신이 일어나지 않고 정해진 값을 리턴하게 만들기 위해서 사용하는 것이다. 즉 `resume`메서드 호출 시에 바로 저장되어있던 값을 반환하게 되는 것이다.

그럼 어떤 것을 먼저 Mock으로 대체하면 좋을까?

모든일의 시초인 `URLSession`부터 `Mock`으로 대체해보자.

```swift
final class NetworkManager {
    let session: URLSessionProtocol
    
    init(session: URLSessionProtocol = URLSession.shared) {
        self.session = session
    }
}
```
원래는 `session`의 타입이 `URLSession`이었지만 테스트 환경에서도 사용할 수 있어야하기 때문에 `URLSessionProtocol`을 받도록 더 위로 추상화시켰다.

```swift
protocol URLSessionProtocol {
    func dataTask(with url: URL, completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void) -> URLSessionDataTask
}
```

그리고 `URLProtocol`을 따르는 것들을 이니셜라이저에 받고 있기 때문에 `URLSession`에서도 이를 채택해줘야 한다.

```swift
extension URLSession: URLSessionProtocol { }
```

`MockURLSession`도 클래스를 만들어주고 `URLProtocol`을 채택해준다.

```swift
class MockURLSession: URLSessionProtocol {
    func dataTask(with url: URL, completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void) -> URLSessionDataTask
}
```

그리고 `URLSession`의 `dataTask`메서드를 통해 만든 DataTask객체도 만들어준다.

```swift
protocol URLSessionDataTaskProtocol {
    func resume()
}

extension URLSessionDataTask: URLSessionDataTaskProtocol

class MockURLSessionDataTask: URLSessionDataTaskProtocol { 
    private let resumeHandler: () -> Void
    
    init(resumeHandler: @escaping () -> Void) {
        self.resumeHandler = resumeHandler
    }
    
    func resume() {
        resumeHandler()
    }
}
```

애플에서 제공하는 `URLSessionDataTask`는 `resume` 시 비동기 통신을 하지만 Mock객체에서는 바로 데이터를 전달해준다.

`MockURLSession`에는 다음과 같이 추가해준다.

```swift
class MockURLSession: URLSessionProtocol {
    typealias Response = (data: Data?, urlResponse: URLResponse?, error: Error?)
    
    let response: Response
    
    init(response: Response) {
        self.response = response
    }
    
    func dataTask(with url: URL,
                 completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void) -> URLSessionDataTaskProtocol {
        return MockURLSessionDataTask(resumeHandler: {
            completionHandler(self.response.data,
                             self.response.urlResponse,
                             self.response.error)
        })
    }
}
```

`MockURLSession`을 만들면서 정해진 응답을 이니셜라이저에 넣어주고, `dataTask`를 통해서 `URLSessionDataTask` 객체를 반환함과 동시에 `completionHandler`를 등록시켜준다.

그 후 `resume`만 실행해주면 등록된 `completionHandler`가 실행되는 구조이다.

테스트코드는 다음과 같이 작성할 수 있다.

먼저 `fetchData`의 함수는 다음과 같다.

```swift
func fetchData<T: Decodable>(for url: String,
                                 dataType: T.Type,
                                 completion: @escaping (Result<T, Error>) -> Void) {
        
    guard let url = URL(string: url) else {
        return
    }

    let dataTask: URLSessionDataTaskProtocol = session.dataTask(with: url, completionHandler: { (data, response, error) in
        if let error = error {
            completion(.failure(error))
            return
        }

        if let data = data,
           let response = response as? HTTPURLResponse,
           200..<400 ~= response.statusCode {
            do {
                let data = try JSONDecoder().decode(T.self, from: data)
                completion(.success(data))
            } catch {
                completion(.failure(NetworkError.failToParse))
            }
        } else {
            completion(.failure(NetworkError.invalid))
        }
    })
    dataTask.resume()
}
```
### 디코딩 후 원하는 객체에 값을 담아 반환하는지

```swift
func test_fetchData_Data가_있고_statusCode가_200일때() {
    // given
    let url = "https://api.sampleapis.com/coffee/hot"
    let mockResponse: MockURLSession.Response = {
        let data: Data? = JsonLoader.data(fileName: "Coffees")
        let successResponse = HTTPURLResponse(url: URL(string: url)!,
                                              statusCode: 200,
                                              httpVersion: nil,
                                              headerFields: nil)
        return (data: data,
                urlResponse: successResponse,
                error: nil)
    }()
    let mockURLSession = MockURLSession(response: mockResponse)
    let sut = NetworkManager(session: mockURLSession)
    
    // when
    var result: [Coffee]?
    sut.fetchData(for: url,
                  dataType: [Coffee].self) { response in
        if case let .success(coffees) = response {
            result = coffees
        }
    }
    
    // then
    let expectation: [Coffee]? = JsonLoader.load(type: [Coffee].self, fileName: "Coffees")
    XCTAssertEqual(result?.count, expectation?.count)
    XCTAssertEqual(result?.first?.title, expectation?.first?.title)
```

### 잘못된 dataType을 넘겼을 때 원하는 에러를 반환하는지
```swift
func test_fetchData_Data에_대한_잘못된_dataType을_넘겼을때() {
    // given
    let mockURLSession = MockURLSession.make(url: url,
                                             data: data,
                                             statusCode: 200)
    let sut = NetworkManager(session: mockURLSession)


    // when
    var result: NetworkError?
    sut.fetchData(for: url,
                  dataType: Coffee.self) { response in
        if case let .failure(error) = response {
            result = error as? NetworkError
        }
    }

    // then
    let expectation: NetworkError = NetworkError.failToParse
    XCTAssertEqual(result, expectation)
}
```

### statusCode가 500이면 에러를 반환하는지
```swift
func test_fetchData_Data가_없고_statusCode가_500일때() {
    // given
    let mockURLSession = MockURLSession.make(url: url,
                                             data: nil,
                                             statusCode: 500)
    let sut = NetworkManager(session: mockURLSession)

    // when
    var result: Error?
    sut.fetchData(for: url,
                  dataType: [Coffee].self) { response in
        if case let .failure(error) = response {
            result = error
        }
    }

    // then
    XCTAssertNotNil(result)
}
```

mockURLSession생성을 위한 중복코드를 줄이기 위한 방법으로 다음의 메서드를 정의할 수도 있다.
```swift

static func make(url: String, data: Data?, statusCode: Int) -> MockURLSession {
    let mockURLSession: MockURLSession = {
        let urlResponse = HTTPURLResponse(url: URL(string: url)!,
                                          statusCode: statusCode,
                                          httpVersion: nil,
                                          headerFields: nil)
        let mockResponse: MockURLSession.Response = (data: data,
                                                     urlResponse: urlResponse,
                                                     error: nil)
        let mockUrlSession = MockURLSession(response: mockResponse)
        return mockUrlSession
    }()
    return mockURLSession
}
```
- [naljin - netWork-unitTest](https://sujinnaljin.medium.com/swift-mock-%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-network-unit-test-%ED%95%98%EA%B8%B0-a69570defb41)




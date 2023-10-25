# 231025 Combine을 이용한 URLSession 데이터 처리.

> 비동기 operator를 이용하여 URL로부터 패치된 데이터를 받고 처리함.


## 개요
URL세션 작업수행은 본질적으로 비동기다. 네트워크 엔드포인트, 파일 시스템, 다른 URL 리소스로부터 데이터 패치에 시간이 오래걸린다. URL 로딩시스템은 이를 델리게이트나 컴플리션 핸들러에 비동기적으로 전달한다.
Combine 프레임워크를 이용하면 이를 단순하게 만든다.

## data task Publisher
`URLSession`은 `URLSession.DataTaskPublisher`를 제공한다. 이는 `URL`이나 `URLRequest`로부터 패치된 데이터결과를 발행한다. `dataTaskPublisher(for:)`를 통해 만들 수 있다. 결과값으로는 URLResponse, data, error가 발행된다. 클로저를 이용한 데이터 처리를 컴바인에 모두 위임해보자.

## 컴바인을 통해 raw data를 원하는 타입으로
data task Publisher는 튜플타입의 Data와 URLResponse를 생산한다. `map` 오퍼레이터를 사용하여 이 튜플을 다른 타입으로 변환할 수도 있다. 만약 데이터를 분류하기전에 응답을 검사하고 싶다면 `tryMap`을 사용할 수도 있다.

```swift
struct User: Codable {
    let name: String
    let userID: String
}
let url = URL(string: "https://example.com/endpoint")!

cancellable = urlSession
    .dataTaskPublisher(for: url)
    .tryMap() {
        guard let httpResponse = element.response as? HTTPURLResponse,
            httpResponse.statusCode == 200 else {
                throw URLError(.badServerResponse)
            }
        
        return element.data
    }
    .decode(type: User.self, decoder: JSONDecoder())
    .sink(receiveCompletion: { print ("Received completion: \($0).") },
          receiveValue: { user in print ("Received user: \(user).")})
```

## 일시적인 오류를 다시 시도하고 지속적인 오류를 찾아 교체한다.
네트워크를 사용하는 모든 앱에서는 오류를 예상하고 정상적으로 처리해야 한다. 일시적인 네트워크 에러는 매우 일반적이므로 즉시 다시 시도하는게 좋을 수 있다. dataTaskPublisher를 이용해서 `retry` 오퍼레이터를 사용할 수 있다. 이는 업스트림 게시자에 대한 구독을 지정된 횟수만큼 다시 생성하여 오류를 처리한다. 하지만 네트워크 작업은 비용이 비싸기 떄문에 작은 시도를 하는 것을 명심해야 한다.

`catch`로 오면 현재 퍼블리셔를 다른 퍼블리셔로 변경한다.

```swift
let pub = urlSession
    .dataTaskPublisher(for: url)
    .retry(1)
    .catch() { _ in
        self.fallbackUrlSession.dataTaskPublisher(for: fallbackURL)
    }
cancellable = pub
    .sink(receiveCompletion: { print("Received completion: \($0).") },
          receiveValue: { print("Received data: \($0.data).") })
```

## 스레드 변경
`receive(on:options:)`를 이용하여 스레드를 변경할 수 있다.
```swift
cancellable = urlSession
    .dataTaskPublisher(for: url)
    .receive(on: DispatchQueue.main)
    .sink(receiveCompletion: { print ("Received completion: \($0).") },
          receiveValue: { print ("Received data: \($0.data).")})
```

## share를 이용하여 다양한 구독자들에게 결과 공유
네트워킹 작업을 한번하고 난 후 여러 구독자들에게 값을 공유하고 싶을 수 있다.

```swift
let sharedPublisher = urlSession
    .dataTaskPublisher(for: url)
    .share()


cancellable1 = sharedPublisher
    .tryMap() {
        guard $0.data.count > 0 else { throw URLError(.zeroByteResource) }
        return $0.data
    }
    .decode(type: User.self, decoder: JSONDecoder())
    .receive(on: DispatchQueue.main)
    .sink(receiveCompletion: { print ("Received completion 1: \($0).") },
          receiveValue: { print ("Received id: \($0.userID).")})


cancellable2 = sharedPublisher
    .map() {
        $0.response
    }
    .sink(receiveCompletion: { print ("Received completion 2: \($0).") },
           receiveValue: { response in
            if let httpResponse = response as? HTTPURLResponse {
                print ("Received HTTP status: \(httpResponse.statusCode).")
            } else {
                print ("Response was not an HTTPURLResponse.")
            }
    }
)
```



- [Processing URL session data task results with Combine](https://developer.apple.com/documentation/foundation/urlsession/processing_url_session_data_task_results_with_combine)

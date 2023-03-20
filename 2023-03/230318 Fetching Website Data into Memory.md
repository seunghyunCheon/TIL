230318 Fetching Website Data into Memory
===
학습내용
---
- Fetching Website Data into Memory

## Fetching Website Data into Memory
`URL session`으로부터 `data task`를 생성해 데이터를 메모리로 직접 수신한다.

### 개요
원격 서버와의 상호소통을 위해 `URLSessionDataTask`클래스를 사용할 수 있다. 이는 응답데이터를 메모리로 받게 해주며 데이터를 파일시스템에 직접 저장하는 `URLSessionDownloadTask` 클래스와는 다르다.

작업을 생성하는데 URL session 인스턴스를 사용한다. 만약 요구하는 것이 매우 단순하다면 `URLSession` 클래스의 `shared`프로퍼티를 사용한다. 
만약 `delegate callback`을 이용해 상호작용을 하려는 경우에는 `shared` 인스턴스 대신에 session을 만들어야할 것이다.

`URLSession`을 만들 때 `URLSessionConfiguration` 인스턴스를 사용하고 `URLSessionDelegate`나 그것의 서브 프로토콜 중 하나를 구현한 클래스도 전달한다. 세션들은 프로퍼티로 저장되어 원하는 속성을 설정할 수 있기에 다중 작업을 만드는데 재사용될 수 있다. 

### Note
필요한 여러개의 세션을 만드는 것을 주의한다. 만약 앱 내에서 비슷하게 설정된 세션이 존재한다면 하나의 세션을 만들어 공유한다.

일단 `Session`을 가지면 `dataTask()`메서드들 중 하나를 사용해 `data task`를 만든다. 이 테스트는 중단된 상태에서 만들어지며 `resume`을 호출해 시작될 수 있다.

### Completion Handler를 통해 결과를 받는다.
데이터를 패치하는 가장 간단한 방법은 completion Handler를 사용하는 data task를 만드는 것이다.
completion Handler에는 서버응답, 에러, 데이터 총 3가지 인자가 존재한다.

작업으로부터 결과를 받는 completion Hanlder를 만드는 그림이다.
<img src="https://i.imgur.com/AEh7Vm5.png" width="700"/><br/>

completion handler를 사용하는 data task를 만들기 위해서 `URLSession` 메서드인 `dataTask(with:)`를 호출한다. completion handler는 다음과 같은 세가지의 역할을 한다.

1. `error` 파라미터가 nil인지 검증한다. 만약 아니라면 전송 에러가 발생하기에 error를 다뤄서 함수를 탈출한다.
2. `response` 파라미터를 확인한다. 상태 코드는 성공인지 MIME type이 예상된 값인지를 검증한다. 만약 일치하지않으면 server error로 다루고 함수를 탈출한다.
3. 필요에 따라 `data` 인스턴스를 사용한다.

다음의 코드는 URL의 내용을 패치하는 `startLoad()` 메서드이다. 이는 `URLSession` 클래스의 shared 인스턴스를 사용함으로써 시작되고 그것의 결과를 completion handler에 전달하는 data task를 만든다.  
local, server error를 확인한 후에 이 handler는 데이터를 String으로 전환하고, `WKWebView` outlet을 구성하는데 사용된다. 

```swift
func startLoad() {
    let url = URL(string: "https://www.example.com/")!
    let task = URLSession.shared.dataTask(with: url) { data, response, error in
        if let error = error {
            self.handleClientError(error)
            return
        }
        guard let httpResponse = response as? HTTPURLResponse,
            (200...299).contains(httpResponse.statusCode) else {
            self.handleServerError(response)
            return
        }
        if let mimeType = httpResponse.mimeType, mimeType == "text/html",
            let data = data,
            let string = String(data: data, encoding: .utf8) {
            DispatchQueue.main.async {
                self.webView.loadHTMLString(string, baseURL: url)
            }
        }
    }
    task.resume()
}
```

### 중요
completion handler는 작업을 생성하는 것과는 다른 GCD에서 호출된다. 그러므로 UI를 업데이트하는 것은 main queue에 위치해야 한다.

### delegate를 이용해 전송 디테일과 결과를 받는다.
data task를 만들 때 진행되는 작업에 대한 더 높은 수준의 접근을 위해 completion handler에 제공하기보다는 session에 delegate를 설정할 수 있다. 

다음 그림은 작업으로부터 결과를 받는 델리게이트를 구현한 그림이다.

<img src="https://i.imgur.com/kpQ0mOg.png" width="700"/><br/>

이 접근 방식을 사용하면 전송이 완료되거나 오류로 실패할때까지 데이터의 일부가 `URLSessionDataDelegate`의 `URLSession(_:dataTask:didReceive:)`메서드에 도착해 제공된다.
이 델리게이트는 전송이 진행됨에 따라 다른 종류의 이벤트도 수신한다.

델리게이트 접근법을 사용할 때 `URLSession` 클래스의 shared 인스턴스를 사용하기보다는 `URLSession` 인스턴스를 생성할 필요가 있다.

새로운 세션을 생성하는 것은 소유한 클래스를 세션 델리게이트로써 설정할 수 있게 허용한다.

클래스가 하나이상의 델리게이트 프로토콜을 구현하도록 선언한다.(URLSessionDelegate, URLSessionTaskDelegate, URLSessionDataDelegate, URLSessionDonwloadDelegate). 그런다음 URL session 인스턴스를 `init(configuration:delegate:delegateQueue:)`를 이용해 만든다.
이 이니셜라이저를 이용해 configuration instance를 커스터마이즈 할 수 있다. 예를 들어 만약 적합한 connection이 일어날 때까지 기다려야 한다면 연결사항이 이용가능하지않을때 바로 실패를 하는 것보다는 `waitsForConnectivity`를 true로 만들어 기다리는 것이 좋은 방법일 것이다.

```swift
private lazy var session: URLSession = {
    let configuration = URLSessionConfiguration.default
    configuration.waitsForConnectivity = true
    return URLSession(configuration: configuration, delegate: self, delegateQueue: nil)
}
```

다음의 코드에서 `startLoad()`메서드는 data task를 시작하는데 이 session을 사용하고 데이터와 에러를 받는데에는 델리게이트 콜백을 사용한다. 
- `urlSession(_:dataTask:didReceive:completionHandler:)`는 응답이 성공적인 HTTP 상태코드를 받는지, MIME type이 `text/html` 또는 `text/plain`인지 검증한다. 만약 이것들중 하나라도 케이스에 해당하지 않는다면 취소되고 케이스에 모두 해당하면 진행한다.
- `urlSession(_:dataTask:didReceive:)`는 task에 의해 수신된 데이터 인스턴스를 `receviedData`라고 불리는 메모리에 추가한다.
- `urlSession(_:task:didCompleteWithError)`는 먼저 전송레벨이 발생했는지를 본다. 만약 에러가 없다면 `receievedData` 버퍼를 문자열로 변환하고 `webView`의 내용으로 설정한다.

```swift

private lazy var session: URLSession = {
    let configuration = URLSessionConfiguration.default
    configuration.waitsForConnectivity = true
    return URLSession(configuration: configuration, delegate: self, delegateQueue: nil)
}

var receivedData: Data?

func startLoad() {
    loadButton.isEnabled = false
    let url = URL(string: "https://www.example.com/")!
    receivedData = Data()
    let task = session.dataTask(with: url)
    task.resume()
}

func urlSession(_ session: URLSession, dataTaask: URLSessionDataTask, didRecevie response: URLResponse, completionHandler: @escaping (URLSession.ResponseDisposition) -> Void) {
    guard let response = response as? HTTPURLResponse, 
    (200...299).contains(response.statusCode),
    let mimeType = response.mimeType,
    mimeType == "text/html" else {
        completionHandle(.cancel)
        return
    }
    completionHandler(.allow)
}

func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive data: Data) {
    self.receivedData?.append(data)
}

func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?) {
    DispatchQueue.main.async {
        self.loadButton.isEnabled = true
        if let error = error {
            handleClientError(error)
        } else if let receivedData = self.receivedData,
        let string = String(data: receivedData, encoding: .utf8) {
            self.webView.loadHTMLString(string, baseURL: task.currentRequest?.url)
        }
    }
}
```
다양한 델리게이트 프로토콜들은 위에 보여지는 코드를 넘어 인증, 리다이렉트, 다른 특별한 경우들을 다룬다. 

- [Apple Docs - Fetching Website Data into Memory](https://developer.apple.com/documentation/foundation/url_loading_system/fetching_website_data_into_memory#2926952)




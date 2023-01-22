230122 URL Loading System API, Endpoint, URLSession
===
TIL (Today I Learned)
===
학습내용
---
- URLSession 학습하기

## 고민한 점 / 해결 방법
## URL Loading System
> 표준 인터넷 프로토콜을 사용해 URL과 상호 작용하고 서버와 통신하는 시스템
- URL 로딩 시스템은 https 또는 custom 프로토콜과 같은 표준 프로토콜을 사용한 URL에 의해 식별된 자원에 접근할 수 있게 만든다.
- 로드는 비동기적으로 수행되므로 앱이 응답하는 것을 유지하고 데이터나 에러와 같은 것들이 도착했을 때 처리할 수 있다.
- 원격 위치에 있는 데이터나 파일들을 다운로드하고 업로드하는 역할을 하는 많은 URLSessionTask 인스턴스를 만들기 위해 URLSession을 사용할 수 있다. 그리고 캐시, 쿠키, 셀룰러 커넥션 허용여부를 결정짓기 위해 URLSessionConfiguration 객체를 사용할 수 있다.
- URLSessionConfiguration에는 네 가지 속성이 있다(Session의 타입)
    - 1. singleton 공유 세션: 기본적인 요청에 사용되며 세션을 생성한 이후로 커스텀할 수 없다. 만약 제한적인 요구사항을 가진다면 이를 사용하는 게 좋다.
    - 2. Default 세션: 공유 세션처럼 생성하지만 세션 생성이후 커스텀할 수 있다는 특징이 있다.
    - 3. Ephemeral 세션: default 세션과 유사하지만 캐시, 쿠키, 디스크에 자격증명을 사용할 수 없다. 
    - 4. Background 세션: 앱이 실행중이지 않을 때 컨텐츠를 업로드하거나 다운로드할 때 사용한다. 
## API, Endpoint
> API는 두 시스템이 상호작용할 수 있게 하는 약속의 총 집합이고 Endpoint는 API가 서버에서 리소스에 접근할 수 있도록 가능하게 하는 URL이다.
## URLSession
> 네트워크 데이터전달과 관련된 그룹들을 관리하는 객체
```swift
class URLSession: NSObject
```
- URLSession은 URL에 의해 정의된 endpoint에서 데이터를 다운로드하고 업로드하는 API를 제공한다. 또한 앱이 실행되고 있지 않거나 앱이 일시중단 된 동안에 이 API를 이용해 백그라운드 다운로드를 수행할 수 있다.
- URLSessionDelegate, URLSessionTaskDelegate를 이용해서 인증하거나 리다이렉션 및 task 완료이벤트를 받을 수 있다.
- 앱은 여러개의 URLSession 인스턴스들을 만들고 있다. 이들은 web browser를 띄워준다거나, 대화형 요소를 사용한다거나, 백그라운드 다운로드를 할 때 만들어지고 사용된다.

### URL Session Type
- 위 URLSessionConfiguration과 동일

### URL Session Task Type
- Session안에서 데이터를 업로드하거나 원격의 파일, 디스크, NSData객체들로부터 데이터를 전달받을 수 있다. URLSession API는 네가지 타입의 task를 제공한다.
    - 1. NSData 객체를 이용해 데이터를 보내거나 받는다. Data task는 서버에 대한 짧은 요청을 위한 것이다.
    - 2. data를 업로드한다. 앱이 작동중이지 않아도 background에서 업로드할 수 있도록 지원한다.
    - 3. 다운로드 작업은 파일형태로부터 데이터를 받는다. 업로드처럼 background에서 다운로드할 수 있도록 지원한다.
    - 4. 웹소켓 작업은 TCP 및 TLS를 통해 메세지를 교환한다.

### Session Delegate 사용
> 세션작업은 공통의 델리게이트 객체를 공유한다. 이는 다양한 이벤트가 발생했을 때 정보를 제공하거나 얻기 위해 구현할 수 있다.

- when
    - 인증이 실패했을 때
    - 데이터가 서버로부터 도착했을 때
    - 데이터 캐싱이 가능할 때
- 만약 델리게이트에의해 제공되는 기능이 필요없다면 Session을 만들 때 nil을 넘기면 된다.
> session 객체는 델리게이트를 강한참조하고 있기 때문에 session이 invalidate되거나 앱이 종료되기 전까지 메모리 누수가 발생할 수 있다.

### 비동기와 URL Session
> 대부분의 네트워킹 API들과 마찬가지로 URL Session API는 매우 비동기적이다. 이들은 세 가지 방법중 하나의 형태로 데이터를 반환받는다.
- 1. swift를 사용한다면 메서드에 async 키워드를 붙여 작업을 수행할 수 있다. 예시로 `data(from:delegate:)`는 데이터를 패치하고 `download(from:delegate:)`은 파일을 다운로드한다. 이러한 메서드들을 호출하는 시점에 await을 붙여주어 전송이 완료될때까지 흐름을 중지할 수 있다. 그리고 `byts(from:delegate:)`메서드는 AsyncSequence로써 데이터를 받는다. 이 접근법은 for-await-in 구문을 사용해 for문을 순회하는 동안 각 원소의 사용을 기다리게 할 수 있다.
- 2. Swift나 Objective-C에서 데이터 전송이 끝났을 때 completion handler 블록을 제공할 수 있다.
- 3. Swift나 Objective-C에서 데이터가 전송중이거나 그것이 끝난 직후에 델리게이트 메서드를 콜백으로 받을 수 있다. 즉 2번의 경우보다 좀 더 디테일하게 이벤트를 처리할 수 있다.


### Reference
- [Apple Developer Documentation - URL Loading System](https://developer.apple.com/documentation/foundation/url_loading_system)
- [Apple Developer Documentation - URLSessionConfiguration](https://developer.apple.com/documentation/foundation/urlsessionconfiguration)
- [Apple Developer Documentation - URLSession](https://developer.apple.com/documentation/foundation/urlsession)

###### tag: `Navigation in Modal`, `싱글톤`, `static let`, `ARC`, `클로저`, `weak`, `unowned`

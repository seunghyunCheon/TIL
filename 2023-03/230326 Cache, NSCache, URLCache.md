230326 Cache, NSCache, URLCache
===
학습내용
- Cache, NSCache, URLCache

## Cache, NSCache, URLCache

### ✏️ Cache
캐시는 컴퓨터 과학에서 데이터나 값을 미리 복사해 놓는 임시장소를 가리킨다. 캐시는 캐시의 접근 시간에 비해 원래 데이터를 접근하는 시간이 오래 걸리는 경우나 값을 다시 계산하는 시간을 절약하고 싶은 경우에 사용한다. 

캐시의 대표적인 예로 CPU 캐시가 있는데, CPU의 수가 증가하면서 메모리 접근속도가 증가하는 것에 비해 CPU의 처리 속도가 훨씬 빠르게 증가하면서 CPU캐시의 중요성이 커졌다.

<img src="https://i.imgur.com/etb2gn1.png" width="300"/><br/>

앱 내에서는 이미지나 썸네일 등의 반복적으로 동일한 결과를 돌려주는 경우에 사용한다.
결국 Cache는 반복적으로 데이터를 불러오는 경우에 지속적으로 DBMS 혹은 서버에 요청하는 것이 아니라 Memory에 저장했다가 불러쓰는 것을 의미한다.

그리고 업데이트가 자주 발생하지 않는 데이터거나 입력값과 출력값이 일정한 데이터에 적용하기 좋다. 

원하는 데이터가 캐시에 존재하는 경우 `Cache Hit`라고 불리고 존재하지 않을때는 서버에 요청하게 되는데 이를 `Cache miss`라고 부른다.

그리고 캐시는 어디에 저장하는가에 따라서 특징과 효율이 다르다.
Local Cache와 Global Cache로 구분한다.

### Local Cache
- 서버마다 캐시를 따로 저장한다
- 다른 서버의 캐시를 참조하기 어렵다
- 서버 내에서 작동하기 때문에 속도가 빠르다
- 로컬 서버 장비의 Resource를 사용한다(Memory, Disk)

### Global Cache
- 여러 서버에서 캐시 서버에 접근해서 참조 할 수 있다
- 별도의 캐시 서버를 이용하기에 서버 간 데이터 공유가 쉽다
- 네트워크 트래픽을 사용해야 해서 로컬 캐시보다는 느리다
- 데이터를 분산하여 저장할 수 있다.

### ✏️ NSCache
리소스가 부족할때 제거될 수 있는 key-value 쌍을 임시로 저장하는 변경가능한 컬렉션

```swift
class NSCache<KeyType, ObjectType> : NSObject where KeyType: AnyObject, ObjectType: AnyObject
```

캐시 객체는 몇가지 면에서 다른 변경가능한 컬렉션과 다르다.
- `NSCache` 다양한 자동제거 정책을 통합한다. 이는 캐시가 시스템의 메모리를 너무많이 사용하지 않는다는 것을 보장한다. 만약 메모리가 다른 애플리케이션에 의해 필요해진다면 이 정책은 캐시로부터 몇개의 item을 제거한다. 즉 메모리 사용량을 최소화하는 것이다.
- 캐시를 잠그는 것 없이 다른 스레드로부터의 캐시에 item을 추가하거나 제거하거나 조회할 수 있다.
- `NSMutableDictionary`객체와는 다르게 cache는 캐시에 넣은 키 객체를 복사하지 않는다. 

일반적으로 `NSCache`객체를 사용해서 일시적으로 만드는데 비용이 많이드는 객체를 저장한다. 이 객체를 재사용하는 것은 재계산하지 않기 때문에 퍼포먼스측면에서 효율을 높인다. 하지만 하지만 그 객체는 앱 내에서 중요하지않고 만약 메모리가 찬다면 제거될 수 있다. 
만약 제거되면 그 값들은 필요할 때 다시 재계산되어야 할 것이다.

사용되지 않을때 버려질 수 있는 subcomponent를 가지는 객체들은 `NSDiscardableContent` 프로토콜을 채택해서 캐시 방출을 개선할 수 있다.
기본적으로 캐시 내부에 `NSDiscardableContent` 객체는 자동제거정책이 변경되었을 지라도 내용이 버려진다면 자동적으로 제거가 된다.
만약 `NSDiscardableContent` 객체가 cache로 들어간다면 캐시는 이 객체를 제거할 때`discardContentIfPossible()`을 호출한다. 

### ✏️ NSURLCache
URL객체를 캐시된 응답 객체와 mapping하는 객체

```swift
@interface NSURLCache: NSObject
```

`NSURLCache` 클래스는 `NSURLRequest` 객체를 `NSCachedURLResponse`객체와 매핑함으로써 URL load 요청의 응답을 캐시하는 것을 구현한다.
그리고 이 클래스는 in-memory와 디스크 캐시의 합성물을 제공하고 이 두개에 대한 사이즈를 조작할 수 있도록 만든다. 그리고 캐시 데이터가 영구적으로 저장되는 경로도 제어할 수 있다.

### Note
iOS에서는 시스템 디스크 공간이 부족할 때 on-disk 캐시가 제거될 수 있지만 앱이 동작하지 않을때만 가능하다


### Thread Safety
iOS8이후로 `NSURLCache`는 thread safe하다

비록 `NSURLCache` 인스턴스 메서드들이 다양한 실행 흐름에서 동시에 안전하게 호출하더라도 `cachedResponseForRequest`와 `storeCachedResponse:forRequest:` 메서드들은 같은 요청에 대해 응답을 읽거나 쓸때 race condition이 일어날 수 있음을 인지해야 한다.

`NSURLCache`를 서브클래싱했다면 thread safe하게 메서드를 재정의 해야 한다.

### Subclassing Notes
`NSURLCache` 클래스를 있는 그대로 사용할 수 있지만 특정한 요구가 있을때 subclass할 수 있다. 예를 들어 어떤 응답들이 캐시되어있는지 보고 싶을때나 보안이나 다른 이유로 저장 메커니즘을 재구현하고 싶을때 사용할 수 있다.

이 클래스의 메서드들을 재정의할 때 `task` 파라미터를 사용하는 메서드가 그렇지 않은 메서드보다 선호된다. 그러므로 task기반의 메서드를 override 해야 한다.
- 캐시에 응답을 저장. request기반의 `storeCachedResponse:forRequest:`를 override하기 보다는 `storeCachedResponse:forDataTask`를 override한다
- 캐시로부터 응답을 얻을때 - `cachedResponseForRequest`보다는 `getCachedResponseForDataTask:conmpletionHandler`:를 override
- 캐시응답을 지울때 - `removeCachedResponseForRequest:` 대신 `removeCachedResponseForDataTask`를 override




### 참고자료
- [위키백과 - 캐시](https://ko.wikipedia.org/wiki/%EC%BA%90%EC%8B%9C)
- [Cache](https://green1229.tistory.com/57)
- [Cache란?](https://goldfishhead.tistory.com/29)
- [Apple Docs - NSURLCache](https://developer.apple.com/documentation/foundation/nsurlcache#2940723)

230410 Cache-Control과 Etag, Caching best practice
===
학습내용
- Cache-Control과 Etag
- URLSession의 캐시정책

## Cache-Control과 Etag

### 캐시를 사용하는 이유

캐시를 사용하는 이유는 네트워크를 거치는 시간을 줄이고 사용자 데이터 사용을 아낄 수 있으며 네트워크 통신에 의한 베터리 소모를 줄일 수 있기 때문에 사용한다.

캐시 서버를 이용하는 경우도 있고 Client내에 캐시매니저가 존재한다면 이를 사용하는 방법도 존재한다. iOS의 경우에는 이러한 Cache Manager를 제공한다. 

### 캐시 동작 순서
URL load Request의 기본 구현이며 HTTP/HTTPS 프로토콜의 경우 다음의 순서를 따른다.

<img src="https://i.imgur.com/DPnRZis.png" width="500"/><br/>

1. 캐시된 문서가 있는지 조회한다
2. 매번 문서의 유효성을 검사해야 한다면 HEAD request를 발행한다. 그렇지 않으면 만료기간을 검증하는데 만료가 되었다면 HEAD request를 발행하고 아니라면 캐시된 응답을 반환한다.
3. 발행한 HEAD request에 대해 변경사항이 있다면 다시 리소스를 불러오고 아니라면 캐시된 자원을 반환한다. 

`URLSession`에 `URLSessionConfiguration`을 설정해줄 때 `configuration`의 `CachePolicy`가 `useProtocolCachePolicy`인 경우에는 서버에서 내려주는 응답헤더의 캐시정책을 따른다. (`URLSession` 내에는 `URLCache`가 기본적으로 내장되어있고 정책은 `useProtocolCachePolicy`를 따른다)

하지만 만약 `URLSession`의 cache정책에 `returnCacheDataElseLoad`를 설정한다면 응답헤더의 캐시정책에 관계없이 캐시된 데이터가 존재하면 반환하고 없다면 서버에 자원을 요청하게 된다. 

서버에서 내려주는 캐시정책이 있다면 클라이언트에서 맞춰서 사용하는 것이 좋다. (보안상으로 위험할 수 있다고 한다)

이제 캐시 동작 순서의 매번 재검증을 하는지에 대한 부분과 만료기간이 되었는지를 확인하는 부분에 대해서 `Cache-Control`과 `Etag`를 살펴보면서 알아보자.

### Cache-Control

캐시가 주는 이점과 순서를 알아봤다. 
하지만 캐시를 사용할 때는 데이터의 불일치를 최소화시켜야 하는 주의가 필요하다. 

HTTP는 Cache된 데이터와 Server가 제공하는 데이터 간의 불일치를 최소화하기 위한 방법으로 Cache-Control과 Etag를 사용하도록 요구한다. 

먼저 Cache-Control은 응답으로 받은 문서의 유효시간이 지나기 전까지 캐시된 데이터를 재사용하는 방식이다. 

즉 유효시간을 설정하는 건데 이 뿐만 아니라 아무것도 캐싱하지 않는 옵션과 캐시를 쓰기전에 항상 서버에 이 캐시를 써도 되는지를 여쭤보는 옵션 등이 있다. 
- no-Store: 캐싱을 금지한다
- no-cache: 캐시 관리자에게 캐싱을 허용하나, 사용 전 필수로 Server에 유효성 체크를 해야 한다.
- max-age=3600: 초단위로 3600은 1시간의 유효기간을 제공한다.
- must-revalidate: 만료된 캐시만 서버에 확인을 받도록 한다. 

no-cache와 must-revalidate가 헷갈렸는데 no-cache는 최신의 데이터를 유지하기 위해 서버에 유효성을 체크하는 것이고 must-revalidate는 만료된 캐시에 대해서만 서버에 자원을 요청하는 것이다.

위 내용을 기반으로 캐싱에 대한 best-practice를 다음과 정의할 수 있다고 한다.

### 변하지않는 내용에는 long max-age를
>`Cache-Control: max-age:31536000`

내용이 변하지 않는다면 긴 기간동안 내용을 가져도 될 것이다. `max-age`만 설정하는 경우 만료된 데이터에 대해서 서버에 데이터가 변화되었는지를 확인하는 작업이 존재하지 않는다. 하지만 상관없다. 내용이 변하지 않는데 그럴 필요 있나?할 때 쓴다.

### 변하는 내용에는 항상 server-revalidate를
>`Cache-Control`: no-cache

내용이 자주 변할 수 있다면 서버에 주기적으로 확인한다. 여기서 확인은 서버에 데이터가 변화되었는지를 확인하는 작업이다. `no-cache`는 위에서 알아봤듯이 캐싱을 허용하지만 서버에 확인은 하는 작업이다. 즉 간단한 확인작업만 통과한다면 캐싱된 자원을 반환한다.

### 변하는 내용에서 max-age는 종종 잘못된 선택을
보여지는 데이터와 실제 데이터베이스의 데이터가 달라질 수 있다. 
만약 page에서 html과, js, css를 요청했다고 가정해보자. 
그렇다면 서버는 max-age=10으로 이 자원들을 캐시에 저장할 것이다. 

그리고 6분 후에 page에서 다시 위의 자원들을 요청할 때 캐시는 style.css가 분실되어 서버에 style.css만 최신으로 가져온 후 html과 js는 기존에 저장되어있던 것을 반환한다. 

그럼 페이지는 6분 전에 응답받은 html과 js파일과 방금 받은 css파일이 존재한다. 이로 인해 보여지는 페이지의 상당부분이 깨질 수 있게 된다. 이 때문에 변할 수 있는 데이터에 max-age만 캐시정책으로 두는 것은 주의가 필요하다.

### Etag
> HTTP 컨텐츠가 바뀌었는지를 검사할 수 있는 태그. 같은 주소의 자원이라도 컨텐츠가 달라졌다면 Etag가 다르다.

유효시간으로 유효성을 체크하는 것이 충분하지 않은 경우가 있다.
- 문서가 주기적으로 업데이트되지만 바뀐 내용이 없는 경우
- 변경된 내용이 있지만, 응답상 의미가 크지 않은 경우
- 유효한 시간을 정하기 어려운 경우

이와 같은 상황에 Etag를 사용할 수 있다.

예를 들어 GET www.brody.com의 응답 본문이 hi brody이고 ETag 헤더 값이 20232222라고하자. 만약 서버 컨텐츠가 동일하다면 매번 GET www.brody.com을 하게된다면 ETag가 20232222로 같을 것이다. 이처럼 서버에게 ETag가 달라졌는지를 검사해서 같다면 캐시된 데이터를 반환하게 된다.

하지만 hi brody에서 bye brody로 바꾼다면 Etag가 달라질 것이고 이는 요청에 담았던 Etag와 다르기 때문에 캐시데이터를 지우고 새로운 데이터를 받게 될 것이다. 

(클라이언트 쪽에서 요청헤더의 `If-none-Match`에 현재 Etag를 넣어서 서버에 요청한다.)

만약 변경된 사항이 없다면 304 Not Modified를 응답하게 될 것이다.


### 참고문서

-[Apple Docs - NSURLRequest.CachePolicy.useProtocolCachePolicy](https://developer.apple.com/documentation/foundation/nsurlrequest/cachepolicy/useprotocolcachepolicy)
-[cache Control이 필요한 이유](https://www.blog-dreamus.com/post/cache-control-%EC%9D%B4-%ED%95%84%EC%9A%94%ED%95%9C-%EC%9D%B4%EC%9C%A0)
-[알아둬야 할 HTTP 쿠키 & 캐시 헤더](https://www.zerocho.com/category/HTTP/post/5b594dd3c06fa2001b89feb9)
-[Caching best practice](https://jakearchibald.com/2016/caching-best-practices/)

230207 WWDC Understanding Swift Performance
===
TIL (Today I Learned)
===
학습내용
---
- WWDC Understanding Swift Performance 시청하기

## WWDC Understanding Swift Performance

성능 영향을 고려하면 보다 관용적인 솔루션을 찾는 데 도움이 된다. 그리고 Swift의 추상화 메커니즘이 성능에 미치는 영향을 이해하는 가장 좋은 방법은 기본 구현을 이해하는 것이다. 

인스턴스를 생성할 때 끊임없이 다음과 같은 질문을 던져야 한다.
- 내 인스턴스가 스택 또는 힙에 할당되는가
- 이 인스턴스를 전달할 때 얼마나 많은 참조 계산 오버헤드가 발생하는가
- 이 인스턴스에서 메소드를 호출할 때 정적 또는 동적으로 디스패치되는가

빠른 Swift코드를 작성하려면 활용하지 않는 역동성과 런타임에 대한 비용을 최대한으로 줄여야 한다.

### Allocation

Swift는 자동으로 메모리를 할당 및 해제한다. 그 메모리의 일부는 스택에 할당이 된다. 스택은 LIFO구조로 이루어져있고 스택의 끝을 가리키는 스택포인터를 단순히 올리거나 내리는 작업으로 메모리를 할당 혹은 해제할 수 있다. 이 작업은 O(1)시간으로 굉장히 빠르다.

힙에 메모리를 할당하려면 적절한 크기의 사용되지 않은 블록을 찾기 위해 힙 데이터 구조를 검색하고 할당해야 한다. 하지만 이러한 작업들이 주 비용은 아니다. 
더 큰 문제는 여러 스레드가 동시에 힙에 메모리를 할당할 수 있으므로 힙은 잠금 또는 동기화 메커니즘을 사용하여 무결성을 보호해야 한다. 이게 힙이 스택보다 느린 결정적인 이유이다.

몇 가지 코드를 통해 swift가 우리를 대신하는 것을 살펴보자.

### Struct 사용

#### 01 코드를 실행하기도 전에 `point1` 인스턴스와 `point2` 인스턴스를 위한 스택 공간을 할당했다. 
<img src="https://i.imgur.com/nONdLSc.png" width="500"/><br/>

#### 02 코드가 실행되면 이미 할당되어있는 메모리에 값을 넣어 초기화를 해줄 뿐이다. 
<img src="https://i.imgur.com/ltgcd0v.png" width="500"/>

위 과정은 스택포인터를 단순히 원래 위치로 증가시키는 것만으로 할당을 간단하게 해제할 수 있다. 

### Class 사용

#### 01 코드가 실행되기 전에 `point1`과 `point2`변수가 인스턴스를 참조하기 위한 메모리를 할당한다
<img src="https://i.imgur.com/7q0kMpP.png" width="500"/>

#### 02 코드 실행 시 `point1`변수는 Point인스턴스를 생성시키고 난 후 가리키게 되고, 프로퍼티로 가지고 있는 `x, y` 뿐만 아니라 우리를 대신하여 관리할 두 개의 공간을 더 할당한다.

<img src="https://i.imgur.com/RmeFB0R.png" width="500"/>

#### 03 `point2`변수에 `point1`을 할당했을 때 구조체처럼 값을 복사하는 것이 아닌 참조를 전달한다. 이는 의도치 않은 상태공유로 이어질 수 있다.
<img src="https://i.imgur.com/gNQKPGp.png" width="500"/><br/>

#### 04 이후 힙에서 사용되지 않는 인스턴스들을 제거하고 스택에서 변수를 pop해서 끝난다.

클래스는 힙에 할당되고 참조체계를 갖기 때문에 ID 및 간접 저장소와 같은 강력한 특성이 존재한다. 그러나 추상화를 위해 이러한 특성이 필요하지 않다면 구조체를 사용하는 것이 좋다.


### Swift코드 개선

메시징 앱에서 말풍선을 만드는 함수가 있다고 가정해보자.

사용자가 스크롤 중에 이 함수를 자주 호출하기 때문에 `cache`라는 변수를 만들어 한 번 로드가 되었다면 빠르게 꺼내서 사용할 수 있도록 만들었다. 

하지만 key값으로 `String`이 들어오는데 `String`은 힙 영역에 메모리가 할당되기 때문에 함수를 호출할 때 많은 비용이 걸리게 될 것이다. 그리고 `key`에는 `brody`라는 이름이 들어갈 수 있을 만큼 취약하다. 안전성 또한 없는 것이다.

<img src="https://i.imgur.com/Hck6kKp.png" width="600"/><br/> 


이를 개선해보자.

먼저 `String`으로 `key`값을 직렬화하는 대신 스택에 메모리가 할당되는 구조체를 만든다. `Hashable`만 채택해준다면 `Dictionary`의 `key`값으로 사용할 수 있다.

<img src="https://i.imgur.com/xuCOAxy.png" width="300"/><br/>


이제 더이상 `makeBallon` 함수를 호출할 때 캐시적중이 있는 경우 힙 할당이 필요하지 않게 되었다.

### 참조 카운트

swift는 `ARC`를 이용해 컴파일 시점에 타입안에`refCount`라는 변수를 삽입하고 변수가 참조되거나 해제될 때 `retain`, `release`를 삽입한다. 

<img src="https://i.imgur.com/DYaXcqJ.png" width="600"/>

### 복잡한 구조체

구조체는 스택 영역에 할당되기 때문에 힙에 할당되는 것보다 성능이 좋다고 위에서 밝혔다. 그럼 구조체 안에 프로퍼티들이 힙영역에 할당되는 경우에는 어떨까?

문자열과 UIFont의경우 힙 영역에 생성되기 때문에 참조 카운트가 필요하다.

<img src="https://i.imgur.com/fUefOHy.png" width="600"/>

만약 `label1`에 구조체 인스턴스를 만든 후 `label2`에 할당한다면 서로 다른 인스턴스가 두개가 스택영역에 할당이 되고 이 변수들의 프로퍼티들은 힙 영역에 쌓이게 되기 때문에 각각 참조 카운트가 2가 되어 총 4가된다.

<img src="https://i.imgur.com/jhMzdpS.png" width="600"/><br/><br/>


만약 이를 클래스로 구현했다면 다음과 같았을 것이다.

<img src="https://i.imgur.com/1ko83b3.png" width="400"/><br/><br/>

즉 구조체안에 힙 영역에 할당되는 프로퍼티들이 두 개이상존재한다면 클래스로 정의하는 것보다 참조계산 오버헤드가 더 많이 존재하게 된다.

### 구조체 내의 힙 영역 메모리 개선

다음과 같이 `String`을 갖는 `uuid`와 `mimeType`프로퍼티가 존재한다고 가정해보자. 이는 `Attachment`타입의 인스턴스가 생성될 때 힙 영역에 할당되기 때문에 참조 카운트를 세야하는 오버헤드가 발생할 것이다.

<img src="https://i.imgur.com/WLwyWZD.png" width="400"/>


#### 01 먼저 `uid`의 타입을 `UUID`로 변경한다. iOS6이후로 `UUID`라는 `struct`가 생겼는데 이는 무작위로된 128비트로 식별되어있고 `struct`에 직접 저장하기 때문에 힙에 할당이 필요없어진다.

<img src="https://i.imgur.com/3TVh5Eb.png" width="200"/>

#### 02 mimeType은 현재 `String`을 `extension`해서 사용하고 있는데 이는 결국 힙영역을 사용하고 있다.
<img src="https://i.imgur.com/MhCaDe7.png" width="200"/>

#### 03 mimeType을 `enum`으로 바꿔주어 스택영역에 저장해 힙 영역을 사용하지 않도록 수정해준다.
<img src="https://i.imgur.com/TyTff68.png" width="200"/>

### 메소드 디스패치

컴파일 타임에 어떤 구현을 실행하도록 결정할 수 있다면 이를 정적 디스패치라고 한다. 이는 인라인과 같은 것을 포함하여 적극적으로 최적화할 수 있게 만든다.


아래 코드에서 `drawPoint`는 사실 `draw`를 호출하는 것과 같다. 컴파일러는 이를 최적화하여 `drawPoint` 대신에 `draw`를 넣게 된다.<br/>
<img src="https://i.imgur.com/4dDM7kq.png" width="600"/>

위의 상황이 가능한 이유는 정적 디스패치의 체인은 컴파일러가 전체 체인을 통해 가시성을 갖게되기 때문이다. 반면에 동적 디스패치는 매 단계마다 추론하지 않고 상위 level에서 차단이 된다.

그럼 왜 동적(다이나믹) 디스패치를 사용하는 걸까?
강력한 장점은 **다형성**이다.

만약 `Drawable`클래스를 상속받은 `Point`와 `Line`이 있다고 가정해보자. 배열안에서는 이들에 대한 참조를 가지고 있기 때문에 같은 크기일 것이다.

<img src="https://i.imgur.com/igigdGK.png" width="600"/><br/><br/>

`for`문을 돌면서 `drawables`배열안의 인스턴스들의 `draw`를 호출하는데 이때 컴파일러 입장에서 어떤 구현을 실행해야 하는지 알 수 없다. 그럼 컴파일러는 어떻게 `draw`를 알고 호출하는 걸까?

컴파일러는 그 `class`타입 정보에 대한 것을 정적 메모리에 저장하고 실제로 `draw`를 호출할 때 `virtual method table`(vtable)을 조회한다. 그리고 실행하기에 적합한 `draw`를 찾고 파라미터로 실제 인스턴스를 전달한다.



- [WWDC - Understanding Swift Performance](https://developer.apple.com/videos/play/wwdc2016/416/)
- [Understanding Swift Performance](https://zeddios.tistory.com/596)

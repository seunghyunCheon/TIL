230225 Dispatch Queues, 뷰 컨트롤러 커스텀 이니셜라이저 구현, 동시성 프로그래밍
TIL (Today I Learned)
===
학습내용
---
- Dispatch Queues 번역
- 뷰 컨트롤러 커스텀 이니셜라이저 구현
- 동시성 프로그래밍

## Dispatch Queues
GCD는 작업을 수행하는데 좋은 도구이다. 디스패치 큐는 임의의 코드블럭을 동기적으로나 비동기적으로 실행할 수 있다. 이를 이용해 분리된 스레드에서 수행될 거의 모든 작업들을 수행할 수 있다. 디스패치 큐의 이점은 스레드기반의 코드에 비해 사용이 단순하고 효율적이다.

이 챕터는 디스패치 큐에서 어떻게 작업들을 실행할지에 대한 부분을 소개한다. 

### ✏️ About Dispatch Queues

디스패치 큐들은 비동기적으로 동시에 작업을 수행하기에 쉬운 방법이다. 하나의 작업은 애플리케이션이 수행할 일부 작업일 뿐이다. 예를 들어 계산을 수행할 작업을 정의할 수 있고, 데이터 구조를 만들고 수정하거나, 파일로부터 데이터를 읽는 등의 작업을 정의할 수 있다. 블럭객체나 함수 내부에 작업을 정의해서 디스패치 큐에 추가해서 사용할 수 있다.

디스패치큐는 객체와 비슷한 형태를 띄어 큐에 넣은 작업들을 관리하는 구조이다. 모든 디스패치큐는 FIFO구조이다. 그러므로 하나의 큐에 추가하려는 작업들은 항상 순차적으로 들어가고 나오게 된다. GCD는 자동으로 디스패치 큐를 제공하지만 다른 일부는 특정 목적을 위해 만들 수 있다. 

### Types of dispatch queues


| Type | Description | 
| -------- | -------- | 
| Serial     | Serial Queue들(private dispatch queue라고 불리는) 한번에 하나의 작업만 실행한다. 현재 실행중인 작업은 디스패치 큐에서 관리되는 구별된 스레드에서 동작한다. Serial queue는 구체적인 리소스에 대한 접근을 동기화할 때 사용한다.<br/> 필요에 따라 많은 Serial queue들을 생성할 수 있고 각각의 큐들은 다른 큐에서 동시에 동작한다. 다른말로 만약 4개의 serial queue가 존재한다면 각각의 큐는 오직 한번에 하나의 작업만 실행하지만 각 대기열에서 하나씩 총 4개의 작업을 동시에 할 수 있는 것이다. 
| Concurrent| Concurrent Queue는 (global Dispatch queue라고 불리는) 하나 이상의 작업들을 동시에 실행한다. 하지만 작업들은 여전히 FIFO형태로 시작을 한다. 현재 실행중인 작업은 구별된 스레드에서 작동한다. 지정된 지점에서 실행되는 작업의 수는 시스템 상태에 따라 변화된다.<br/> iOS5 이후로,`DISPATCH_QUEUE_CONCURRENT`를 명시하여 concurrent dispatch queue를 만들 수 있게 되었다. 추가적으로 미리 정의된 4개의 global concurrent queue가 존재한다. |
| Main dispatch queue| Main dispatch queue는 애플리케이션의 메인 스레드에 있는 전역적으로 이용가능한 Serial queue이다. 이 큐는 애플리케이션의 런루프와 함께 동작하고 큐에 있는 작업들과 런루프에 있는 이벤트들을 삽입한다. 메인 스레드에서 동작하기 때문에 메인큐는 key synchroniztion point로 자주 사용이 된다.|


## 뷰 컨트롤러 커스텀 이니셜라이저 구현
### 문제
스토리보드에 있는 뷰 컨트롤러를 불러와 커스텀 이니셜라이저를 이용해 인스턴스를 생성할 때 프로퍼티를 주입시키려고 했는데 
```swift
storyboard.instantiateViewController(withIdentifier: "ViewController")
```

위의 코드로 뷰 컨트롤러를 생성하면 `required initializer`인 `init(coder:)`가 호출되어 프로퍼티를 이니셜라이저에 넣어줄 수 없었다. 

### 해결방법
iOS13이후로 나온 
```swift
storyboard.instantiateViewController(identifier:, creator:)
```

위 코드를 사용하면 된다. 

```swift
// 정의
@MainActor func instantiateViewController<ViewController>(
    identifier: String,
    creator: ((NSCoder) -> ViewController?)? = nil
) -> ViewController where ViewController : UIViewController

// 사용 
if let itemDetailVC = storyboard?.instantiateViewController(identifier: "ItemDetailViewController", creator: { creator in
    let item = self.expositionItems[indexPath.row]
    let viewController = ItemDetailViewController(item: item, coder: creator)
    return viewController
}) {
    self.navigationController?.pushViewController(itemDetailVC, animated: true)
}
```
이 메서드는 스토리보드에서 만든 뷰컨트롤러에 대해 커스텀 이니셜라이저 코드를 만들 수 있는 메서드이다. `creator`인자에서 `NSCoder`를 받고 커스텀 이니셜라이저를 이용해 만든 뷰 컨트롤러에 주입할 데이터와 `NSCoder`를 넣어서 사용한다.

## 동시성 프로그래밍 

### ✏️ 코어와 스레드
컴퓨터는 동영상을 인코딩하면서 음악을 들을 수도있고, 음악을 들으면서 워드작업을 할 수도 있다. 이는 어떻게 가능한걸까?

### 코어
코어는 CPU의 핵심으로 실제로 일을 처리하는 부품이다. 코어는 한 번에 한가지 일을 하기에 여러개의 코어가 존재한다면 여러가지 일을 할 수 있다. 하지만 싱글코어도 여러 작업을 동시에 처리할 수 있다. 이는 한 가지 일만 처리하고있는데 여러가지 일을 작은 단위로 쪼개어 번갈아가면서 작업을 하는 것이기 때문에 동시에 진행되는 것처럼 보이는 것일 뿐이다. 

이렇게 여러 가지 작업을 시분할로 나누어 번갈아 가며 처리하는 것을 동시성 프로그래밍이라고 한다.(병렬 프로그래밍과는 차이점이 있다.)

코어가 많으면 무조건 좋은 것은 또 아니다. 보통은 더 좋지만 싱글코어 최적화된 소프트웨어가 있을 수도 있고 어떤 작업에 대해 어느 코어로 작업을 할당해주는 것도 비용이 들기 때문에 이 비용이 싱글코어에서 작동하는 비용보다 크다면 더 느릴 수도 있다.

### 스레드
스레드는 하드웨어의 스레드와 소프트웨어의 스레드 두 개로 나뉜다.

하드웨어에서의 스레드는 하이퍼스레딩 기술을 이용하여 하나의 코어로 2가지 작업을 동시에 수행할 수 있는 논리적인 코어이다. 일반적으로 컴퓨터를 알아볼때 보이는 스레드가 이에 해당한다.

소프트웨어에서의 스레드는 논리적인 스레드로 프로그램 내부에서 작업 단위가 되는 가상의 스레드이다. 이 소프트웨어 스레드는 가상으로 만들어진 스레드이기 때문에 물리적인 스레드 갯수와는 상관없이 많은 스레드를 만들 수 있다.

### ✏️ 동시성 프로그래밍과 병렬 프로그래밍

### 병렬 프로그래밍
병렬 프로그래밍은 여러개의 CPU(코어)가 하나의 작업을 분담해서 처리하는 것이다. 계산이 복잡하고 큰 그래픽을 처리한다고 할 때 사용이 된다.
실세계의 예시로 컨퍼런스 발표를 준비한다면 누군가는 화면을 내리고, 누군가는 컴퓨터를 키고, 누군가는 프로젝트 빔을 쏘는 작업을 하고 있는 것과 같다. 이는 혼자하는 것보다 여러명이 같이하는 것이 효율을 낼 가능성이 높다.

그래서 병렬 프로그래밍은 물리적인 개념으로 CPU(코어)가 여러 개 있을 때 가능하다. 실제로 동시에 작업을 하는 것이기 때문에 싱글코어에서는 불가능하다.
운영체제에서 알아서 관리를 해주는 영역이기에 이를 직접 구현하는 일은 iOS에서는 없을 것이다.

### 동시성 프로그래밍
동시성 프로그래밍은 하나의 CPU가 여러 작업을 동시에 처리하는 것이다. 동시성 프로그래밍은 병렬 프로그래밍과 다르게 싱글코어에서도 가능한 논리적인 개념이다. 

여러 개의 스레드를 이용하여 동시에 여러 작업을 처리하며 이는 실제로 동시에 처리하는 것은 아니지만 빠르게 switching하기 때문에 동시에 처리하는 것처럼 보여진다.

동시성 프로그래밍과 반대되는 개념으로 직렬성 프로그래밍이 있는데 동시성은 여러개의 스레드를 이용하는 반면 직렬성은 하나의 스레드만 이용한다.


### 동기와 비동기

동기는 어떤 작업이 끝날때까지 기다리는 것이다. 
비동기는 어떤 작업이 끝날떄까지 기다리지않고 다음 코드 블록을 바로 실행시키는 것이다. 
이러한 개념의 차이는 실행 종료 시점을 알 수 있는가에 대한 차이로 이어진다.

동시성은 비동기와 개념이 다르다.
- 동시성: 단일 스레드인가 다중 스레드인가
- 동기/비동기: 스레드의 수와 무관하게 작업이 끝나기를 기다리냐, 안기다리냐

Serial환경에서 비동기로 처리할 수도, 동기로 처리할 수도 있고 Concurrent환경에서 동기, 비동기로 처리할 수도 있다.

### 동시성 프로그래밍은 왜 필요할까?

여러개의 스레드에서 작업들을 분담해서 처리하기 때문에 효율적으로 프로그래밍할 수 있게 된다. 그리고 이는 사용성과 반응성의 향상으로 귀결되기 때문에 사용자에게 좋은 경험을 주는 관점에서도 필요하다.

스위프트에서 동시성 프로그래밍을 구현하는 방법으로는 `GCD`, `Operation`, `async/await`이 있고 GCD는 Operaiton의 근간이 되는 API이다. 즉 GCD에 세부적인 기능들이 추가된 API이다.




### 참고자료
- [Apple Archive - Concurrency and Application Design](https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/ConcurrencyandApplicationDesign/ConcurrencyandApplicationDesign.html#//apple_ref/doc/uid/TP40008091-CH100-SW1)
- [Apple Developer Documentation - instantiateviewcontroller](https://developer.apple.com/documentation/uikit/uistoryboard/3213989-instantiateviewcontroller)
- [UIViewController 서브클래스의 custom initializer 만들기](https://velog.io/@dev_jane/UIViewController-%EC%84%9C%EB%B8%8C%ED%81%B4%EB%9E%98%EC%8A%A4%EC%9D%98-custom-initializer-%EB%A7%8C%EB%93%A4%EA%B8%B0required-initializer-initcoder-must-be-provided-by-subclass-of-UIViewController)
- [야곰닷넷 - 동시성프로그래밍](https://yagom.net/courses/%eb%8f%99%ec%8b%9c%ec%84%b1-%ed%94%84%eb%a1%9c%ea%b7%b8%eb%9e%98%eb%b0%8d-concurrency-programming/)

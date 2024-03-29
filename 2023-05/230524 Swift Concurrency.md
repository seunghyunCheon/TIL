# 230524 Swift Concurrency
### 학습내용
- Swift Concurrency

## Swift Concurrency

스위프트는 구조화된 방법으로 비동기와 병령 코드를 작성하는 것을 지원하고 있다.
비동기 코드는 중단되고 후에 재게될 수 있다. 
프로그램에서 코드를 일시 중지하고 다시 시작하면 네트워크를 통해 데이터를 가져오거나 파일을 구문 분석하는 것과 같은 장기 실행 작업을 계속하면서 UI 업데이트와 같은 단기 작업을 계속 진행할 수 있.

병렬코드는 코드의 여러부분이 동시에 동작하는 것을 의미한다. 예를 들어 네개의 코어를 가진 컴퓨터는 같은시간에 네 개의 코드작업을 실행할 수 있다. 이는 각각의 코어가 하나의 작업을 갖기 때문이다. 병렬과 비동기 코드를 가진 코드는 여러개의 작업을 한 번에 할 수 있다. 외부 시스템을 기다리는 작업을 일시 중단하고 메모리 안전 방식으로 이 코드를 더 쉽게 작성할 수 있다.

병렬 또는 비동기 코드의 추가 스케줄링 유연성은 복잡성 증가 비용과 함께 제공된다. Swift는 일부 컴파일 시간 검사를 가능하게 하는 방식으로 의도를 표현할 수 있. 예를 들어 액터를 사용하여 변경 가능한 상태에 안전하게 액세스할 수 있다. 하지만 느리거나 버그가 있는 코드에 동시성을 추가한다고 해서 코드가 빠르고 정확해진다는 보장은 없다.

사실 동시성을 추가하는 것은 디버깅하기 어렵게 만든다. 하지만 스위프트 언어레벨에서 동시성을 사용하는 것은 컴파일 시간에 문제를 잡아내도록 스위프트가 도와줄 수 있다는 의미이다.

기존의 동시성 코드는 컴플리션 핸들러로 코드를 작성해야 하기 때문에 중첩 클로저를 작성하게 된다. 이는 중첩이 깊어져 코드가 복잡해지면 다루기 어려워질 수 있다.

### 비동기 함수 정의하고 호출

비동기 함수나 메소드는 실행도중에 멈출 수 있는 특수한 종류의 함수 또는 메소드이다. 
이렇게 함수가 비동기임을 알려주기 위해 `async`라는 키워드를 명시해야 한다. 

```swift
func listPhotos(inGallery name: String) async -> [String] {
    let result = // ... network
    return result
}
```

비동기 메서드를 호출할 때 호출 앞에 `await`를 작성하여 잠시 멈춤 가능지점을 표시할 수 있다. 비동기 메서드 안에서는 또 다른 비동기 메서드를 호출할때만 실행흐름이 잠시 멈춘다. 잠시멈춤은 절대로 암시적 또는 선점이 아니다. 이는 `await`로 잠시 멈춤 가능한 모든 지점을 표시한다는 의미이다. 

예를 들어 다음 코드는 전시관의 모든 사진 이름을 가져온 다음 첫 번째 사진을 보여준다.

```swift
let photoNames = await listPhotos(inGallery: "Summer Vacation")
let sortedNames = photoNames.sorted()
let name = sortedNames[0]
let photo = await downloadPhoto(named: name)
show(photo)
```

`listPhotos`와 `downloadPhoto`함수 둘 다 네트워크 요청이 필요하기 때문에 완료까지 긴 시간이 걸릴 수 있다. 이 함수 앞에 `async`작성으로 둘 다 비동기로 만들면 이 코드가 사진이 준비되길 기다리는 동안 앱 나머지 코드가 실행을 유지하게 해준다.

순서는 이렇다.
- `listPhotos`함수를 호출하고 그 함수가 반환하길 기다리는 동안 실행을 잠시 멈춘다.
- 이 코드 실행이 잠시 멈춘 동안, 동일한 프로그램의 일부 다른 동시성 코드를 실행한다. 예를 들어 오래 걸리는 백그라운드 임무가 새로운 사진 전시관 목록의 갱신을 계속할 수도 있을 것이다.  그 코드도 `await`로 표시한 다음 멈춤지점까지 또는 완료할 때까지 실행한다.
- `listPhotos` 반환 후 그 지점에서 코드의 실행을 계속한다. 반환 값을 `photoNames`에 할당한다.
- `sortedNames`와 `name`을 정의하는 줄은 표준적인 동기코드이다. 이 줄엔 `await`표시가 아무것도 없기 때문에 잠시 멈춤가능지점이 없다.
- `downloadPhhotos`도 `listPhotos`처럼 잠시멈춘다음 값을 반환한다.

코드에서 `await`로 표시한 잠시 멈춤 가능지점은 비동기 함수나 메서드의 반환을 기다리는 동안 현재 코드 조각을 일시정지할지도 모른다고 지시한다. 이를 쓰레드 넘겨주기라고 하는데 스위프트가 그 이면에서 현재 쓰레드에서의 코드실행을 잠시 멈추고 그 대신 그 쓰레드에서 어떠한 다른 코드를 실행하기 때문이다. `await`가 있는 코드는 잠시 멈출 수 있어야 하기 때문에 비동기 함수나 메소드는 프로그램의 정해진 곳에서만 호출할 수 있다. 
- 비동기 함수나, 메서드 또는 본문 안에 있는 코드
- @main으로 표시한 구조체나 `main()`안에 있는 코드
- 비동기 메서드 안에 Task.sleep과 같은 시간이 걸리는 코드가 있는 경우

```swift
func listPhotos(inGallery name: String) async -> [String] {
    await Task.sleep(2 * 1_000_000_000)
    return ["IMG001", "IMG99", "IMG0404"]
}
```

### 비동기 시퀀스

한번에 배열 전체를 비동기로 반환하는 것이 아닌 비동기 시퀀스를 사용하여 한번에 한 집합체 원소를 기다릴 수도 있다. 
```swift
import Foundation

let handle = FileHandle.standardInput
for try await line in handle.bytes.lines {
    print(line)
}
```

평범한 `for-in`반복문을 사용하는 대신 위 코드에서는 `for`뒤에 `await`을 작성한다. 이는 잠시 멈춤가능 지점을 지시하기에 다음 원소의 사용을 기다릴 때 각 회차의 맨 앞에서 실행을 잠시 멈출 가능성이 있다. 

이는 자신만의 타입을 `for-in` 반복문에서 사용하려면 `Sequence`프로토콜을 준수하면 되는 것과 똑같이, 자신만의 타입을 `for-await-in` 반복문에서 사용하려면 `AsyncSequence`프로토콜을 준수하면 된다.

### 비동기 함수를 병렬로 호출하기

`await`코드는 한번에 한 조각의 코드만 실행한다. 비동기 코드를 실행하는 동안 그 코드가 종료하길 기다리고 다음 코드를 실행하는 것이다.
따라서 다음 코드는 세 개의 `downloadPhoto`함수를 차례대로 기다린 후에 `show`함수가 호출이 될 것이다.
```swift
let firstPhoto = await downloadPhoto(named: photoNames[0])
let secondPhoto = await downloadPhoto(named: photoNames[1])
let thirdPhoto = await downloadPhoto(named: photoNames[2])

let photos = [firstPhoto, secondPhoto, thirdPhoto]
show(photos)
```

이러한 결점을 해결하기 위해 상수 정의 때 `let` 앞에 `async`를 작성한 다음, 그 상수를 사용할 때마다 `await`을 작성하면 된다.
```swift
async let firstPhoto = downloadPhoto(named: photoNames[0])
async let secondPhoto = downloadPhoto(named: photoNames[1])
async let thirdPhoto = downloadPhoto(named: photoNames[2])

let photos = await [firstPhoto, secondPhoto, thirdPhoto]
show(photos)
```

위 코드는 새 `downlaodPhoto`가 이전처럼 완료하길 기다리지 않는다. 대신 세 개의 결과를 `await`하고 있기 때문에 동시적으로 발생한 세개의 함수가 모두 끝이 나야 `photos`에 담기게 된다.

이 두 접근법의 차이점은 이렇다.
- 다음 줄 코드가 그 함수 결과에 의존할 땐 `await`를 호출한다.
- 코드 나중에서야 결과가 필요할 때는 `async-let`로 비동기 함수를 호출한다. 이는 병렬로 실시할 수 있는 작업을 생성
- `await`와 `async-let`둘 다 자신이 잠시 멈춘 동안 다른 코드가 실행하는 것을 허용한다.
- 두 경우 모두 잠시 멈춤 가능지점을 `await`로 표시하여 필요하다면, 비동기 함수가 반환할 때까지 실행을 일시정지할 것을 지시한다.

### Task and Task Groups

임무(task)는 프로그램에서 비동기로 실행할 수 있는 작업단위이다. 모든 비동기 코드는 어떠한 임무의 일부분으로써 실행이 된다. 위 코드의 `async-let`구문은 자식 임무를 생성한다. 임무 그룹을 생성하여 그 그룹에 자식 임무를 추가할 수도 있는데, 이러면 우선 순위와 취소 작업을 더 잘 제어할 수 있으며 동적 개수의 임무도 생성하게 해준다.

임무는 계층 구조로 배열한다. 임무 그룹에 있는 각각의 임무는 동일한 부모 임무를 가지며, 각각의 임무도 자식 임무를 가질 수 있다. 임무와 임무 그룹 사이의 명시적인 관계 때문에 이러한 접근법을 구조화된 동시성(structed concurrency)라고 한다. 올바로 할 책임은 개발자가 져야 하지만, 임무 사이의 명시적인 부모-자식관계는 취소 전파와 같은 일부 동작을 스위프트가 처리하게 하며 컴파일 시간에 일부 에러도 스위프트가 탐지하게 해준다.
```swift
await withTaskGroup(of: Data.self) { taskGroup in
    let photosNames = await listPhotos(inGallery: "Summer Vacation")
    for name in photoNames {
        taskGroup.async { await downloadPhoto(named: name) }
    }
}
```

### 구조화 안된 동시성

스위프트는 구조화되지않은 동시성도 지원한다. 임무 그룹의 일부인 임무들과는 달리 구조화 안된 임무에는 부모 임무가 없다. 구조화 안된 임무는 프로그램에 필요하다면 무슨 방식으로든 완전히 유연하게 관리할 수 있지만, 그게 올바른 지도 완전히 책임져야 한다. 구조화 안된 임무를 현재 행위자에서 실행하도록 생성하려면 `Task.init(priority:operation:)`초기자를 호출한다. 

```swift
let newPhoto = // ... 어떠한 사진 자료 ...
let handle = async {
  return await add(newPhoto, toGalleryNamed: "Spring Adventures")
}
let result = await handle.get()
```
### 참고문서
- [WWDC - Combine in Practice](https://developer.apple.com/videos/play/wwdc2019/721/)

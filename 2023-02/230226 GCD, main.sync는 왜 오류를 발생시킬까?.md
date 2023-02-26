230226 GCD, main.sync는 왜 오류를 발생시킬까?
TIL (Today I Learned)
===
학습내용
---

## GCD

일을 효율적으로, 동시에 처리하기 위해서는 여러개의 스레드를 사용해야 한다. 개발자는 이 스레드를 직접 관리하지 않고 동기/비동기만 처리해준다면 시스템이 알아서 코어와 스레드를 관리해준다. `GCD`는 애플이 멀티 코어 환경과 멀티 스레드 환경에서 최적화된 프로그래밍을 할 수 있도록 개발한 기술이며 이를 이용해 동기/비동기 코드를 멀티스레드에서 다룰 수 있다.

### ✏️ DispatchQueue: Serial / Concurrent

`DispatchQueue`는 `GCD`의 일부로 이 대기열에 작업을 추가만 하면 시스템이 알아서 작업을 처리하도록 도와준다. `FIFO` 형태로 동작하며 이에 작업을 보낼때 단일 스레드를 사용할 것인지 다중 스레드를 사용할 것인지, 동기를 사용할지 비동기를 사용할지 결정해주어야 한다.

Dispatch 프레임워크에는 `DispatchQueue`, `DispatchSource`, `DispatchGroup` 등 여러개의 클래스가 존재한다. 

### Serial / Concurrent

DispatchQueue를 초기화할 때 `attributes`를 따로 `.concurrent`하지 않으면 기본값이 `serial`이 된다.

```swift
// Serial Queue
DispatchQueue.main

// Concurrent Queue
DispatchQueue.global()
```

`main` 프로퍼티는 앱이 구동되고 계속 상주하는 스레드로 Serial 큐이기 때문에 동시에 여러작업을 처리할 수 없다. 반면에 `global`에 작업을 추가하면 새로운 스레드를 만들어 그 위에서 작업을 한다.

`global`은 `main`과 달리 메모리에 올라왔다가, 작업이 끝나면 제거된다. 

아래 코드는 sync/async까지 적용한 코드이다.
```swift
// 동기, sync
DispatchQueue.main.sync {}
DispatchQueue.global().sync {}
 
// 비동기, async
DispatchQueue.main.async {}
DispatchQueue.global().async {}
```

### ✏️ 메인 스레드

메인 스레드는 앱이 켜지고 끝날때까지 살아있는 스레드이다. 즉 생명주기가 같다. 동시성 프로그래밍에서는 여러개의 스레드를 사용한다고 했는데 이 스레드들이 메인스레드에서 파생되어 생겨난 스레드인 것이다.

메인 스레드 특징
- 전역적으로 사용가능
- global스레드와 다르게 Run Loop가 자동으로 설정되고 실행. 메인 스레드에서 동작하는 Run Loop를 Main Run Loop라고 함.
- UI 작업은 메인 스레드에서만 작업할 수 있다.

### ✏️ DispatchQueue 2: sync / async

`Dispatch.global().sync {}` 는 메인 스레드 말고 새로운 스레드를 만들어서 작업을 처리할거야. 그런데 내 작업이 끝날때까지 기다려야해! 라는 의미이다.

`Dispatch.global().async {} `는 main스레드말고 새로운 스레드에서 작업할 건데 다음 작업이 있다면 날 기다리지말고 처리해도 좋아. 나는 새 스레드에서 알아서 작업할게! 라는 의미이다.

### main.async

```swift
import Foundation

DispatchQueue.main.async {
    for _ in 1...5 {
        print("😀😀😀😀😀")
        sleep(1)
    }
}

DispatchQueue.main.async {
    for _ in 1...5 {
        print("🥶🥶🥶🥶🥶")
        sleep(2)
    }
}
```
main 스레드라는 단일 스레드에서만 작업이 이루어지기 때문에 동시에 이모티콘이 보이지 않고 처음 async코드가 끝난 후에 두번째 async코드 블럭이 실행된다.

### global().async
```swift
import Foundation

DispatchQueue.global().async {
    for _ in 1...5 {
        print("😀😀😀😀😀")
        sleep(1)
    }
}

DispatchQueue.global().async {
    for _ in 1...5 {
        print("🥶🥶🥶🥶🥶")
        sleep(2)
    }
}

DispatchQueue.main.async {
    for _ in 1...5 {
        print("🥵🥵🥵🥵🥵")
        sleep(1)
    }
}
```

첫번째와 두번째 async코드는 글로벌로 만들어진 concurrent 큐이기 때문에 새로운 스레드에서 작업하지만 세번째 async코드는 메인스레드에서 작업한다. 

그리고 이들은 다른 스레드에서 async로 동작하기 때문에 동시에 나오기에 실행순서를 예측할 수 없다.

### global().sync
```swift
import Foundation

DispatchQueue.global().sync {
    for _ in 1...5 {
        print("😀😀😀😀😀")
        sleep(1)
    }
}

DispatchQueue.global().sync {
    for _ in 1...5 {
        print("🥶🥶🥶🥶🥶")
        sleep(2)
    }
}

for _ in 1...5 {
    print("🥵🥵🥵🥵🥵")
    sleep(1)
}
```

global을 사용해 다른 스레드에서 작업하지만 sync를 붙어있기에 첫번째 이모지가 다 출력된후에 두번째 이모지가 다나온다. 그다음에서야 메인스레드인 for문이 출력된다. 

즉 세 코드블럭은 전부 다른 스레드지만 동기적으로 일을 처리하기에 각각의 작업이 끝나기를 기다린다. 

### main.sync

main.sync를 직접 호출하면 `deadlock`(교착상태)에 빠지게 된다. 이는 async와는 반대되는 작업이 끝나기를 기다리는 Sync의 특성 때문이다. sync는 코드블록이 처리되기 전까지 다음코드로 넘어가지 않는다. 이러한 상황을 `Block-wait`이라고 하는데, 코드 블록이 끝나기 전까지 그 스레드는 멈춰있겠다는 뜻이다. 

따라서 main 스레드에서 Main.sync를 호출하면 main 스레드는 Sync의 코드 블록이 수행되기를 기다려야 한다. 하지만 이때 sync의 코드 블록은 main 스레드에서 실행되고 있던 코드이기 때문에 멈춘게 된다. 따라서 main 스레드는 sync가 끝나기를, sync는 main 스레드의 Block-wait이 끝나기를 기다리는 상태가 되어버린다.

정리하면 main스레드에서 메인큐에 작업을 sync로 보내면 자신의 스레드를 block한 뒤 작업이 끝날때까지 접근할 수 없게 만든다. 하지만 메인큐는 작업을 반드시 메인 스레드에 보내야하는데 메인 스레드는 block된 상태이기에 서로가 기다리는 상태여서 교착상태가 일어나는 것이다. 
메인 스레드에서 커스텀직렬큐를 만들어 작업을 보낼때는 커스텀직렬큐가 다시 메인스레드로 보내는것이아닌 스레드1과 같은 새로운 스레드에 할당하기 때문에 문제가 생기지 않는다.

만약 메인스레드가 아닌 다른 새로운 concurrent큐에서 main.sync를한다면 다른 스레드가 멈추고 main에거 동기적으로 코드블록을 실행할 수 있기에 global을 사용하면 된다. 

```swift
import Foundation

DispatchQueue.global().async {
    DispatchQueue.main.sync {
        for _ in 1...5 {
            print("😀😀😀😀😀")
            sleep(1)
        }
    }
}

for _ in 1...5 {
    print("🥶🥶🥶🥶🥶")
    sleep(2)
}
```

### DispatchQueue 3: 그 외 다양한 기능들

### DispatchWorkItem
DispatchQueue에서 코드블록을 호출하는 것 대신에 `DispatchWorkItem`을 활용해 코드 블록을 캡슐화할 수 있다. 

```swift
import Foundation

let red = DispatchWorkItem {
    for _ in 1...5 {
        print("🥵🥵🥵🥵🥵")
        sleep(1)
    }
}

let yellow = DispatchWorkItem {
    for _ in 1...5 {
        print("😀😀😀😀😀")
        sleep(1)
    }
}

let blue = DispatchWorkItem {
    for _ in 1...5 {
        print("🥶🥶🥶🥶🥶")
        sleep(2)
    }
}

DispatchQueue.main.async(execute: yellow)
DispatchQueue.global().async(execute: blue)
DispatchQueue.global().sync(execute: red)
```

위 실행은 파란색과 빨간색이 동시에나온 후에 마지막에 노란색이 출력된다.

### asyncAfter
asyncAfter는 async 메서드를 원하는 시간에 호출하는 메서드이다. 
```swift
DispatchQueue.global().asyncAfter(deadline: .now + 5, execute: yellow)
```
현재 now로부터 5초후에 async코드를 실행한다.

```swift
DispatchQueue.global().asyncAfter(WallDeadline: .now() + 5, execute: blue)
```

wallDeadline은 시스템(기기)를 기준으로 카운트하는 것이다. (지금 5시니까 5초에 작업 시작해야지와 같은 작업 수행)



### 참고자료
- [야곰닷넷 - GCD](https://yagom.net/courses/%eb%8f%99%ec%8b%9c%ec%84%b1-%ed%94%84%eb%a1%9c%ea%b7%b8%eb%9e%98%eb%b0%8d-concurrency-programming/lessons/%eb%8f%99%ec%8b%9c%ec%84%b1-%ed%94%84%eb%a1%9c%ea%b7%b8%eb%9e%98%eb%b0%8d/)
- [Main.sync](https://limjs-dev.tistory.com/106)

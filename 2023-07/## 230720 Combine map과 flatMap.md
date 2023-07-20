## 230720 Combine map과 flatMap

### map

map은 Publisher로 부터 전달되는 값을 변환하는 메서드이다. 
```swift
let publisher = [1, 2, 3, 4].publisher
let cancellable = publisher
    .map { $0 * 2 }
    .sink(receiveValue: { print($0) })  // 2, 4, 6, 8
```

### flatMap

> upstream publisher의 모든 요소를 지정한 최대 publishers 수까지 새로운 publishers로 변환합니다.

flatMap의 가장 큰 특징은 새로운 publisher를 반환하는 것이다. 단순 데이터 변환이 아닌 새로운 퍼블리셔를 가져오고 싶을 때. 예를 들어 모델에 있는 데이터를 로드하거나 비동기작업을 수행한 뒤에 가져온 값을 기반으로 추가 작업을 하고 싶을 때 사용한다.

프로젝트 진행 중 단순하게 모델에 있는 값을 가져오고 싶어서 다음과 같이 적용했다.
```swift
input.viewDidLoadEvent
    .flatMap { [weak self] _ -> AnyPublisher<[NotificationCycle], Never> in
        guard let self else { return }
        return self.goalSettingUseCase.notificationCycle()
    }
    .sink { notificationCycleList in
        output.notificationCycleList = notificationCycleList
    }
    .store(&cancellables)
```

하지만 viewDidEvent가 일어날 때 많은 정보를 받아야 한다면 위 코드는 가독성이 좋지 않을 수 있다. 그래서 notificationCycle을 로드하는 함수는 호출하되 뷰모델에서 useCase의 notificationCycle 변수를 감지하도록 변경했다.

```swift
input.viewDidLoadEvent
    .sink { [weak self] in
        self?.goalSettingUseCase.loadNotificationCycle()
    }
    .store(in: &cancellables)

self.goalSettingUseCase.notificationCycleList
    .sink { notificationCycleList in
        output.notificationCycleList.send(notificationCycleList)
    }
    .store(in: &cancellables)
```
### 참고문서
- [map과 flatMap](https://zeddios.tistory.com/1031)

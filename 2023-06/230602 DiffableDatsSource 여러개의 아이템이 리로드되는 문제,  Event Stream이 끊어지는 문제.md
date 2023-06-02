# 230602 DiffableDatsSource 여러개의 아이템이 리로드되는 문제,  Event Stream이 끊어지는 문제
## 여러개의 아이템이 리로드 되는 문제

### 🔍 문제점
컬렉션 뷰의 데이터소스로 존재하는 배열이 변경될 때마다 스냅샷에서 그 배열의 전체 id를 가져와 `reloadItems`를 사용하여 업데이트 했습니다.
```swift
func bind() {
    viewModel.$planList
        .sink { isUpdating, planList in
            if isUpdating {
                self.updateSnapshot(planList)
            }
        }
}

func updateSnapshot(planList: [Plan]) {
    let planIDList = planList.map { $0.id }
    let snapshot = Snapshot()
    snapshot.appendSection([.main])
    snapshot.reloadItems([planIDList])
    dataSource.apply(snapshot)
}
```
하지만 위 방법은 스냅샷에서 `hashable`한 모델 자체를 들고있는 것이 아니라 `id`만을 들고 있기 때문에 `id`에 해당하는 내용이 변경되더라도 `id`는 변경되지 않기 때문에 리로드되지 않는 문제가 발생했습니다. 

그래서 다음과 같이 변경했습니다.

```swift
func updateSnapshot(planList: [Plan]) {
    let snapshot = dataSource.snapshot()
    let planIDList = planList.map { $0.id }
    snapshot.reloadSection([.main])
    snapshot.reloadItems([planIDList])
    dataSource.apply(snapshot)
}
```

하지만 위 코드는 셀의 업데이트가 발생했을 때 전체를 리로드하는 문제가 존재했고 삭제기능 또한 마찬가지였습니다.
</br>

###  ⚒️ 해결방안

### 생성, 수정, 삭제에 대한 스냅샷 함수를 분리하여 문제를 해결했습니다. 

`taskList`자체를 구독하고 있다면 디테일한 기능을 정의 할 수 없었기 때문에 `taskList`를 `Publisher`로 두고있던 코드를 제거했습니다.

만약 스냅샷에서 관리하고 있는 것이 `id`가 아닌 `Hashable`한 모델이라면 `dataSource`에 전부 가져와서 `apply`만 호출했을때 `DiffableDatasource`가 자체적으로 차이를 비교하여 업데이트를 할 것이기 때문에 디테일한 기능을 정의할 필요가 없을 것입니다. 

하지만 현재 `DiffableDataSource`에는 `Hashable`한 모델을 들고다니는 것이 아닌 `UUID`를 들고다니기 때문에 디테일한 기능정의가 필요했기 때문에 `taskList`의 `Publisher`를 제거했습니다.

그리고 생성, 삭제, 수정 이벤트에 대한 `Publisher`를 정의했습니다.
```swift
protocol PlanListViewModel: AnyObject {
    var planList: [Plan] { get set }
    var planCountChanged: PassthroughSubject<Int, Never> { get }
    var planCreated: PassthroughSubject<Int, Never> { get }
    var planUpdated: PassthroughSubject<UUID, Never> { get }
    var planDeleted: PassthroughSubject<(Int, UUID), Never> { get }
    // ...
}
```
생성, 삭제, 수정이 일어났을 때 각각의 이벤트 퍼블리셔에 `send`를 통해 연관된 값(ex. 삭제될 id, 업데이트 될 id 등)을 발행함으로써 연관된 작업에 맞게 데이터소스가 변경될 수 있도록 했습니다.

```swift
// PlanCollectionViewController.swift 
private func bindState() {
    viewModel.planCreated
        .sink { [weak self] planListCount in
            self?.applyLatestSnapshot()
            self?.viewModel.planCountChanged.send(planListCount)
        }
        .store(in: &cancellables)

    viewModel.planUpdated
        .sink { [weak self] planID in
            self?.reloadItems(id: planID)
        }
        .store(in: &cancellables)

    viewModel.planDeleted
        .sink { [weak self] (planListCount, planID) in
            self?.deleteItems(id: planID)
            self?.viewModel.planCountChanged.send(planListCount)
        }
        .store(in: &cancellables)
}

private func applyLatestSnapshot() {
    let planIDList = viewModel.planList.map { $0.id }

    var snapshot = Snapshot()
    snapshot.appendSections([.main])
    snapshot.appendItems(planIDList)
    dataSource?.apply(snapshot)
}

private func reloadItems(id: UUID) {
    guard var snapshot = dataSource?.snapshot() else { return }

    snapshot.reloadItems([id])
    dataSource?.apply(snapshot)
}

private func deleteItems(id: UUID) {
    guard var snapshot = dataSource?.snapshot() else { return }

    snapshot.deleteItems([id])
    dataSource?.apply(snapshot)
}
```


</br>

##  Event Stream이 끊어지는 문제
### 🔍 문제점
Combine을 활용하여 뷰에서 발생한 Event의 Publisher를 구독할 때, Stream이 ViewModel 내부에서 구독이 이루어져 다음과 같은 문제가 있었습니다.

1. 구독이 ViewController와 ViewModel에서 모두 이루어지고 있어 이벤트 처리 과정의 파악이 어렵다.
2. 구독의 수가 늘어나면 저장되는 cancellable의 인스턴스가 늘어나 메모리 사용량이 늘어날 것이다.

```swift
// ViewModel 구현부...
struct Input {
    let titleTextEvent: AnyPublisher<String?, Never>
    let bodyTextEvent: AnyPublisher<String?, Never>
}

struct Output {
    let isEditingDone = PassthroughSubject<Bool, Error>()
}

private var cancellables = Set<AnyCancellable>()

func transform(input: Input) -> Output {
    let output = Output()

    input.titleTextEvent
        .sink(receiveValue: { [weak self] title in
            guard let self else { return }

            self.title = title
            output.isEditingDone.send(true)
        })
        .store(in: &cancellables)

    input.bodyTextEvent
        .sink(receiveValue: { [weak self] body in
            guard let self else { return }

            self.body = body
            output.isEditingDone.send(true)
        })
        .store(in: &cancellables)

    return output
}
```

</br>

###  ⚒️ 해결방안
ViewModel에서 cancellables를 제거하고, operator를 통해 이벤트를 통해 받은 publisher를 가공하여 관련된 publisher를 하나의 스트림으로 merge했습니다.

merge된 publisher를 Output으로 전달해 ViewController에서만 구독이 1회만 일어나게 되어 연결된 스트림을 가질 수 있도록 해결했습니다.

```swift
struct Input {
    let titleTextEvent: AnyPublisher<String?, Never>
    let bodyTextEvent: AnyPublisher<String?, Never>
    let datePickerEvent: AnyPublisher<Date, Never>
}

struct Output {
    let isEditingDone: AnyPublisher<Bool, Error>
}

func transform(input: Input) -> Output {
    let titlePublisher = input.titleTextEvent
        .tryMap { [weak self] title in
            guard let title else { throw EditError.nilText }
            guard let self else { throw EditError.nilViewModel }

            self.title = title

            return true
        }

    let bodyPublisher = input.bodyTextEvent
        .tryMap { [weak self] body in
            guard let body else { throw EditError.nilText }
            guard let self else { throw EditError.nilViewModel }

            self.body = body

            return true
        }

    let datePublisher = input.datePickerEvent
        .tryMap { [weak self] date in
            guard let self else { throw EditError.nilViewModel}

            self.date = date

            return true
        }

    let isEditingDone = titlePublisher
        .merge(with: bodyPublisher, datePublisher)
        .eraseToAnyPublisher()

    return Output(isEditingDone: isEditingDone)
}
```

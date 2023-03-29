230328 UICollectionViewListCell과 ContentConfiguration 커스텀해서 사용하기
===
학습내용
- UICollectionViewList는 뭐가 좋은걸까?
- UICollectionViewListCell과 ContentConfiguration 커스텀해서 사용하기

## UICollectionViewList는 뭐가 좋은걸까?
iOS14이후로 애플은 list기반의 특징을 가지고 있는 `UICollectionViewListCell`을 소개했다. 기존의 컬렉션 뷰 셀은 테이블 뷰셀처럼 리스트기반의 틀이 정해져 있지 않았기에 separator, indentaion, accessroy View등 직접 구현해줘야 했다. 

### TableView의 기본 레이아웃을 탑재
테이블 뷰와 비슷한 외형으로 5가지 기본형태를 지원한다(grouped, insetGrouped, plain, sidebar, sidebarPlain)

### 간단한 애니메이션 
그리고 만약 셀을 선택했을 때 배경색 변경에 대한 애니메이션을 주고 싶다면 애니메이션 코드안에 `self.setNeedsUpdateConfiguration()`을 사용하면 된다. 이 메서드는 셀에게 현재 state로 업데이트하도록 알리는 메서드로 `updateConfiguration`메서드를 호출한다. 

```swift
class MyCell: UICollectionViewListCell {
    var item: MyItem?
    
    override var isSelected: Bool {
        didSet {
            UIView.animate(withDuration: 0.5) {
                self.setNeedsUpdateConfiguration()
            }
        }
    }
    
    override func updateConfiguration(using state: UICellConfigurationState) {
        // 현재 state에 맞는 기본 구성을 가져온다
        var content = self.defaultContentConfiguration().updated(for: state)

        content.image = self.item?.icon
        content.text = self.item?.title

        // 현재 state에 맞게 기본구성의 스타일을 변화시킨다
        if state.isHighlighted || state.isSelected {
            content.imageProperties.tintColor = .white
            content.textProperties.color = .white
        } else {
            content.imageProperties.tintColor = .black
            content.textProperties.color = .black
        }

        // 변화된 구성을 현재 셀의 구성으로 교체시킨다.
        self.contentConfiguration = content
    }
}
```
<img src="https://i.imgur.com/MnlwmqZ.gif" width="500"/><br/>


하지만 위 방법은 `defaultContentConfiguration`을 사용하고 있기 때문에 기본 셀의 외형에 대해서만 지원한다. 즉 `text`, `secondaryText`, `image` 프로퍼티 등 몇가지만 존재하고 오토레이아웃도 잡혀있는 상태이다. 

그래서 ContentConfiguration을 이용해 좀 더 복잡한 셀을 구성하기 위해서는 커스텀 셀과 커스텀 구성을 사용해야 한다.

### 커스텀 CollectionViewListCell, 커스텀 ContentConfiguration을 이용한 애니메이션

커스텀으로 이들을 구성하는 방법에 대해선 아래에서 구체적으로 다룰 예정이지만 애니메이션 부분만 간단히 먼저 살펴보자.

먼저, `defaultContentConfiguration`을 이용한 방식에서는 접근할 수 없었던 ContentView를 직접 정의해야 한다.

```swift
class MyContentView: UIView, UIContentView {
    // 컨텐츠 뷰 내부에서 보여질 프로퍼티 정의
    
    func setupAllView() {
        // 오토레이아웃 설정
    }
    
    func apply(configuration: MyContentConfiguration) {
        UIView.animate(withDuration: 0.2) {
            self.backgroundColor = configuration.customBackColor
        }
        
        // 다른 프로퍼티들 설정
    }
}
```
기존의 방식과는 다르게 ContentView 내부에서 애니메이션 코드를 작성할 수 있다. 직접 정의했기 때문에 더 구체적으로 어떤 부분에서 애니메이션을 줄 수 있는지 결정할 수 있는 것이다. 

### 상태에 따른 셀의 외형, 외형, 데이터 주입을 분리
`automaticallyUpdatesContentConfiguration`와 `automaticallyUpdatesBackgroundConfiguration` 프로퍼티가 true로 되어있기 때문에 셀의 state가 변하면 자동으로 업데이트하게 된다.

만약 `UICollectionViewCell`을 이용한다면 상태에 따른 외형을 결정하는 코드도 `configure`하는 부분에 포함되어있기 때문에 역할분리에 한계가 있을 것이다.

이외에도 복잡한 버그 제거나 성능 최적화의 이유로 `UICollectionViewList`와 `ContentConfiguration`의 사용을 권장하고 있다.

Configuration State(구성 상태)를 이용해서 다이나믹 타입이나 커스텀 State도 정의할 수 있는데 이는 나중에 알아봐야겠다...



## UICollectionViewListCell과 ContentConfiguration 커스텀해서 사용하기

직접 사용하기에 앞서 다음 세 가지 개념을 먼저 알고가보자

#### content view
- `UIView`의 서브클래스이자 `UIContentView` 프로토콜을 채택. 
- 커스텀 셀의 레이아웃과 외형을 정의하고 주어진 `content configuration`에 기반한 올바른 데이터와 외형을 보여준다.

#### content configuration
- `content view`의 뷰 모델로 `UIContentConfiguration` 프로토콜을 채택한다. 
- 커스텀 셀을 위한 `content view`를 생성한다. 그러므로 이는 `content view`와 `custom cell`의 다리 역할을 한다.

#### custom cell
- `UICollectionViewListCell`을 서브클래싱하고 있는 셀로, 적절히 설정된 `content configuration`을 state(selected, highlighed)에 기반해서 생성시키는 역할만 한다. 그리고 그것을 자신의 configuration에 할당한다.

<img src="https://i.imgur.com/JC8pJuF.png" width="700"/><br/>

### 프로젝트 API
`DiffableDataSource`, `CompositionalLayout`, `UICollectionViewListCell`사용

### BoxOfficeDTO, BoxOfficeItem
서버를 통해 받아온 전체 데이터들을 받기 위한 모델인 `BoxOfficeDTO`와 실제로 앱에서 보여지는 데이터를 담은 `BoxOfficeItem`모델로 구성 
`BoxOfficeItem`의 id프로퍼티는 추후에 `DiffableDataSoruce`의 itemIdentifier로 쓰인다.

```swift
struct BoxOfficeDTO: Decodable {
    let boxOfficeResult: BoxOfficeResult
    
    struct BoxOfficeResult: Decodable {
        // ...
    }
}

struct BoxOfficeItem: Identifiable {
    var id = UUID()
    let rank: String
    // ...
    
    init(rank: String,
         rankIncrement: String,
         // ...
    ) {
        self.rank = rank
        self.rankIncrement = rankIncrement
        // ...    
    }
}
```

### BoxOfficeContentConfiguration
`UIContentConfiguration`과 `Hashable` 프로토콜을 채택하여 서브클래스를 만든다.
Content View의 뷰모델이 될 것이기 때문에 위에서 정의한 `BoxOfficeItem`타입의 데이터들과 동일한 프로퍼티들을 포함하고, 스타일에 대한 프로퍼티도 정의해야 한다.

아래 코드를 보면 알 수 있듯이 `makeContentView`를 통해 현재 설정된 `configuration`을 통해 `ContentView`를 만들 수 있고, `updated`를 통해 현재 state에 따라 셀의 구성을 변화하고 이를 반환할 수 있다.

```swift
struct BoxOfficeContentConfiguration: UIContentConfiguration, Hashable {
    var rank: String?
    var rankIncrement: String?
    var rankColor: UIColor?
    
    func makeContentView() -> UIView & UIContentView {
        return BoxOfficeContentView(configuration: self)
    }
    
    func updated(for state: UIConfigurationState) -> Self {
        guard let state = state as? UICellConfigurationState else {
            return self
        }
        
        let updatedConfiguration = self
        
        if state == .isSelected {
            // ...
        } else {
            // ...
        }
        
        return updatedConfiguration
    }
}
```

### BoxOfficeContentView
```swift
class BoxOfficeContentView: UIView, UIContentView {
    private var currentConfiguration: BoxOfficeContentConfiguration!
    var configuration: UIContentConfiguration {
        get {
            return currentConfiguration
        }
        set {
            guard let newConfiguration = newValue as? BoxOfficeContent else {
                return
            }
            
            apply(configuration: newConfigureation)
        }
    }
    
    let rankLabel: UILabel
    // ...
    
    func setupAllViews() {
        // 오토레이아웃 설정
    }
    
    func apply(configuration: BoxOfficeContentConfiguration) {
        guard currentConfiguration != configuration else {
            return
        }
        
        currentConfiguration = configuration
        
        rankLabel.text = configuration.rank
        // configuration에 담긴 데이터, 스타일 설정
    }
}
```

처음에는 `BoxOfficeContentView`에서 `configure`속성을 갖는다는 점과 `ContentView`는 어느시점에서 생성이되는지 감이 안왔다.

확실하지는 않지만 메서드의 호출을 찍어본 결과 `ContentConfiguration`과 `ContentView`의 예상되는 흐름은 다음과 같다.

1. `DiffableDataSource`에서 재사용 셀을 dequeue한다
2. 등록된 셀을 반환한다. 이때 cell의 데이터가 설정된다.
3. ListCell내부에 override한 `updateConfiguration(using state)` 메서드를 호출한다. 
4. `BoxOfficeContentConfiguration`생성자를 통해 만들고 들어온 state를 넣어 해당 state에 맞는 `ContentConfiguration`을 반환한다.
5. 반환된 구성에 cell내부에 있는 데이터를 전달한다
6. List셀의 `contentConfiguration`에 전달된 데이터가 있는 `newConfiguration`을 할당한다

여기까지가 사실이고 이후부터는 예상하는 내용이다

7. ListCell에 할당된 구성을 통해 `ContentView`를 만든적이 없다면 `makeContentView`에 현재 구성을 넣어 호출해서 `ContentView`를 만든다.
8. `ContentView`의 오토레이아웃을 잡는다
9. `apply(configuration)`을 통해 현재 들어온 구성을 기반으로 `contentView`에 보여질 UI요소들에 데이터를 넣는다. ex) `rankLabel.text = configuration.rank`

그리고 만약 셀을 클릭했다면 다음과 같은 흐름이다.
1. `BoxOfficeContentConfiguration` 내의 `updated`함수가 호출되어 상태에 맞는 `configuration`을 구성하고 반환한다
2. 1번에서 반환한 구성이 `contentView`의 `apply`메서드의 인자로 들어가서 호출이된다.
3. 위 순서의 3-6번을 따른다.
4. `apply`메서드가 할당된 `newConfiguration`을 인자로받아 실행된다.

셀 클릭시 바로 `updateConfiguration(using)`메서드로 갈 줄 알았는데 `Configuration`의 `updated`메서드로 먼저 가게된다. 그리고 `apply`를 한 뒤에 `updateConfiguration(using)`이 호출된다.

1번과 2번의 작업이 없어도 올바르게 상태에 따라 업데이트가 될 것 같은데 왜 들어있는지는 이유를 모르겠다. (최적화의 문제일 수도)

그래서 본 글을 정리하면 다음과 같다

### 정리
- `UICollectionViewListCell`, `UIContentConfiguration`, `UIContentView`를 커스텀해서 만들면 `UICollectionViewListCell`을 좀 더 다양하고 역할이 분리된 형태로 만들 수 있다.
- `BoxOfficeContentConfiguration`의 `updated`메서드에서는 현재 변경된 상태를 기반으로 구성상태의 스타일을 변화시키는 메서드이다.
- `ListCell`의 `updateConfiguration`메서드는 셀이 처음 만들어질 때와 상태가 변할때마다 호출이 되며 변경된 구성을 자신의 `contentConfiguration`에 넣는 작업을 한다.
- 아마 makeContentView메서드는 ListCell내부에 `contentConfiguration`에 구성을 할당하면서 ContentView가 존재하지않을때 호출할 것 같다. 그리고 존재한다면 해당 메서드를 실행하지 않을 것 같다.

```swift
struct BoxOfficeContentConfiguration: UIContentConfiguration, Hashable {
     func makeContentView() -> UIView & UIContentView {
        print(#function)
        return BoxOfficeContentView(configuration: self)
    }   
}
```

### 참고문서
- [Apple Docs - UICollectionViewListCell](https://developer.apple.com/documentation/uikit/uicollectionviewlistcell)
- [Apple Docs - contentConfiguration](https://developer.apple.com/documentation/uikit/uitableviewcell/3601057-contentconfiguration)
- [WWDC - Modern cell configuration](https://developer.apple.com/videos/play/wwdc2020/10027/)
- [UICollectionView List with Custom Cell and Custom Configuration](https://swiftsenpai.com/development/uicollectionview-list-custom-cell/)


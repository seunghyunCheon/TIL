230324 UIListContentView, moya코드를 통한 API 통신관리(associatedtype), 애플 공식예제를 보면서 Compositional Layout 톺아보기 - 1
===
학습내용
- UIListContentView
- moya코드를 통한 API 통신관리(associatedtype)
- 애플 공식예제를 보면서 Compositional Layout 톺아보기 - 1

## UIListContentView
list기반의 컨텐츠를 보여주는 content view
```swift
@MainActor class UIListContentView: UIView
```

### Overview

커스텀 뷰 계층안에서 리스트기반의 content를 보여주는 content view를 사용한다. 커스텀 셀이나 `UIStackview`와 같은 컨테이너 뷰안에 안에 list content view를 추가한다. `Auto Layout`이나 수동적인 레이아웃 기술을 사용해서 그 view를 위치시킬 수 있다. 그리고 그 뷰의 높이는 보여지는 내용에 따라(width, space) 자동으로 조절된다. 
즉 dynamic type에 대응이 된다는 뜻

list content view는 스타일과 내용을 제공하기 위해 list content configuration에 의존한다. `init`할 때 `UIListContentConfiguration`을 전달받아서 만들 수 있다. 
content view를 업데이트하기 위해서는 새로운 configuration을 `configuration` 프로퍼티에 설정해야 한다.

만약 `UICollectionView`나 `UITableView`를 사용한다면 수동으로 list content view를 만들 필요가 없다. 대신에 cell, header, footer에 존재하는 `contentConfiguration` 프로퍼티에 할당하면 된다.

## associatedtype

프로토콜 내에서 일부로 사용하는 타입을 위한 placeholder역할을 한다.

현재 프로젝트에서 APIProvdier의 역할을 하는 `BoxOfficeProvider` 인스턴스를 만들때 프로토콜을 채택하는 제네릭 변수로 선언을 했을때 associatedtype을 사용하지않으면 범용성이 떨어지는 일이 있다.

글로 보면 잘 모르겠으니 코드로 확인해보자.

### Requestable
서버에 있는 데이터를 요청할 수 있는 프로토콜이다.
```swift
func fetchData<T: Decodable>(_ target: API, type: T.Type, completion: @escaping (Result<T, Error>) -> Void)
```

### BoxOfficeProvider
`Requestable`을 채택하는 타입으로 프로토콜에 정의된 `fetchData`메서드를 구현해서 준수하고 있다.
```swift
final class BoxOfficeProvider<Target: API>: Requestable {
func fetchData<T>(_ target: API, type: T.Type, completion: @escaping (Result<T, Error>) -> Void) where T : Decodable {
    guard let endPoint = self.makeEndpoint(for: target),
          let request = endPoint.urlRequest() else {
        return
    }
     // ...URL Session task   
    }
}
```

### BoxOfficeAPI
API 프로토콜을 채택하는 구체적인 타입이다. 
```swift
enum BoxOfficeAPI {
    case dailyBoxOffice(date: String)
    case detailMovieInformation(movieCode: String)
}

extension BoxOfficeAPI: API {
    // ...something
}
```
### BoxOfficeViewController
APIProvider를 정의하고 어떤 API를 받아서 사용할지 제네릭을 이용해 인스턴스를 만들어 사용하는 부분이다.
```swift
    let boxOfficeProvider = BoxOfficeProvider<BoxOfficeAPI>()
    boxOfficeProvider.fetchData(target: API, type: Decodable,Protocol, completion: (Result<Decodable, Error>) -> Void)
```

위 코드를 보면 `fetchData`의 인자로 target에는 API타입을 넣어줘야 한다. `BoxOfficeProvider` 내부의 `fetchData`정의부에 target은 API타입을 받기에 당연하다. 

하지만 이렇게 되면 API를 채택하고 있는 구체적인 타입을 사용하지 못한다. 목적이 프로토콜을 사용하는 것이 아닌 실제 타입인 `BoxOfficeAPI`의 타입을 사용해야하기 때문이다. 

이때 associatedtype을 사용해서 문제를 해결할 수 있다.

associatedtype은 프로토콜을 채택했을 때 정해지는 타입이기 때문에 `BoxOfficeProvider`를 이니셜라이저할때 들어오는 제네릭타입이 associatedtype이 된다. 그리고 이는 `fetchData`에서 target의 타입이기 때문에 API프로토콜을 채택하면서 구체적인 타입이 들어올 수 있게 되는 것이다.

즉 채택했을 때 정해지는 타입을 쓸 때 활용이 된다. 그리고 이는 제네릭과 함께 사용된다면 프로토콜에서 정의한 모든 메서드의 인자나 리턴값으로 활용될 수 있기 때문에 더욱 강력하다.(만약 제네릭이 존재하지 않는다면 채택하는 곳에 `typealias Target = Int` 등의 코드를 작성하거나 메서드에서 직접 타입을 넣어줘서 associatedtype을 설정해줘야 한다)
```swift
protocol Requestable {
    associatedtype Target
    func fetchData<T: Decodable>(_ target: Target, type: T.Type, completion: @escaping (Result<T, Error>) -> Void)
}
```

```swift
func fetchData<T>(_ target: Target, type: T.Type, completion: @escaping (Result<T, Error>) -> Void) where T : Decodable {
}
```

정리를 했을때 이제 `let boxOfficeProvider = BoxOfficeProvider<BoxOfficeAPI>()`이 코드를 실행하는 순간 제네릭 타입인 Target에 `BoxOfficeAPI`가 들어오고, 이는 곳 Target의 타입이 되어 `fetchData`
메서드에서 target인자의 타입이 `API`타입이 아닌 구체화된 `BoxOfficeAPI`타입이 오는 것이다.

## 애플 공식예제를 보면서 Compositional Layout 톺아보기

처음으로 가장 기본적인 Grid형식으로 Compositional Layout을 사용하고자 한다. 

먼저 코드로 컬렉션뷰에 compositionalLayout을 적용하기 위해서는 컬렉션뷰 인스턴스를 만들때 Layout을 만들어줘야 한다.

```swift
collectionView = UICollectionView(frame: view.bounds, collectionViewLayout: createLayout())
```

`createLayout`메서드는 `UICollectionViewLayout`타입을 반환하는 메서드인데`UICollectionViewCompositionalLayout`은 이를 채택하고 있기 때문에 `UICollectionViewCompositionalLayout`를 반환해도 된다. 

코드를 살펴보기 전에 Compositional Layout에서 사용되는 기본적인 타입을 알 필요가 있다.

### NSCollectionLayout Item
컬렉션 뷰 레이아웃의 가장 기본적인 컴포넌트
```swift
@MainActor class NSCollectionLayoutItem: NSObject
```
하나의 item은 컬렉션 뷰내에서 내용에 대한 크기, 간격, 정렬방법에 대해 알려주는 청사진이다. 
하나의 item은 렌더링되는 화면에 보여지는 단일 뷰를 표현한다. 일반적으로 하나의 아이템은 셀이지만 헤더, 푸터, decoration 등이 될 수 있다.

예를 들어 포토앱에서 item은 단일 사진을 표현하고, App Store앱에서는 app icon, app name, download button등을 표현하고 있다. 

각각의 Item은 가로와 높이 dimension에 따라 사이즈를 명시한다. Items는 사이즈를 그들의 container와 상대적인 치수로 표현할 수 있다. 그게 절대값이 될 수도 있고 런타임에 따라 변할 수 있는 estimated 값이 될 수도 있다. 이를 표현하는 타입이 `NSCollectionLayoutDimension`타입이다.

items를 서로에 대해 어떻게 배치할지를 결정하는 group으로 결합시킬 수 있다. 이를 표현하는 타입이 `NSCollectionLayoutGroup`이다.


### NSCollectionLayoutDimension
컬렉션 뷰 내 item의 너비와 높이를 표현하는 dimension(치수). iOS에서는 13이상에서 지원한다.
```swift
@MainActor class NSCollectionLayoutDimension: NSObject
```

컬렉션 뷰 내의 아이템들은 명시적인 너비와 높이 치수를 표현하는 `NSCollectionLayoutDimension`를 가지는데 이 타입 두개(너비와 높이 각각)은`NSCollectionLayoutSize`로 결합이 된다.

item의 치수를 absolute, estimated, fractional 값으로 표현할 수 있다.

44 x 44 point square와 같은 정확한 치수에는 absolute value를 사용한다.
```swift
let absoluteSize = NSCollectionLayoutSize(widthDimension: .absolute(44),
                                         heightDimension: .absolute(44))
```

만약 내용이 런타임환경에서 변경될 때(데이터가 로드되거나 시스템 폰트 사이즈에 반응할 때) estimated value를 사용한다. 초기 estimated Size를 정할 수 있고, 후에 시스템이 실제 값을 계산한다.

```swift
let estimatedSize = NSCollectionLayoutSize(widthDimension: .estimated(200),
                                          heightDimension: .estimated(100))
```

item의 컨테이너와의 상대적인 치수를 정의하고 싶을 때는 fractional value를 사용한다. 이는 aspect ratio를 표현하는 것을 단순화한다. 
예를 들어 다음의 코드는 container의 width에 대해 20%의 크기를 가지고 height는 컨테이너 width의 20%이기 때문에 1:1비율을 갖게 된다.

```swift
let fractionalSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(0.2),
                                           heightDimension: .fractionalWidth(0.2))
```

### NSCollectionLayoutGroup
item들의 path를 그리는 item 집합을 위한 컨테이너
```swift
@MainActor class NSCollectionLayoutGroup: NSCollectionLayoutItem
```

Group은 컬렉션 뷰 내에서 아이템들을 서로에 대해 상대적으로 어떻게 배치할지를 결정한다. 
하나의 group은 item들을 가로로 배치할 지, 세로로 배치할 지, 커스텀으로 배치할 지를 결정한다. 그리고 서로 관련하여 어떻게 아이템들을 랜더링할 지에 대한 규칙도 정의하지만 그 자체로는 내용을 렌더링 하지는 않는다.

각각의 그룹은 width dimension과 height dimension을 이용해 size를 정의한다.
그룹도 사이즈를 아이템이 size를 정할때와 마찬가지로 absolute, estimated, fractional을 이용해서 정의할 수 있다.

group은 `NSCollectionLayoutItem`을 subclass하고 있기 때문에 item처럼 행동한다. 그룹을 다른 아이템과 그룹을 합쳐서 복잡한 레이아웃을 구성할 수 있다.

<img src="https://i.imgur.com/W4QCmHw.png" width="500"/><br/>

그룹을 구성한 뒤에는 `NSCollectionLayoutSection`의 인자로 사용해야 한다.

### NSCollectionLayoutSection
group의 집합들을 구별되는 grouping으로 결합하는 컨테이너
```swift
@MainActor class NSCollectionLayoutSection: NSObject
```

컬렉션 뷰는 하나이상의 section을 가진다. section은 레이아웃을 구별되는 형태로 분리시키는 방법을 제공한다. 

각각의 섹션은 컬렉션 뷰 내에서 같은 레이아웃을 가질 수도있고, 다른 레이아웃을 가질 수도 있다. section의 레이아웃은 group(`NSCollectionLayoutGroup`)의 프로퍼티에 의해 결정이 된다. 이는 바로 위에서 본 것처럼 section을 만들 때 사용되기 때문이다.

포토앱에서는 각 섹션에 대해 같은 레이아웃을 사용하지만 앱스토어에서는 다른 것을 볼 수 있다.
각가의 섹션은 다른 섹션과 구별되게 하는 섹션 고유의 배경색, 헤더, 푸터를 가질 수 있다. 

다시 본론으로 돌아와서 `createLayout`함수를 살펴보자.

<img src="https://i.imgur.com/foBiWZd.png" width="500"/><br/>

위 형태의 grid를 표현하기 위해 다음과 같이 정의한다.

```swift
private func createLayout() -> UICollectionViewLayout {
    let itemSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(0.2),
                                         heightDimension: .fractionalHeight(1.0))
    let item = NSCollectionLayoutItem(layoutSize: itemSize)

    let groupSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                          heightDimension: .fractionalWidth(0.2))
    let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize,
                                                     subitems: [item])

    let section = NSCollectionLayoutSection(group: group)

    let layout = UICollectionViewCompositionalLayout(section: section)
    return layout
}
```

item의 width는 부모 container(group)의 1/5 크기이므로 0.2로 설정한다. height는 부모 컨테이너의 height를 따르게 만들기 위해 1.0으로 설정한다.

group의 width는 컬렉션뷰 컨테이너의 width를 따를 것이기 때문에 1.0으로 설정하고, height는 width와 1:1 비율로 주기위해 0.2로 준다.

### TwoColumnViewController

<img src="https://i.imgur.com/QUNqxeY.png" width="400"/><br/>

위의 레이아웃을 그리려면 어떻게 해야할까?

2가지 방법이 존재하는데 첫 번째 방법은 위에서 설명한 Grid방식과 동일한 방법을 이용한다.

```swift
func createLayout() -> UICollectionViewLayout {
    let itemSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(0.5),
                                          heightDimension: .fractionalHeight(1.0))

    let item = NSCollectionLayoutItem(layoutSize: itemSize)

    let groupSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                           heightDimension: .absolute(44))

    let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitems: [item])
    
    let section = NSCollectionLayoutSection(group: group)
    let spacing = CGFloat(10)
    section.interGroupSpacing = spacing
    section.contentInsets = NSDirectionalEdgeInsets(top: 0, leading: 10, bottom: 0, trailing: 10)
    let layout = UICollectionViewCompositionalLayout(section: section)
    
    return layout
}       
```

위의 코드만으로는 다음의 그림을 나타낸다.

<img src="https://i.imgur.com/Zv2jrgE.png" width="400"/><br/>

그룹 내의 아이템 사이의 간격을 추가했을 때 맨 위의 화면이 예상되었는데 다음화면처럼 보여졌다. 
```swift
group.interItemSpacing = .fixed(spacing)
```
<img src="https://i.imgur.com/qNKij9J.png" width="400"/><br/>

item의 Size에서 Width가 절반으로 정해진 상태에서 고정값인 10을 추가해서 width값이 화면을 넘겨서 넘어가서 생기는 것이다.

#### 이는 어떻게 해결해야 할까?

초기에 itemSize를 정하지 않고 그룹에서 item의 개수를 정해줘서 설정해줄 수 있다. 이는 item의 사이즈가 명확히 정해지지 않았기 때문에 중간에 10이라는 고정 space를 준 뒤에 사이즈가 정해지게 된다. 

위 방법은 사이즈가 정해진 뒤에 10을 주었기 때문에 넘어간 것이기 때문에 확실히 사이즈가 정해지는 시점이 다른 것 같다.

```swift
let itemSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                     heightDimension: .fractionalHeight(1.0))
let item = NSCollectionLayoutItem(layoutSize: itemSize)

let groupSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                      heightDimension: .absolute(44))
let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitem: item, count: 2)
let spacing = CGFloat(10)
group.interItemSpacing = .fixed(spacing)
```

item의 사이즈가 일관적이라면 이를 사용하는 게 좋다고 생각이 든다.






### 참고자료
- [Moya github](https://github.com/Moya/Moya)
- [associated type](https://zeddios.tistory.com/382)
- [Apple docs - NSCollectionlayoutItem](https://developer.apple.com/documentation/uikit/nscollectionlayoutitem)
- [Apple docs - NSCollectionalayoutDimension](https://developer.apple.com/documentation/uikit/nscollectionlayoutdimension)
- [Apple docs - NSCollectionLayoutGroup](https://developer.apple.com/documentation/uikit/nscollectionlayoutgroup)
- [Apple docs - Compositional Layout](https://developer.apple.com/documentation/uikit/views_and_controls/collection_views/implementing_modern_collection_views#4080030)

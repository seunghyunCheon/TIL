230321 UICollectionView, URLsession이 비동기적으로 작업을 처리하는 이유
===
학습내용
---
- UICollectionView
- URLsession이 비동기적으로 작업을 처리하는 이유

## UICollectionView
순서가 있는 데이터컬렉션들을 관리하고 커스터마이즈한 레이아웃을 사용하여 보여주는 객체

```swift
@MainActor class UICollectionView: UIScrollView
```

### 개요
컬렉션 뷰를 도입할 때 주된 일은 컬렉션 뷰와 연관된 데이터를 관리하는 것이다. 컬렉션 뷰는 컬렉션 뷰의 `dataSoruce` 프로퍼티안에 저장된 객체로부터 데이터를 얻는다. 
Data Source로는 `UICollectionViewDiffableDataSource`를 사용할 수 있다. 이를 사용하면 단순하고 효율적으로 업데이트하고 관리할 수 있다. 대안적으로 `UICollectionViewDataSource` 프로토콜을 채택함으로써 커스텀 data source객체를 만들 수 있다.

컬렉션 뷰 내의 데이터는 섹션 안의 그룹이 될 수 있는 아이템으로 정돈된다. 아이템은 보여지는 가장 작은 단위이다.
예를 들어 포토앱에서 아이템은 단일 이미지가 될 것이다. 컬렉션 뷰는 `dataSource`에서 제공하고 수정할 수 있는 `UICollectionViewCell` 클래스 인스턴스인 셀을 사용해서 아이템을 보여준다.

셀에 더하여 컬렉션 뷰는 다른 타입의 뷰를 이용하여 데이터를 보여줄 수 있다. 
예시로 섹션 헤더와 푸터가 있다. 이를 supplementary View라고 하는데 선택사항이고 collection view의 레이아웃 객체에 의해 정의가 된다. 

`UICollectionView`를 추가하는 것 외에도 아이템들의 시각적인 부분을 data source 객체 내의 순서와 매치시키도록 하기위해 메서드를 사용할 수도 있다. `UICollectionViewDiffableDataSource`는 이를 자동적으로 해준다. 만약 커스텀 data source를 사용한다면 데이터를 컬렉션 뷰에서 추가하거나 지우거나 재정렬할 때마다 `UICollectionView`의 메서드를 사용해야 한다. (일치하는 셀을 정하기 위해)

그리고 선택된 아이템을 관리하기 위해 컬렉션 뷰 객체를 사용할 수도 있지만 이 방법은 `delegate` 객체와 함께 동작한다.

### Layouts
레이아웃 객체는 컬렉션 뷰 내의 시각적인 내용 배치를 정의한다. `UICollectionViewLayout`의 서브클래스인 레이아웃 객체는 셀의 조직과 위치, 컬렉션 뷰 내의 부가적인 뷰를 정의한다. 비록 이 레이아웃 객체가 이러한 내용을 정의하더라도 실제로 정보를 상응하는 뷰에 적용시키지는 않는다. 컬렉션 뷰가 레이아웃 정보를 상응하는 뷰에 적용시킨다. 왜냐하면 셀과 부가적인 뷰들의 생성은 컬렉션 뷰와 data Source 객체 사이에서 조화를 수반하기 때문이다. 
레이아웃 객체는 데이터 대신에 시각적인 정보를 제공한다는 것을 제외하면 또 다른 data source와도 같다.

일반적으로 컬렉션 뷰를 만들때 레이아웃 객체를 명시하지만 또 컬렉션 뷰의 레이아웃을 동적으로 바꿀 수도 있다. 레이아웃 객체는 `collectionViewLayout` 프로퍼티 내에 저장되어 있다. 이 프로퍼티를 직접적으로 설정하면 변화를 애니메이션하는 것 없이 레이아웃을 즉각적으로 업데이트한다. 만약 애니메이션을 넣고 싶다면 `setCollectionViewLayout(_:animated:completion:)`메서드를 대신 사용하면 된다.


상호작용하는 전환을 만들기 위해(gesture recognizer나 touch event와 같은) 레이아웃을 변경시키기 위한 메서드로 `startInteractiveTransition(to:completion:)`메서드를 사용한다. 이 메서드는 중간의 레이아웃 객체를 설치하는데 이 객체는 gesture recognizer나 이벤트 처리코드와 함께 동작하여 전환 진행을 추적한다. -> 상호적인 transition 행동을 관리하기 위한 중간 transition 레이아웃 객체라고 보면 됨.
이벤트 처리코드가 transition이 끝났음을 결정할 때 `finishInteractiveTransition`이나 `cancelInterativeTransition`을 호출하면 된다. -> 이때 중간 객체가 제거되고 target 레이아웃 객체가 설치된다.

### Cells and supplementary views

컬렉션 뷰의 data source 객체는 아이템의 내용과 뷰 둘 다 제공한다. 컬렉션 뷰가 처음 내용을 로드할 때 data source에게 보여질 view를 제공해달라고 요청한다.
컬렉션 뷰는 data source가 재사용을 위해 표시하는 큐, view 객체의 리스트를 유지하고 있다. 새로운 뷰를 계속해서 만드는 것 대신에 view를 dequeue해서 사용하는 것이다. 

view를 dequeue하는데에는 두 가지 메서드가 존재한다. 
- 컬렉션 뷰 내의 하나의 셀을 얻고자 할 때 `dequeueReusableCell(withReuseIdentifier:for:)`를 사용한다.
- 레이아웃 객체에 의해 요청된 supplementary view를 얻고자 할 때 `dequeueReuseableSupplementaryView(ofKind:withReuseIdentifier:for:)`를 이용한다.

이러한 메서드들을 호출하기 전에 컬렉션뷰에게 어떻게 어떻게 상응하는 뷰를 만들지 말해줘야 한다. 
이를 위해 하나의 클래스나 컬렉션 뷰와 연결된 nib file을 등록시켜야 한다. 예를들어 셀을 등록할 때 `register(_:forCellWithReuseIdentifier)`메서드를 사용해서 클래스나 nib file을 등록시키는 것이다. 
등록절차로 뷰를 식별할 수 있는 `reuse identifier`를 명시해야 한다. 이는 후에 dequeue할 때 사용하는 string이다.

### Data prefetching
컬렉션 뷰는 반응성을 향상시키기 위해 두가지 prefetching 기술을 제공한다.
- `Cell prefetching`은 셀들이 요구되기 이전에 미리 준비를 한다. 그리드 레이아웃의 새 셀 행과 같이 컬렉션뷰가 동시에 많은 양을 요구할 때 필요한 식나보다 먼저 셀을 요청한다. 따라서 셀 렌더링은 여러 레이아웃에 분산되어있어 더 부드러운 스크롤하는 경험을 준다. 이는 기본적으로 활성화되어있다.
- `Data prefecthing`은 셀이 요구하기 전에 컬렉션뷰에서 필요한 데이터를 알리는 메커니즘이다. 셀 내용이 네트워크 요청과 같은 비싼 데이터 로딩 절차를 가질 때 유용하다. 셀에 대한 데이터를 프리패치할 때 이에 대한 알림을 받기 위해 `UICollectionViewDataSourcePrefetching`프로토콜을 채택한 객체를 `prefetchDataSource`프로퍼티에 할당한다. 

### Reorder items interactively

컬렉션뷰는 유저와의 상호작용에 기반하여 아이템을 옮길 수 있게 만들어준다. 일반적으로 컬렉션 뷰 내에서 아이템의 순서는 data source에 기반한다. 만약 사용자가 아이템을 재배치할 수 있도록 만드려면 `gesture recognizer`를 이용해 유저 상호작용을 추적하게 만들 수 있다. 

인터렉티브한 아이템 재배치를 하기 위해서는 컬렉션뷰의 `beginInteractiveMovementForItem(at:)` 메서드를 호출해야 한다. gesture recognizer가 터치 이벤트를 추적하는 동안 터치 이벤트를 보고하기 위해 `updateInteractiveMovementTargetPosition(_:)`메서드를 호출한다. 추적을 멈췄을 때는 `endInteractiveMovement()`나 `cancelInteractiveMovement()`를 호출하면 된다. 

유저와의 상호작용 동안에 컬렉션 뷰는 아이템의 현재 위치를 반영하기 위해 레이아웃을 동적으로 비활성화한다. 아무것도 하지않으면 레이아웃을 재배치하지만 원한다면 레이아웃 애니메이션을 정의할 수도 있다. 인터렉션이 끝났을 때 컬렉션뷰는 아이템의 새로운 위치로 data source 객체를 업데이트 한다.

`UICollectionViewController` 클래스는 컬렉션 뷰 내에서 아이템들을 재배치 시킬 수 있도록 gesture recognizer를 제공한다. 이를 설치하기 위해 `installsStandardGestureForInteractiveMovement` 프로퍼티를 `true`로 설정한다.

### Interface Builder attributes

#### items
prototype cell들의 숫자. 이 프로퍼티들은 스토리보드에서 설정하는 구체적인 숫자다. 컬렉션 뷰들을 항상 최소 하나이상의 셀을 가져야하고 다른 타입의 내용을 보여주는 다양한 셀들을 가질 수 있고 같은 내용을 다른 방식으로 보여줄 수도 있다.

#### Layout
사용할 레이아웃 객체. `UICollectionViewFlowLayout` 객체와 커스텀 객체 사이에서 고른다.

flow layout을 선택했을 때 스크롤 방향을 설정할 수 있고 header, footer도 설정할 수 있다. 헤더와 푸터 뷰를 가능하게 하는 것은 재사용가능한 뷰를 추가한다. 물론 이러한 뷰들을 코드로도 만들 수 있다.

커스텀 레이아웃을 고른다면 `UICollectionViewLayout`을 서브클래싱 해야 한다.

Flow layout이 선택되었을 때 컬렉션뷰의 Size inspector는 flow layout metrics를 수정할 수 있는 attribute를 포함하게 된다. 이를 이용해 셀의 사이즈와 헤더, 푸터의 사이즈, 셀 사이의 최소 간격, 섹션 사이의 마진등을 정의한다. 

### Internationalization
컬렉션 뷰는 국제화할 직접적인 내용이 없다. 대신 셀들과 재사용 뷰들을 국제화할 수 있다.

### Accessibility
컬렉션 뷰는 접근가능한 내용을 가지고 있지 않다. 만약 셀들과 재사용가능한 뷰들이 표준 UIKit control을 가진다면 접근가능하게 만들 수 있다. 컬렉션 뷰가 화면의 레이아웃을 변경할 때 `layoutChanged` 알림으로 post할수도 있다.


## URLsession이 비동기적으로 작업을 처리하는 이유

URLSession은 네트워크 통신을 할 때 사용하는 객체이며 비용이 큰 작업을 불러올 때 사용한다. 
사용자입장에서는 작업을 불러올 때 UI가 끊기거나 멈추면 안되기 때문에 다른 스레드에서 비동기적으로 처리하는 것이다. 

### URLSession.swift

우선 `URLSession` 깃허브 소스를 살펴보면 다음과 같은 부분이 있다. `workQueue`라는 프로퍼티에 직렬 큐를 할당해주고 있다.

<img src="https://i.imgur.com/jQNOLYW.png" width="700"/><br/>

그리고 `DataTask`메서드를 호출할 때 이 직렬큐에 async하게 작업을 넣어두고 task를 리턴하게 된다. 비동기적으로 작업을 넣어두고 만들어진 task를 리턴하는 것이다.

<img src="https://i.imgur.com/0FRVLBi.png" width="700"/><br/>

그리고 위에 `let task = URLSessionTask(...)`를 통해 알 수 있는 것은 다음의 사진과 연관지어 보았을 때 DataTask가 이니셜라이저한 부분의 session과 이어진다는 것이다.


### URLSessionTask.swift

<img src="https://i.imgur.com/gqX7hDm.png" width="800"/><br/>

마지막으로 `resume`함수 호출부는 다음과 같다.

<img src="https://i.imgur.com/SQIufKQ.png" width="800"/><br/>

조금 복잡하지만 resume할 때 작업큐를 동기적으로 실행한다는 사실을 알 수 있다.



- [Apple Docs - UICollectionView](https://developer.apple.com/documentation/uikit/uicollectionview)
- [Apple git - URLSession](https://github.com/apple/swift-corelibs-foundation/blob/main/Sources/FoundationNetworking/URLSession/URLSession.swift)

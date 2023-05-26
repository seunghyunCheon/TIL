# 230526 요구사항에 맞는 레이아웃 설계, 프로젝트를 통해 알게 된 Combine, MVVM, Compositional Layout 개념
### 학습내용
- 요구사항에 맞는 레이아웃 설계
- 프로젝트를 통해 알게 된 Combine, MVVM, Compositional Layout 개념

## 요구사항에 맞는 레이아웃 설계

한 화면에 3개의 테이블 뷰가 보여지는 화면을 구현하는 요구사항이 존재했다.

초기에는 하나의 컬렉션 뷰에서 컴포지셔널 레이아웃을 사용한다면 각각의 화면에 접근이 쉬워진다고 생각하여 이 방법을 채택했다.

</br>

### 🔍 문제점
하지만 이를 구현하면서 다음의 문제가 발생했다.
-  section header가 셀 내용과 겹쳐보이는 문제. 
-  각각의 섹션을 listConfiguration을 사용하여 구성했을 때 section의 크기를 잡지 못하는 문제. 
    - 이를 해결하기 위해 각각의 섹션을 list형식이 아니라 compositionalLayout으로 구성했지만 trailingSwipeActionsConfigurationProvider을 추가하지 못하는 문제가 또 발생함.
    -  delete하는 것처럼 보여지는 스크롤뷰를 만들 수는 있으나 애플에서 지원하는 SwipeAction기능을 사용하지 않고 있기에 이때부터 설계가 잘못되었다고 생각.
- 하나의 뷰모델에서 모든 기능을 관리하기에 확장성이 낮아지는 문제. 

<br/>

```swift
 private func collectionViewLayout() -> UICollectionViewLayout {
    let layoutConfiguration = UICollectionViewCompositionalLayoutConfiguration()
    layoutConfiguration.scrollDirection = .horizontal

    let layout = UICollectionViewCompositionalLayout(sectionProvider: { sectionIndex, _ in
        let itemSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                              heightDimension: .fractionalWidth(0.2))
        let item = NSCollectionLayoutItem(layoutSize: itemSize)

        let groupSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1/3),
                                               heightDimension: .fractionalHeight(1))
        let group = NSCollectionLayoutGroup.vertical(layoutSize: groupSize,
                                                       subitems: [item])
        group.edgeSpacing = NSCollectionLayoutEdgeSpacing(leading: nil,
                                                          top: .flexible(50),
                                                          trailing: nil,
                                                          bottom: nil)

        let section = NSCollectionLayoutSection(group: group)
        section.orthogonalScrollingBehavior = .continuous
        // 헤더 설정.
        // deleteAction 커스텀 구현해야 함.

        return section
    }, configuration: layoutConfiguration)

    return layout
}
```

</br>

###  ⚒️ 해결방안

위 문제들을 고려했을 때 하나의 컬렉션뷰에서 관리하기 보다는 3개의 컬렉션 뷰를 만드는 것이 좋다고 생각하여 이 방법으로 변경했다.

변경이후 다음의 장점이 있었다.

- 확장성을 증가시키고 유지보수 용이하게 만들어 개방폐쇄원칙 준수.
- 각각의 뷰가 뷰모델에 1:1관계를 맺고있기 때문에 단일책임원칙을 준수.
- 복잡한 1개의 레이아웃에서 간단한 3개의 레이아웃 구성하여 가독성 증가.

<br/>

```swift
private func collectionViewLayout() -> UICollectionViewLayout {
    let layout = UICollectionViewCompositionalLayout { _, layoutEnvironment in
        var config = UICollectionLayoutListConfiguration(appearance: .grouped)
        // 헤더 설정
        // 애플이 적용하는 UIContextualAction사용하여 Delete 구현.

        let section = NSCollectionLayoutSection.list(using: config, layoutEnvironment: layoutEnvironment)
        section.interGroupSpacing = 10

        return section
    }

    return layout
}
```

</br>


## 프로젝트를 통해 알게 된 Combine, MVVM, sectionProvider 개념


### Combine

#### Publisher
채택한 타입이 시간이 지남에 따라 일련의 값을 전송할 수 있음을 선언하는 프로토콜.
여러가지 구독 메서드를 통해 Subscriber에 자신의 변경사항을 알린다.

**Creating Your Own Publishers**
Publisher 프로토콜을 직접 구현하는 대신 Combine 프레임워크에서 제공하는 여러 타입 중 하나를 사용하여 고유한 Publisher를 만들 수 있다.

* `PassthroughSubject`와 같은 `Subject`의 구체적인 하위 클래스를 사용하여 `send(_:)` 메서드를 호출해 필요에 따라 값을 게시.
* `CurrentValueSubject`를 사용하여 subject의 기본 값을 업데이트할 때마다 값을 게시.
* 커스텀 타입의 속성에 @Published attribute를 추가하여 프로퍼티를 게시.

#### Subscriber
Publisher로부터 input을 전달 받을 수 있는 타입을 선언하는 프로토콜.
Subscriber 인스턴스는 Publisher로부터 스트림의 요소를 전달 받는다.

#### Subject
외부 호출자가 요소를 게시할 수 있는 메서드를 노출하는 Publisher.
`send(_:)` 를 통해 스트림에 어떤 값을 주입할 수 있다.


### Property Wrapper
#### @Published
해당 프로퍼티 래퍼 attribute로 표시된 프로퍼티는 타입이 publish하게 된다.
* 게시된 프로퍼티의 publisher에는 프로퍼티 이름 앞에 $표시를 추가하여 접근할 수 있다.
* @Published 프로퍼티 래퍼는 class 프로퍼티에만 적용할 수 있다.

<br/>

### MVVM 

#### Model
* 비즈니스 데이터를 가지고 있는 계층. 
* Repository에 데이터를 CRUD하는 로직이 존재.  

#### View
* 뷰모델과 연결되는 바인딩이 존재.
* 그 외 레이아웃을 그리는 코드만 존재.
    
#### ViewModel
* 데이터 바인딩 대상을 제공. 모델을 직접 노출하거나 특정 모델 멤버를 래핑하는 멤버를 제공.
* UIKit를 import하지않고 뷰에게 바인딩해주는 모델과 Presentation Logic만 존재.
    
 <br/>
 
### Compositional Layout

#### sectionProvider
* Section안의 요소 간 간격을 주기 위해 `UICollectionViewCompositionalLayout.list`사용이 아닌 `sectionProvider`적용.
#### layout.scrollDirection
* 컬렉션 뷰 레이아웃이 스크롤되는 축을 결정하는 속성.
#### section.orthogonalScrollingBehavior
* 현재 레이아웃방향의 수직방향으로 스크롤 스타일을 주는 속성.
</details>

---

</br>

### 참고문서
- [WWDC - Combine in Practice](https://developer.apple.com/videos/play/wwdc2019/721/)

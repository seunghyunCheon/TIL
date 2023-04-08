230404 git rebase, clipsToBounds, adjustFontSizeToFitWidth, Compositional layout self-sizing한 셀 만들기
===
학습내용
- git rebase
- clipsToBounds
- adjustFontSizeToFitWidth
- Compositional layout self-sizing한 셀 만들기
## git rebase
> 두 개의 공통 Base를 가진 Branch에서 한 Branch의 Base를 다른 Branch의 커밋으로 branch의 base를 옮기는 작업이다.

A브랜치의 작업사항들을 B브랜치에 추가해야 하는 상황이기 때문에 git fork를 이용해서 rebase를 진행했다. 

B브랜치에 추가해야 하는 경우 rebase on A를 사용해야 B브랜치에 A브랜치의 작업내용들을 추가할 수 있게된다. 

## clipsToBounds
> subview들이 view의 bound에 제한이 될지 말지를 결정하는 Bool 값이다.

이 값이 true라면 서브뷰들이 뷰의 bound 내로 잘리게 된다. 그렇지 않으면 서브뷰들의 프레임이 뷰의 bound를 넘어 잘리지 않게 된다.

이미지 뷰를 오토레이아웃해서 설정해줬는데 정해준 width를 넘어가는 일이 발생했었다. 처음에는 이미지의 `contentMode`문제인줄 알았는데 clipsToBound가 기본값이 false인 게 이유였다. 
`contentMode`의 `scaleAspectFill`은 비율을 유지하며 화면에 꽉 채우려고 하기 때문에 정해준 이미지 뷰의 크기보다 클 가능성이 있다. 이러한 상황에서 `clipsToBound`가 false라면 뷰의 bound를 넘어가게 되어 이미지 뷰보다 크게 보여진다.(hierachy에서는 이미지 뷰의 사이즈로 나온다.) 

그래서 이를 true로 설정해주면 뷰의 bound내로 설정되어 이미지 뷰만큼 이미지가 보여진다.

## adjustFontSizeToFitWidth
> 레이블이 텍스트를 레이블의 bounding rectangle에 맞추기 위해 폰트 사이즈를 줄일지 말지를 결정하는 Bool 값이다.

기본적으로 레이블은 명시한 `font` 프로퍼티에 의해 텍스트를 그린다. 만약 이 프로퍼티르 `true`이고 `text`프로퍼티의 텍스트가 레이블의 bounding rectangle을 넘어간다면 레이블은 텍스트가 맞춰질 때까지 최소 크기까지 줄인다. 기본값은 `false`이다. 

만약 `true`로 변경하면 `minimumScaleFactor`를 수정함으로써 적절한 최소 폰트 크기를 설정해야 한다. 자동으로 축소하는 건 오직 싱글라인 레이블에서만 된다. 

minimumScaleFactor를 0.3으로 설정할 경우 텍스트의 크기가 30%까지 축소가 된다. 

## Compositional layout self-sizing한 셀 만들기
애플에서 제공하는 기본적인 Compositional layout을 사용하면 자동으로 self-sizing한 셀을 구성할 수 있다. 
```swift
let configure = UICollectionLayoutListConfiguration(appearance: .plain)
let layout = UICollectionViewCompositionalLayout.list(using: configure)
collectionView = UICollectionView(
    frame: view.bounds,
    collectionViewLayout: createLayout(for: layout)
)
```

하지만 커스텀으로 레이아웃을 구성하는 경우에는 self-sizing을 지원하지 않기 때문에 레이아웃을 손봐줘야 한다.

처음 시도는 다음과 같았다.

```swift
let itemSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                      heightDimension: .fractionalHeight(1.0))
let item = NSCollectionLayoutItem(layoutSize: itemSize)

let groupSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                       heightDimension: .estimated(100))
let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize,
                                                 subitems: [item])
```

item의 height는 그룹에 의존하게 되고 그룹에서 self-sizing하게 만들기위해 estimated를 주었는데 dynamic type을 적용한 결과 셀의 높이가 증가하지 않았다. 
이는 item의 height가 .fractionalHeight이기 때문에 높이가 동적으로 변경되더라도 item의 높이가 변하지 않기 때문이다. 이를 해결하기 위해 item도 estimated를 줘야 한다.

```swift
let itemSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                      heightDimension: .estimated(100))
let item = NSCollectionLayoutItem(layoutSize: itemSize)

let groupSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                       heightDimension: .estimated(100))
let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize,
                                                 subitems: [item])
```

group도 .estimated를 주는 이유는 아이템에따라 그룹도 변동이 될 수 있기 때문이다. 만약 그룹에 .fractionalHeight(1.0)을 주게되면 상위 섹션이 컬렉션 뷰의 전체이기 때문에 그룹 하나 당 컬렉션 뷰의 height를 갖게 될 것이다.


### 참고문서
- [Apple Docs - clipsToBounds](https://developer.apple.com/documentation/uikit/uiview/1622415-clipstobounds)
- [Apple Docs - adjustsFontSizeToFitWidth](https://developer.apple.com/documentation/uikit/uilabel/1620546-adjustsfontsizetofitwidth)


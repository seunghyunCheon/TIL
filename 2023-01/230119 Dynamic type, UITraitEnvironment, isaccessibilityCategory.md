230119 Dynamic type, UITraitEnvironment, isaccessibilityCategory
===
TIL (Today I Learned)
===
학습내용
---
- 코드로 오토레이아웃 구현하기2
- Dynamic type 적용하기
## 고민한 점 / 해결 방법

## 코드로 오토레이아웃 구현2
### ScrollView + Stackview
- scrollview안에 일반view대신 stackview를 넣을 수도 있다. 
```swift
 @IBAction func addLabel() {
    let label = UILabel()
    label.font = UIFont.preferredFont(forTextStyle: .largeTitle)
    label.text = """
        aaaaaaaaa
        bbbbbbbbb
        ccccccccc
        ddddddddd
        eeeeeeeee
        """
    label.numberOfLines = 0
    label.isHidden = true
    label.adjustsFontForContentSizeCategory = true
    stackView.addArrangedSubview(label)
    UIView.animate(withDuration: 0.3) {
        label.isHidden = false
    }
}
```
- 위 코드는 스택뷰에 레이블을 넣는 방식이다. (여러 줄을 보여지게 하려면 numberOfLines를 0으로 해야 함)
## Dynamic Type
> 사용자가 기기에 설정한 텍스트 크기에 따라 앱에서 이를 적용하여 나타내는 타입. 접근성을 높이기 위해 사용한다.
- 바로 위 예제에서 `label.adjustsFontForContentSizeCategory = true`코드는 사용자 기기의 content size변경에 따라 객체를 업데이트 할 것인지를 Bool타입으로 결정하는 코드이다. 스토리보드에서 작업할 때는 다음의 위치에 해당한다.
<img src="https://i.imgur.com/jtKkG0j.png" width="300"/>
- 다이나믹 타입은 시스템이 설정한 폰트에서만 지원이 된다.(body, Headline, caption과 같은) 아래 사진의 System과 System Italic은 지원이 안되니 유의하자
<img src="https://i.imgur.com/krhSnBi.png" width="300"/>

#### Accessabilty 확인하는 법
- xcode -> open Developer tool -> Accessability inspector
<img src="https://i.imgur.com/ALfW1mT.png"/>
- 이후 오른쪽 상단의 설정에 들어가서 Dynamic type을 적용할 수 있다.

#### 그런데 새로운 궁금증이 생겼다. Font만 변경되면 그 레이블을 담고 있는 superview는 변하지 않는 것이 아닐까?
<img src="https://i.imgur.com/MQISa91.png" width="300"/>

예상대로 기기 폰트를 크게하고 앱을 작동시킬 경우 Superview영역을 넘어 잘리게 된다. 그래서 다이나믹 타입을 적용하는 프로젝트는 어떻게 레이아웃을 적용하는지 찾아봤다.(아래 Apps with Dynamic Type Example에서 소개). 이를 보기 전에 isAccessibility속성에 대해서 알아야 한다.

#### isAccessibilityCategory
> content size category가 접근성과 연관되어있는지를 가리키는 Bool값
<img src="https://i.imgur.com/7l893Ap.png" width="700" height="70"/>
- 위 12개의 속성 중 앞에 7개는 false가, 뒤 5개는 true가 나온다. 이를 이용해서 레이아웃에 대한 분기를 처리할 수 있다.
- 이 속성은 `traitCollection.preferredContentSizeCategory`의 하위 프로퍼티이다.
- UIView Controller와 UIView는 UITraitEnvironment를 채택하고 있기 때문에 traitCollection이라는 속성을 갖게 된다. 

#### UITraitEnvironment는 뭐지?
> iOS 인터페이스 환경을 앱에서 사용할 수 있도록 하는 일련의 방법들
```swift
@MainActor protocol UITraitEnvironment
```
iOS 인터페이스 환경은 (horizontal, vertical) size class, display scale, user interface idiom을 포함한다. 이 환경들에 접근하기 위해서는 이 프로토콜을 채택하고 trait Collection property를 사용하면 된다.(더 깊게는 나중에 알아봐야 겠다)

- 결론적으로 `traitCollection.preferredContentSizeCategory.isAccessibilityCategory`에 접근해서 true일 때와 false일 때를 나눠서 다른 제약조건을 넣어주면 된다.

#### Apps with Dynamic Type Example
```swift
var largeFontTypeLayout: [NSLayoutConstraint]?
var standartFontTypeLayout: [NSLayoutConstraint]?

largeFontTypeLayout = [
    second.firstBaselineAnchor.constraint(equalToSystemSpacingBelow: first.lastBaselineAnchor, multiplier: 1),
    second.leadingAnchor.constraint(equalTo: first.leadingAnchor)
]

standartFontTypeLayout = [
    second.centerYAnchor.constraint(equalTo: first.centerYAnchor), 
    second.leadingAnchor.constraint(equalTo: first.trailingAnchor, constant: 10)
]

if traitCollection.preferredContentSizeCategory.isAccessibilityCategory {
    largeFontTypeLayout?.forEach {
        $0.isActive = true
    }
} else {
    standartFontTypeLayout?.forEach {
        $0.isActive = true
    }
}
```
- 하지만 앱을 백그라운드로 보내고 기기의 폰트 크기를 바꾸게 되면 대응을 못하게 된다. 이 때문에 기기 폰트 설정이 변경되는 것에 반응하기 위해 notification을 달아준다.
```swift
override func viewDidLoad() {
    super.viewDidLoad()
    NotificationCenter.default.addObserver(self, selector: #selector(setLayoutByDynamicType), name: UIContentSizeCategory.didChangeNotification, object: nil)
}

// setLayoutByDynamicType
@objc func setLayoutByDynamicType() {
    if traitCollection.preferredContentSizeCategory.isAccessibilityCategory {
        standartFontTypeLayout?.forEach {
            $0.isActive = false
        }
        LargeFontTypeLayout?.forEach {
            $0.isActive = true
        }    
    } else {
        LargeFontTypeLayout?.forEach {
            $0.isActive = false
        }
        standartFontTypeLayout?.forEach {
            $0.isActive = true
        }
    }
}
```
- UIContentCategory의 didChangeNotification을 관찰하면서 변경이 되었을 때 제약을 활성화시키도록 만들었다.


### 결론
- 오토레이아웃을 사용하다보니 width와 height를 직접 기입하지 않고 웬만한 레이아웃 구현이 가능하다고 느꼈다. 기준이 되는 요소를 잡아서 그 요소의 비율대로 맞추는 것도 하나의 방법인 것 같다.
- 꼭 사용하고자 하는 폰트가 존재하는 것이 아니라면 Dynamic type을 지원하는 시스템 폰트를 사용하자. `isAccessibilityCategory`속성을 이용해서 텍스트 크기별로 레이아웃 요소들도 배치할 수 있다.(나중에 꼭 적용해봐야겠다.)
- stackView의 fill은 강력하다. subview들의 width혹은 height를 stackview에 맞게 함으로써 동적인 레이아웃을 만들 수 있게 한다.



###### tags: `auto layout`, `dynamic type`, `isAccessibilityCategory` `UITraitEnvironment`


### Reference
- [야곰닷넷 - Auto Layout](https://yagom.net/courses/autolayout/)
- [Dynamic Type Adaptable Layouts](https://www.blog.kevin-hirsch.com/dynamic-type-adaptable-layouts)
- [Dynamic Type에 따른 AutoLayout 조정하기](https://leechamin.tistory.com/553)
- [Apple Developer Documentation - isAccessibilityCategory](https://developer.apple.com/documentation/uikit/uicontentsizecategory/2897444-isaccessibilitycategory)
- [Apple Developer Documentation - UITraitEnvironment](https://developer.apple.com/documentation/uikit/uitraitenvironment)

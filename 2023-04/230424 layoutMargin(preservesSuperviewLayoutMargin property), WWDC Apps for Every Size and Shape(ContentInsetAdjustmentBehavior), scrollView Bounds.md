## 230424 layoutMargin(preservesSuperviewLayoutMargin property), WWDC Apps for Every Size and Shape(ContentInsetAdjustmentBehavior), scrollView Bounds 
===

학습내용
- layoutMargin
- WWDC Apps for Every Size and Shape 

## layoutMargin
뷰 안의 컨텐츠를 그릴때 사용하는 기본 spacing

```swift
var layoutMargins: UIEdgeInsets: { get set }
```

### Discussion
iOS 11이후로 `directionalLayoutMargin` 프로퍼티를 사용해서 이 값대신에 레이아웃 마진을 명시한다. leading과 trailing edge inset이 `directionalLayoutMargin` 프로퍼티를 통해 동기화가 된다. (RTL적용이 됨)

뷰 컨트롤러의 루트 뷰인 경우 이 프로퍼티의 기본값은 system 최소 margin과 safe area inset을 반영한다. 뷰 계층안의 다른 서브뷰들은 기본적으로 8포인트지만 만약 뷰가 safeArea를 벗어나거나 `preservesSuperviewLayoutMargins` 프로퍼티를 `true`로한다면 더 커질 수 있다. (`preservesSuperviewLayoutMargin`은 현재 뷰가 슈퍼뷰의 마진을 반영할지 말지를 결정하는 Bool 값이다.)

두 가지 경우를 실험을 통해 확인해보자.

### 1️⃣ 뷰가 safeArea를 벗어난 경우

먼저 루트 뷰 내의 서브뷰가 safeArea를 벗어나지 않는다면 layoutMargin이 기본값인 8로 설정된다.

```swift
class MyView: UIView {
    override func layoutSubviews() {
        super.layoutSubviews()
        print("Layout margins: \(layoutMargins)")
        print("Safe area insets: \(safeAreaInsets)")
    }
}

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let myView = MyView()
        myView.backgroundColor = .red
        myView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(myView)

        let safeAreaGuide = view.safeAreaLayoutGuide
        NSLayoutConstraint.activate([
            myView.topAnchor.constraint(equalTo: safeAreaGuide.topAnchor),
            myView.leadingAnchor.constraint(equalTo: safeAreaGuide.leadingAnchor),
            myView.trailingAnchor.constraint(equalTo: safeAreaGuide.trailingAnchor),
            myView.bottomAnchor.constraint(equalTo: safeAreaGuide.bottomAnchor)
        ])
    }
}
```

<img src="https://i.imgur.com/D5jgSsu.png" width="500"/><br/>

하지만 `myView`를 `safeAreaLayoutGuide`가 아닌 view에 맞추게 된다면 safeArea를 벗어나게 되어 layoutMargin값과 safeAreaInset값이 기본적으로 가지는 값인 8을 넘어서게 된다.

```swift
// 부분 변경
NSLayoutConstraint.activate([
    myView.topAnchor.constraint(equalTo: view.topAnchor),
    myView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
    myView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
    myView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
])
```

<img src="https://i.imgur.com/4iHGsPm.png" width="500"/><br/>
제약을 view에 맞추게되면서 layoutMargin과 safeAreaInset은 증가하게 되었지만 시뮬레이터에는 safeArea를 침범하게 되었다. 
부모뷰의 top, leading, bottom, trailing에만 맞추었고 margin은 신경쓰지 않았기 때문이다. 
즉 safeArea가 침범당하면서 루트뷰의 layoutMargin값이 safeAreaInset과 동일하게 변경되었지만 myView 제약을 설정할 때 margin만큼 constant를 주지않았기 때문에 꽉 차있는 것이다.

<img src="https://i.imgur.com/rzR9gzn.png" width="300"/><br/>

따라서 다음과 같이 변경해주었다.
```swift
NSLayoutConstraint.activate([
    myView.topAnchor.constraint(equalTo: view.topAnchor, constant: view.layoutMargins.top),
    myView.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: view.layoutMargins.left),
    myView.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -view.layoutMargins.right),
    myView.bottomAnchor.constraint(equalTo: view.bottomAnchor, constant: -view.layoutMargins.bottom)
])
```

하지만 결과가 동일했다.
왜 layoutMargin에 해당하는 constant를 주었는데 여전히 동일할까?

해답은 뷰 컨트롤러의 생명주기에 있다. 

`viewDidLoad`는 뷰가 메모리에 로드된 상태에 호출되는 메서드이다. 이는 메모리에 올라갔을 뿐이지 뷰가 화면에 보여지진 않은 상태이고 safeAreaLayoutGuide가 적용되기 전의 상태이다. 

즉 `viewDidLoad`도 `viewWillAppear`도 아닌 `viewDidAppaer`에서 마진값을 적용시켜줘야 하는 것이다. 

```swift
override func viewDidAppear(_ animated: Bool) {
    myView.backgroundColor = .red
    myView.translatesAutoresizingMaskIntoConstraints = false
    view.addSubview(myView)

    NSLayoutConstraint.activate([
        myView.topAnchor.constraint(equalTo: view.topAnchor, constant: view.layoutMargins.top),
        myView.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: view.layoutMargins.left),
        myView.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -view.layoutMargins.right),
        myView.bottomAnchor.constraint(equalTo: view.bottomAnchor, constant: -view.layoutMargins.bottom)
    ])
}
```

#### 그럼 `layoutMargin`을 적용시키려면 전부 `viewDidAppear`에서 적용시켜야 하는걸까?
그렇지는 않다.

`viewDidload`는 `safeAreaLayoutGuide`를 잡지 못하는 상태이다. 기본적으로 8point로 갖고있는 `layoutMargin`은 적용시킬 수가 있다. 
즉 안전영역을 침범하는 경우만 유의한다면 서브뷰들에게도 제약조건 뿐만 아니라 `layoutMargin`을 적용시켜 UI를 구성할 수 있다.
그리고 안전영역을 침범하면 안되는 UI라면 `viewDidload`에서 `safeAreaGuide`를 사용해서 제약을 설정해줄 수도 있다. 

```swift
NSLayoutConstraint.activate([
    myView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
    myView.leadingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.leadingAnchor),
    myView.trailingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.trailingAnchor),
    myView.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor)
])

let myView2 = MyView()
myView2.backgroundColor = .yellow
myView2.translatesAutoresizingMaskIntoConstraints = false

myView.addSubview(myView2)

NSLayoutConstraint.activate([
    myView2.topAnchor.constraint(equalTo: myView.topAnchor, constant: myView.layoutMargins.top),
    myView2.leadingAnchor.constraint(equalTo: myView.leadingAnchor, constant: myView.layoutMargins.left),
    myView2.trailingAnchor.constraint(equalTo: myView.trailingAnchor, constant: -myView.layoutMargins.right),
    myView2.bottomAnchor.constraint(equalTo: myView.bottomAnchor, constant: -myView.layoutMargins.bottom)
])
```

### 2️⃣ preservesSuperviewLayoutMargin 프로퍼티를 true로 설정했을 때
> 현재 뷰가 슈퍼뷰의 마진을 반영할지 말지를 결정하는 Bool값

이 속성값을 true로 주더라도 현재 뷰의 가장자리와 슈퍼 뷰 사이의 거리가 슈퍼뷰의 layoutMargin보다 작은 레이아웃경우에 영향을 준다.

이 값이 true라면 부모뷰의 layoutMargin을 변경했을 때 자식뷰의 layoutMargin이 동일하게 설정되기 때문에 서브뷰들을 담은 컨테이너를 관리할 때 사용하면 좋을 것 같다.


### 결론
`layoutMargin`은 UI를 코드로 작성할 때 일괄적으로 설정, 수정할 필요가 있을때 사용하면 좋을 것 같다는 생각이 들었다. 
그리고 `stackView`에서는 `conetntInset`이 아닌 `layoutMargins`를 사용해야 inset을 줄 수 있으니 스택뷰를 사용할 때 적용해봐야겠다.

## WWDC Apps for Every Size and Shape 
`layoutMargin` 대신에 `directionalLayoutMargin`을 사용해서 RTL언어에도 적용되도록 한다.

### ScrollView
스크롤 뷰는 `contentOffset`을 이용해서 현재 스크롤 좌표를 표현한다.

`contentOffset`은 contentView의 origin이 스크롤 뷰의 origin에서 offset지점을 의미한다. 
즉 스크롤 뷰는 위아래로 스크롤이 가능한데 아래방향으로 100만큼 이동했다면 그 좌표는 (0, 100)이 되고 이는 `contentOffset`을 의미하게 된다.

`contentInset`은 `contentView`의 가장자리와 `content` 사이의 공간을 확장시키는 프로퍼티이다. 

말로만 들으면 이해가 잘 안 된다. 직접사용해서 조금 더 알아보자 

초기에는 기본적인 테이블 뷰를 작성한 상태이다.

<img src="https://i.imgur.com/teISFbG.png" width="300"/><br/>

`contentInset`을 조작했을 때의 화면은 다음과 같다.
```swift
 self.tableView.contentInset.top = 100
```

<img src="https://i.imgur.com/Fq7Lo6o.png" width="300"/><br/>

그리고 테이블 뷰의 bound를 살펴보면 y가 -100인 것을 볼 수 있다.
<img src="https://i.imgur.com/iswaGj7.png" width="300"/><br/>

처음에는 `contentView`가 밑으로 갔기에 y가 100이 되어야 하는 게 아닌가 싶었다.
그러다 bounds에 대해서 조금 더 찾아봤다.

bounds는 자신의 좌표시스템에서 뷰의 위치와 사이즈를 설명하는 사각형범위이다.

`contentInset.top`을 조정하기 전의 테이블 뷰(스크롤 뷰라고 부르겠음)의 bounds의 x,y좌표는 (0, 0)이다. 즉 스크롤 뷰는 (0, 0)좌표에서 내용을 그린다.(frame과는 관련이 없다)

하지만 `contentInset.top`을 100만큼 준다면 스크롤 뷰가 시작되는 위치가 생긴 여백만큼 올라가기 때문에 자신의 좌표시스템에서 뷰의 위치가 위로 올라가게 되어 음수가 된다. 즉 bounds가 (0, -100)이 된다. 

이는 스크롤 뷰를 위에서 아래로 드래그했을 때 숨겨져있던 위의 내용들을 보게 되는데 위 방향이 음수라는 사실을 알면 조금 이해하기 편할 것 같다. 
contentOffset도 마찬가지로 -100이 된다. (스크롤 뷰가 위로 올라갔기 때문).

만약 navigationBar나 tabBar가 존재한다면 이 `contentInset`을 이용해서 안전영역에 침범하지 못하게 할 수 있다. 

하지만 iOS11이후로 `UIScrollViewController`에 읽기전용인 `adjustedContentInset` 프로퍼티가 생겼고 이 프로퍼티를 통해 안전영역 침범을 막을 수 있다.(오히려 직접 주지 않아도 되기에 적극 사용하는 게 좋아보인다)

이는 `adjustContentInset = contentInset + systemInset`으로 `contentInset`은 직접 정의하는 것이고 `system inset`은 안전영역과 그 이상의 것이다. `adjustContentInset`을 설정하려면 `ContentInsetAdjustmentBehavior`을 이용해서 조작해줘야 한다. (추가적으로 `contentInset`을 설정하면 추가적인 여백을 줄 수 있다.)

### ContentInsetAdjustmentBehavior
말 그대로 contentInset을 조정하는 행동으로 정의해주는 프로퍼티이다.
enum으로 정의되어있으며 `automatic`, `scrollableAxes`, `never`, `always`가 존재한다. 

기본값이 `automatic`인데 이는 `systemInset`을 고려해서 적용하기 때문에 안전영역이 침범되지 않는다. 
하지만 `automatic`의 경우 scrollable한 상태만큼 content가 꽉차지않으면 safeAreaInset을 주지않기 때문에 이 경우에는 `always`속성을 사용한다.



### 참고문서
- [Apple Docs - layoutMargin](https://developer.apple.com/documentation/uikit/uiview/1622566-layoutmargins)
- [Apple Docs - preserveSuperviewLayoutMargin](https://developer.apple.com/documentation/uikit/uiview/1622653-preservessuperviewlayoutmargins)
- [Apple Docs - Bounds](https://developer.apple.com/documentation/uikit/uiview/1622580-bounds)
- [WWDC Notes](https://www.wwdcnotes.com/notes/wwdc18/235/)
- [let us go - layoutmargin](https://academy.realm.io/kr/posts/ios-layoutmargins/)
- [layout margin이란?](https://woozzang.tistory.com/147)
- [UIStackView Margin적용하기](https://velog.io/@dvhuni/UIStackView-Margin-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0)
- [adjustedContentInset / contentInsetAdjustmentBehavior](https://zeddios.tistory.com/809)
- [소들이 블로그- Bounds](https://babbab2.tistory.com/45)

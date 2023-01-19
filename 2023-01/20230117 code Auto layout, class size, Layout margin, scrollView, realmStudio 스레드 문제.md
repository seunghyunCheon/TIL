20230117 code Auto layout, class size, Layout margin, scrollView, realmStudio 스레드 문제
===
TIL (Today I Learned)
===
학습내용
---
- 코드로 오토레이아웃 구현하기
- realmStudio 스레드 문제
## 고민한 점 / 해결 방법

## 코드로 오토레이아웃 구현
### Anchor방법
#### 01 view를 만들고 속성값을 지정해준다.
```swift
let button = UIButton()
button.setTitle("Button", for: .normal)
button.tintColor = .white
button.backgroundColor = .systemGreen
view.addSubview(button)
```
#### 02 UIView가 기존의 autoresizing하게 배치되는 제약을 풀어준다.
```swift
button.translatesAutoresizingMaskIntoConstraints = false
```
#### 03 safeArea를 중점으로 제약사항을 설정할 것이기 때문에 safeArea를 변수로 만든다.
```swift
let safeArea = view.safeAreaLayoutGuide
```
#### 04 제약사항을 걸어준다. `isActive = true`를 통해 오토레이아웃 제약사항을 활성화한다. 
```swift
button.leadingAnchor.constraint(equalTo: safeArea.leadingAnchor, constant: 16).isActive = true
button.trailingAnchor.constraint(equalTo: safeArea.trailingAnchor, constant: -16).isActive = true

// 1
let safeBottomAnchor = button.bottomAnchor.constraint(equalTo: safeArea.bottomAnchor)
safeBottomAnchor.isActive = true
safeBottomAnchor.priority = .defaultHigh

// 2
button.bottomAnchor.constraint(lessThanOrEqualTo: view.bottomAnchor, constant: -20).isActive = true
```
- 우선순위를 2번제약보다 낮추면 2번이 먼저실행되고 이 safeBottomAnchor가 실행된다. 즉 2번의 view의 bottom - 20보다 작은 제약중에서 safeArea의 bottom을 갖는다는 뜻
- button의 bottomAnchor가 view의 bottomAnchor의 -20한 값보다 작거나 같아야 한다.(safeArea가 존재하지 않는 SE기종같은 경우에 사용)

### NSLayoutConstraint방법
- 직접 NSLayoutConstraint인스턴스를 생성시키는 방법이다. Anchor방법으로는 구현하기 힘든 세부사항을 고려할 때 사용하면 좋다.
- 하지만 Anchor는 제네릭을 지원하기 때문에 잘못된 제약사항을 줬을 때 컴파일에러가 발생하는 반면 NSLayoutContraint방법은 제네릭을 지원하지않아서 컴파일에러가 발생하지 않는다. 그래서 잘못된 제약사항을 줘도 돌아가는 단점이 있다.
```swift

let bottomView = NSLayoutConstraint(item: button,
                                    attribute: .bottom,
                                    relatedBy: .lessThanOrEqual,
                                    toItem: view,
                                    attribute: .bottom,
                                    multiplier: 1,
                                    constant: -20)
```

### Dynamic Stackview
- 추가버튼을 눌러 새로운 뷰가 쌓일때 애니메이션 효과를 보여주게 하는 방법
```swift
@objc func addView() {
    let view = UIView()
    view.backgroundColor = .black
    view.isHidden = true
    verticalStackView.addArrangedSubview(view)

    UIView.animate(withDuration: 0.3) {
        view.isHidden = false
    }
}
```
- 추가한 후 숨겨놓은 뒤 0.3초 딜레이동안 숨겨놓은 걸 풀어준다.
- 반대로 삭제하는 방법
```swift
@objc func removeView() {
    guard let last = verticalStackView.arrangedSubviews.last else {
        return
    }
    UIView.animate(withDuration: 0.3) {
        last.isHidden = true
    } completion: { _ in
        self.verticalStackView.removeArrangedSubview(last)
    }
}
```
- 삭제하고 딜레이를 거는 것은 아무효과를 보지못하기 때문에 먼저 요소를 눈에서 가린 뒤에 remove하는 코드를 넣어준다.

### size class
> 기기별로 element의 크기를 설정할 수 있게 만드는 기능
- iOS 기기의 폭과 높이를 compact와 regular로 표시
- iPhone12는 wChR로 Compact width, regular height
- 보통은 아이폰앱과 아이패드앱을 구분하여 개발하는 경우가 많아서 현업에서도 많이 쓰이지 않는다고 한다. 

### Layout margin
> view안에 컨텐츠와의 기본 margin
```swift
var layoutMargins: UIEdgeInsets { get set }
```
- safeArea랑은 다른 개념으로 view내부에 기본적으로 갖는 마진이다.
<img src="https://i.imgur.com/dDUb2Ui.png" width="200"/>

- UIView를 스토리보드에서 만들고 View를 기준으로 leading, trailing, top을 0으로 준 뒤 constarint margin을 체크하면 다음의 화면처럼 배치가 된다.
<img src="https://i.imgur.com/0kapg7B.png" width="300"/>
- view자체에는 layout margin이 존재하기 때문
<img src="https://i.imgur.com/cbcYOAf.png" width="300"/>

- 추가적으로 파란뷰의 layoutMargin을 leading과 trailing을 20을 주고 파란뷰안에 초록색 뷰를 넣을 때 layout constraint를 체크한다면 다음과 같이 화면에 보여진다.

<img src="https://i.imgur.com/XNXk33h.png" width="300"/>


### ScrollView 오토레이아웃 적용

- 01 스크롤뷰를 scene에 추가한다
- 02 오토레이아웃을 적용한다. (음..오토레이아웃 설정해도 빨간 줄이 보인다. 일단 임의로 늘려놓자)<br>
<img src="https://i.imgur.com/7190udn.png" width="300" height="600"/>
- 03 content View라는 이름으로 view를 scrollView에 추가한다.<br>
<img src="https://i.imgur.com/BSYtkSV.png" width="300" height="600"/>
- 04 content view의 엣지를 스크롤뷰의 엣지에 맞춘다. content view는 스크롤뷰에서 보여지는 영역이 될 것이다.
- 05 스크롤 뷰가 위아래로만 스크롤할 수 있게하려면 content view의 width를 scroll view와 동일하게, 만약 옆으로 하려면 height를 동일하게 준다.(width를 동일하게 줬더니 content view의 가로가 꽉 찼다)<br>
<img src="https://i.imgur.com/q2RBPH8.png" width="300" height="600"/>
- 06 하지만 아직도 빨간줄이 사라지지 않는다. 이는 scroll view가 움직여야 하는 전체 사이즈는 content의 사이즈이기 때문이다. content size내부에는 어떠한 content도 담기지 않았기 때문에 scroll view에서 제약사항이 잡히지 않는 것이다. 그래서 UILabel이나 여러 객체들을 content view에 넣게되면 문제가 해결된다.


#### Content Layout Guide, Frame Layout Guide
- content layout은 contents가 커지면 계속해서 커질 수 있고 Frame Layout은 contents증가와는 무관하게 고정된 값을 가진다. 이러한 이유로 스크롤 시에 어떤 값은 스크롤되기를 원하지 않는다면 Frame Layout Guide에 오토레이아웃 제약을 걸어주면 고정된 값으로 줄 수 있다.


## RealmStudio 스레드 문제
- 데이터확인을 위해서 realmSwift파일을 열어놓은 상태로 xcode를 실행했더니 다음의 오류가 발생했다.
<img src="https://i.imgur.com/6a79bFt.png" width="600"/>
- Realm파일은 접근 공유를 할 수 없는데 다른 프로세스에서 열려있기 때문에 발생하는 문제였다. 그래서 realmSwift를 키지않은 상태로 앱을 구동한 후 realm을 이용해 데이터 CRUD를 하고 앱을 종료시킨 뒤 데이터가 제대로 들어왔는지 확인해야 했다. 
#### 음.. 그럼 앱 전역에서 realm을 사용하고 있다면 앱 구동하는 동안 realm Swift를 절대 못 쓰는건가?
- realm을 전역으로 사용하는 것이 아닌 함수 내부에서 지역변수로 설정해 함수호출이 끝나면 realm이 존재하지 않도록 설정했다.
```swift
func okButtonTapped() {
    let realm = try! Realm()
        
    let myGoalAmount = GoalAmount(date: Date(), goalAmount: goalAmount)
    let user = User(nickName: userName, notificationCycle: notification)

    print(user.goalAmounts)
    user.goalAmounts.append(myGoalAmount)

    print(user)
    try! realm.write {
        realm.add(user)
    }
    
    let tabBarvc = TabBarController()
    navigationController?.pushViewController(tabBarvc, animated: true)
}
```
- realm에 데이터를 삽입하고 tabBar로 이동하면서 okButtonTapped는 더이상 realm객체를 참조하고 있지 않게 되었다. 결과는 탭바로 이동한 후에 realmSwift파일을 열어보니 crash나지않고 잘 열리는 것을 볼 수 있었다.
- 하지만 realm을 사용하는 앱 서비스에서는 상시로 사용하기 때문에 보통 전역으로 선언해야 하고 이는 앱을 종료한 후에 테스트를 해야 한다는 뜻과 같다.





###### tags: `code Auto layout`, `class size`, `Layout margin` `scrollView`


### Reference
- [야곰닷넷 - Auto Layout](https://yagom.net/courses/autolayout/)

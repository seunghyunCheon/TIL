230111 Auto layout, ARC(Automatic Reference Counting)
===
TIL (Today I Learned)
===
학습내용
---
- UIViewController

## 고민한 점 / 해결 방법

## UIViewController
> UIKit App의 뷰 계층을 관리하는 객체
```swift
@MainActor class UIViewController: UIResponder
```
UIViewController클래스는 모든 view controller의 공통적인 부분을 정의한다. 보통 UIViewController를 subclass해서 메서드와 프로퍼티를 추가해 사용한다.

#### viewController의 주 책임
- 데이터 변화에 따라 뷰의 내용을 업데이트
- 뷰를 통한 유저의 상호작용에 대한 반응
- 전반적인 인터페이스 레이아웃을 관리하거나 view의 사이즈 관리
- 앱 내의 다른 객체들과의 협력

viewController는 자신이 관리하는 view와 단단히 바인딩 되어있으며, UIResponder 객체로써 responder chain에 삽입이 된다. 이 때문에 만약 viewController에서 이벤트를 처리하지 않는다면 상위 뷰로 전달할 수 있는 옵션을 제공한다.

일반적으로 하나의 viewController에는 하나의 뷰만 표시가 된다.

#### 뷰 관리
- 각 viewController는 view의 계층 구조를 관리하며 루트 뷰는 view 프로퍼티에 저장된다.
- 루트 뷰의 크기와 위치는 해당 view를 소유한 객체(상위 view Controller, 앱의 window)에 의해 결정된다. window가 소유하고 있는 viewController는 앱의 루트 뷰 컨트롤러이며 view의 크기는 window를 채울 수 있는 정도로 조정된다.

ViewController는 소유한 view를 곧바로 load하지 않는다. view는 처음 액세스할때 로드되거나 생성이 된다. viewController의 view를 지정하는 방법에는 여러가지가 존재한다.
- 스토리보드에서 View Controller와 view를 지정.
- Nib 파일을 이용해 view지정. nib 파일은 단일 viewController의 view를 지정할 수 있지만 viewController 간의 segue는 정의하지 못한다. 만약 nib 파일을 사용하려고 한다면 init(nibName: bundle :)메서드를 사용해서 초기화할 수 있다.
- loadView() 메서드를 사용하여 View Controller의 view를 지정한다. 이 방법은 뷰 계층을 프로그래밍 방식으로 만들고 해당 계층 구조의 루트 뷰를 viewController의 view 프로퍼티에 할당한다. (아래 loadView 실험)

> viewController는 view와 view가 생성하는 하위뷰의 유일한 소유자다. 스토리보드나 nib파일로 생성된 뷰의 경우 뷰를 요청받을 때 사본을 제공하지만 view를 수동적으로 생성한다면 viewController는 반드시 고유한 view집합을 소유하고 있어야 한다. 

#### 뷰와 관련된 알림
> View가 나타나거나 사라지면 컨트롤러는 자동적으로 자체 메서드를 호출해 하위 클래스가 변동사항에 응답할 수 있도록 한다. viewWillAppear는 화면에 표시 할 뷰를 준비하고 viewWillDisapper에는 변경 내용이나 다른 상태 정보를 저장한다.
> 
<img src="https://i.imgur.com/uAtwUMe.png" width="400" />

#### 뷰 회전 처리
- iOS 8부터 모든 회전 관련 메서드는 deprecated되었고 회전은 이제 viewWillTransition(to:with)메서드를 통해 보고된다.
- viewController는 supportedInterfaceOrientations메서드를 오버라이드해서 방향 리스트를 제한할 수 있다. 일반적으로 시스템은 이 메서드를 윈도우의 루트 viewController 또는 전체화면을 채우기 위해 제공된 viewController에만 호출이 된다. 하위 viewController는 상위 viewController가 화면상에서 제공하는 부분을 사용할 뿐 회전모드의 지원 여부에는 영향을 줄 수 없다.

> 앱이 시작될 때 앱은 반드시 세로 방향으로 인터페이스를 설정해야 한다. 앱이 정의한 메커니즘에 따라 화면을 회전시키는 것은  `application(_:didFinishLaunchingWithOptions:)`가 값을 리턴한 이후부터 적용된다.

#### 컨테이너 뷰 컨트롤러 구현
> UIViewController의 하위 커스텀 클래스는 컨테이너 뷰 컨트롤러로 작동할 수 있다. 컨테이너 뷰 컨트롤러는 여러 개의 컨트롤러를 담고있는 형태로 예시로는 UINavigationController와 UITabBarController가 있다.
- 하위 루트 뷰를 뷰 계층에 추가하기 전에 컨테이너 뷰 컨트롤러가 하위 뷰 컨트롤러를 연결해야 한다. 이렇게 하면 iOS에서 이벤트를 하위 뷰 컨트롤러로 올바르게 라우팅하고 컨테이너가 관리하는 뷰를 볼 수 있다. 

#### 컨테이너 뷰 커스텀 구현
```swift
// child ViewController 생성
let storyboard = UIStoryboard(name: "Main", bundle: .main)
if let viewController = storyboard.instantiateViewController(identifier: "imageViewController")
                                    as? ImageViewController {
   // 현재 viewController에 child viewController 연결
   addChild(viewController)
   view.addSubview(viewController.view)
            
                                        
   // chile View의 제약사항을 생성하고 활성화
   onscreenConstraints = configureConstraintsForContainedView(containedView: viewController.view,
                             stage: .onscreen)
   NSLayoutConstraint.activate(onscreenConstraints)
     
   // child View의 이동이 완료되었음을 알려준다     
   viewController.didMove(toParent: self)
}
```
- container-child 관계를 구축하면 UIKit가 의도치않게 인터페이스를 간섭하는 일을 막아준다. 이 관계가 구축되어있다고 가정하면 UIKit은 container View를 통해 많은 요청을 라우팅하여 하위 뷰 컨트롤러의 동작이 변경될 수 있게 만든다.

#### 메모리 관리
> 메모리는 iOS에서 중요한 리소스이며, viewController는 중요한 시간에 메모리 공간을 줄일 수 있는 내장 기능을 지원한다. 이는 불필요한 메모리를 해제하는 didReceiveMemoryWaning메서드를 통해 자동으로 처리한다.

#### 상태 보존과 복원
- viewController의 restorationIdentifier속성에 값을 할당하면 앱이 백그라운드로 전활될 때 시스템이 viewController에 인코딩을 요청할 수 있다. 이는 restorationIdentifier가 있는 뷰 계층의 뷰를 보장한다. 


## LoadView는 무엇일까? 
> LoadView는 UIViewController가 가지고 있는 메서드로 controller가 관리하는 뷰를 생성한다. 
- 지금까지 사용한 경우는 스토리보드를 통해 view를 추가했다. 하지만 하나의 viewController에는 하나의 루트 view가 존재하기에 스토리보드에 레이블과 버튼과 같은 뷰 객체를 넣는 것은 루트 view안에 넣는 것과 같다. 
쉽게 말해 스토리보드에 레이블과 버튼을 넣으면 view라는 루트 뷰 프로퍼티에 addSubView가 되고 실제로 viewController가 실행될 때 loadView내부에서 스토리보드의 root 뷰를 viewController의 view 프로퍼티에 넣게되어 연결이 된다.
- LoadView는 스토리보드에서 뷰를 만들었다면 사용할 필요가 없다.(재정의하지 않으면 기본적으로 view 프로퍼티를 생성하고 객체들을 생성한다)
- 하지만 코드로 뷰를 만들었다면 loadView내부에서 `super.loadView()`를 통해 view를 생성시켜 준 뒤 `self.view.addSubview(somethingView)`와 같이 루트 뷰에 객체뷰를 추가해줘야 한다. 그런데 여기서 의문이 드는 것은 코드로만 View를 제공할 때 조차도 loadView를 꼭 재정의할 필요는 없어보이는 것이다. 웹뷰를 루트뷰로 넣을 때? 말고는 딱히 효용성을 모르겠다..

#### ViewController의 View 생성 과정
1. viewController.view등의 UIViewController객체의 view property에 접근(메인 스토리보드가 있다면 view가 존재)
2. 메모리상에 view가 존재하지 않으면, loadView가 호출
3. loadView가 override가 되어 있으면, 그 안의 내용을 생성
4. loadView가 override가 안되어 있다면, nibName, nibBundle properties의 nib파일의 로드를 시도
5. 4에서 파일이 존재하지 않으면, ViewController 이름의 nib파일의 로드를 시도
6. 그래도 없다면, empty UIView를 생성
7. viewDidLoad를 호출하여줌.

###### tags: `Auto layout`

#### Reference
- [ViewController](https://melod-it.gitbook.io/sagwa/app-frameworks/uikit/view-controllers/uiviewcontroller)
- [Apple Developer Documentation - UIViewController](https://developer.apple.com/documentation/uikit/uiviewcontroller)
- [loadView](https://medium.com/yay-its-erica/viewdidload-vs-loadview-swift3-47f4ad195602)

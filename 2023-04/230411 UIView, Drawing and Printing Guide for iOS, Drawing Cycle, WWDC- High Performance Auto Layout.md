230411 UIView, Drawing and Printing Guide for iOS, Drawing Cycle, WWDC- High Performance Auto Layout
===
학습내용
- UIView
- Drawing and Printing Guide for iOS
- Drawing Cycle

## UIView

화면 위의 사각형 영역에 대한 내용을 관리하는 객체
> @MainActor class UIView: UIResponder

### Overview
View들은 앱 인터페이스의 근간이되고 `UIView` 클래스는 모든 view에 공통적인 행동을 정의한다. 하나의 뷰 객체는 그 뷰의 사각형 영역안에 내용을 렌더링하고 그 내용과의 상호작용을 처리한다.

`UIView` 클래스는 인스턴스화하고 고정폭의 배경색을 보여주는데 사용되는 구체적인 클래스이다. 
또한 이를 서브클래싱하여 더욱 복잡한 내용을 그릴 수 있다. 앱 내의 레이블, 이미지, 버튼, 다른 요소들을 보여주기 위해 직접 정의하려고 시도하기보다는 UIKit이 제공하는 view subclass를 사용하는 것이 좋다.

view객체들은 사용자와의 상호작용을 하기위한 주 방법이기 때문에 몇가지 책임을 가지고 있다.
- 그리기, 애니메이션
    - 뷰들은 UIKit과 Core Graphics를 이용하여 사각형 영역안에 내용을 그린다.
    - 일부 뷰의 속성들을 새 값으로 애니메이션할 수 있다.
- 레이아웃, 서브뷰 관리
    - 뷰들은 0개이상의 서브뷰들을 가진다
    - 뷰들은 그 뷰의 서브뷰들의 위치와 크기를 조정할 수 있다.
    - 뷰 계층변화에 반응하기 위해 오토레이아웃을 이용해서 뷰의 resizing과 repositioning 규칙을 정의한다.
- 이벤트 핸들링
    - 뷰는 `UIResponder`의 서브클래스이기에 터치나 다른 이벤트에 반응할 수 있다.
    - 뷰들은 gesture recognizer를 설치하여 gesture들을 다룰 수 있다.

뷰들은 연관된 내용들을 조직화하는 뷰 계층을 만들기위해 다른 뷰들안으로 중첩될 수 있다. 
뷰를 중첩하는 것은 중첩된 자식 뷰(subview라 불리는)와 부모 뷰(superview라 불리는)의 관계를 만든다. 부모 뷰는 여러개의 서브뷰들을 포함할 수 있지만 각 서브뷰는 오직 하나의 superview만을 가진다. 
기본적으로 하나의 서브뷰의 보이는 영역이 superview의 영역 바깥으로 확장한다면 Subview내용의 잘림이 발생하지 않는다. 이를 잘리게 하려면 `clipsToBounds` 프로퍼티를 변경하면 된다.

`frame`과 `bounds` 프로퍼티는 각 뷰의 geometry를 정의한다. `frame`프로퍼티는 그 뷰의 superview의 coordinate 시스템안의 origin과 dimension을 정의한다. 
`bounds` 프로퍼티는 뷰가 보는 내부 dimension을 정의하며 custom drawing code에만 거의 독점적으로 사용이 된다. `center` 프로퍼티는 `frame`과 `bounds` 프로퍼티를 직접적으로 변경하는것없이 뷰를 reposition하는 편리한 방법이다. 

### Create a view
보통 스토리보스안에서 library를 통해 뷰를 생성시킬 것이다. 또한 코드로 만들 수도 있다. 
뷰를 만들때 초기 사이즈와 superview와의 위치를 명시해야 한다.
예를 들어 다음 예시는 superview좌표계의 (10, 10)지점만큼 top-left 방향으로 띄어져있다.

```swift
let rect = CGRect(x: 10, y: 10, width: 100, height: 100)
let myView = UIView(frame: rect)
```

또다른 뷰를 추가하기위해 superview위에 `addSubview(_:)`메서드를 호출한다. 뷰에 원하는 만큼 추가할 수 있으며 형제뷰는 overlap되지않고 보여질 수 있다. 
`insertSubview(_:aboveSubview:)`, `insertSubView(_:belowSuubView:)를 이용하여 상대적인 z-order를 지정할수도 있다.

### Draw views
뷰 drawing은 필요에 따라 발생한다. 뷰가 처음 보여졌을때, 레이아웃 변화로 인해 전체 또는 일부가 보이는 경우에 시스템은 뷰에게 내용을 그릴 것을 요청한다.

UIKit과 Core Graphics를 사용하는 custom content를 포함하는 뷰들에는 시스템이 뷰의 `draw(_:)`메서드를 호출한다. 이 메서드의 구현은 뷰의 내용을 현재 graphic context로 그리는 것에 책임을 지고 있다. (graphic context는 `draw`메서드가 호출되기 전에 자동으로 설정한다.)
이렇게 하면 화면에 보여질 정적인 시각화면을 만들 수 있게 된다. 

실제 뷰의 내용이 변경될 때 시스템에게 다시 그려야한다고 알려야 할 책임이 있다. 이를 `setNeedsDisplay`나 `setNeedsDisplay`메서드를 호출함으로써 알릴 수 있다. 이러한 메서드들은 시스템이 다음 drawing cycle에 업데이트해야한다고 알려주는 메서드이다. 
다음 drawing cycle에 뷰를 업데이트할 때까지 기다리기 때문에 이 메서드를 여러 뷰를 동시에 업데이트하는데 호출할 수 있다.

### Animate views
몇가지 뷰 프로퍼티의 변화는 애니메이션될 수 있다. 프로퍼티를 변경하면서 현재값에서 목표값까지 애니메이션할 수 있게 만든다. `UIView` 클래스의 프로퍼티들 중 애니메이션이 가능한 것들은 다음과 같다.
- frame
- bounds
- center
- transfomr
- alpha
- backgroundColor

변화를 animate하기 위해 `UIViewPropertyAnimator`객체를 만들고 handler block을 사용해 뷰 프로퍼티의 값을 변화한다. `UIViewPropertyAnimator` 클래스는 애니메이션 시간과 타이밍곡선을 명시하도록 요구하지만 실제 애니메이션을 수행한다. 
현재 실행 중인 속성 기반 애니메이터를 일시 중지하여 애니메이션을 중단하고 인터렉티브하게 구동할 수도 있다.

### Threading consideration
앱 인터페이스의 조작은 메인 스레드에서 해야한다. 그러므로 항상 `UIView` 클래스의 메서드는 메인스레드에서 호출해야 한다. 이 규칙을 지키지 않는 유일한 시간은 뷰 객체가 스스로 만들어질 때이다. 하지만 모든 다른 조작은 메인스레드에서 되어야 한다.


### Subclassing notes
`UIView` 클래스는 사용자 상호작용도 필요한 시각적인 내용을 위한 핵심 subclassing 지점이다. 비록 `UIView`를 서브클래싱 해야 하는 많은 이유가 많지만 기본적인 `UIView`와 표준 시스템 뷰가 필요한 기능을 제공하지 않은 경우에 그렇게 하는 것이 좋다. 서브클래싱하는 것은 뷰를 구현하고 성능을 조정하기 위해 더 많은 작업이 필요하기 때문이다.

### Methods to override
`UIView`를 서브클래싱할 때 재정의해야 하는 몇 가지 메서드와 필요에 따라 재정의할 수 있는 많은 메서드가 있다. `UIView`는 거의 모든 것이 설정가능한 클래스이기 때문에 커스텀 메서드를 override하는 것 없이 정교한 뷰를 구성하는데 많은 방법이 존재한다. 이는 Alterantives to Subclassing section에서 다룬다. 
다음 메서드는 `UIView`를 서브클래싱하면서 override할 수 있는 메서드들이다.

- initialization
    - init(frame:): 이 메서드를 구현할 것은 권고한다. 이 메서드를 구현하는 것 대신에 커스텀 이니셜라이저를 구현할 수도 있다.
    - init(coder:): 만약 스토리보드나 nib 파일로부터 뷰를 로드하고 커스텀 이니셜라이저를 필요로 한다면 이 메서드를 구현한다.
    - layerClass: 만약 뷰가 백업 스토어를 위해 다른 Core Animation layer를 사용하길 원한다면 사용한다. 예를 들어 만약 뷰가 큰 스크롤 영역에 보여지게 한다면 `CATiledLayer` 클래스 프로퍼티를 설정할 수 있다.  
- Drawing and printing
    - draw(_:): 만약 뷰가 커스텀 content를 갖는다면 이 메서드를 구현한다. 만약 뷰가 커스텀 drawing을 갖지 않는다면 이 메서드를 오버라이드 하지 않는다.
    - draw(_:for:): 출력되는 동안 뷰의 내용이 다르게 보여지길 원한다면 이 메서드를 구현한다.
- Layout and Constraints
    - requiresConstraintBasedLayout: 만약 뷰 클래스가 적절하게 작동하기위해 제약이 필요한 경우 이 프로퍼티를 사용한다. true가 기본값으로 이는 constraint-based 즉 오토레이아웃 시스템을 따른다는 뜻이다.
    - updateConstraints(): 만약 뷰가 서브뷰들 사이에 커스텀 제약을 만들 필요가 있다면 이 메서드를 구현한다.
    - alignmentRect(forFrame:), frame(forAlignmentRect:): 다른 뷰와의 align을 어떻게 할지를 결정하려면 이 메서드를 override한다.
    - didAddSubView(_:), willRemoveSubview(_:): 서브뷰를 추가하고 제거하는 것을 추추적하고자 할 때 이 메서드를 구현한다.
    - willMove(toSuperView:), didMoveToSuperView(): 뷰 계층에서 현재 뷰의 이동을 추적하고 싶을 때 이 메서드를 구현한다.
- Event Handling
    - gestureRecognizerShouldBegin(_:): 만약 뷰가 터치 이벤트를 직접적으로 다루게 하길 원하고 연결된 gesture recognizer가 추가 액션을 트리거하는 것을 막기위해 이 메서드를 구현한다. (false를 주면 모든 gesutre recognizer가 무시됨)
    - touchesBegan(_:with:), touchesMoved(_:with:), touchesEnded(_:with), touchesCanclled(_:with:): 만약 터치이벤트를 직접 다루고 싶다면 이 메서드를 구현한다. 

### Alternatives to subclassing
많은 뷰 행위들은 서브클래싱하는 것없이 수정될 수 있다. 메서드를 오버라이드하기 전에 다음의 프로퍼티로 수정할 수 있는지를 고려해보자.
- addConstraint: 뷰와 서브뷰들을 위해 자동적인 레이아웃 동작을 정의한다.
- autoresizingMask: 슈퍼뷰의 frame이 변경되었을 때 자동적인 레이아웃 동작을 제공한다. 이 동작들은 constraint와 함께 결합될 수 있다. 뷰의 bounds가 변경된다면 그 뷰의 서브뷰들은 각 서브뷰들의 autoresizing mask에 따라 resize하게 된다. 
- contentMode: 뷰의 프레임과 달리 내용에 대한 레이아웃 동작을 제공한다. 또한 이 프로퍼티는 내용이 뷰에 어떻게 맞출지 영향을 주고 캐시될지 다시그려질지에 영향을 준다.
- isHidden, alpha: 렌더링된 내용을 숨기거나 alpha를 적용하기 보다는 뷰의 투명성을 변경한다. 
- backgroundColor: 스스로 컬러를 그리기보다는 view의 컬러를 설정한다.
- Subviews: `draw`메서드를 사용해 내용을 그리기보다는 이미지나 레이블 삽입한다.
- Gesture recognizer: 터치 이벤트를 방해하고 관리하는 subclass를 사용하기 보다는 gesture recognizer를 추가해서 사용한다
- Animations: 직접 애니메이션 변화를 주기보다는 Core Animation에 의해 제공되는 내장된 애니메이션 지원을 사용한다. 빠르고 쓰기도 쉽다
- Image-based backgrounds: 상대적으로 정적인 내용을 보여주는 뷰들에는 이미지를 서브클래싱하고 직접 그리는 것 대신에 UIImageView 객체를 gesture recognizer와 함께 사용하는 것을 고려한다. 대안적으로 일반 UIView 객체를 사용하고 뷰의 CALayer 객체의 콘텐츠로 이미지를 할당할 수도 있다.

애니메이션들은 복잡한 drawing code를 구현하거나 서브클래싱하는 것 없이 뷰의 변화를 보여주는 또다른 방법이다. `UIView`클래스의 많은 프로퍼티들은 animatable하며 이는 이러한 프로퍼티들의 변경이 system-generated한 애니메이션을 트리거할 수 있다는 것을 의미한다. 

## Drawing and Printing Guide for iOS
유저 인터페이스에 있어 품질 좋은 그래픽은 중요한 부분이다. 고품질 그래픽을 제공하는 것은 앱을 좋아보이게 만들 뿐만 아니라 시스템의 나머지 부분의 자연스러운 확장처럼 보이게 한다. 
iOS는 고품질 그래픽을 만드는데 두가지 주요한 경로를 제공한다. (OpenGL또는 Quartz를 사용하여 렌더링하는 CoreAnimation, UIKit) 이 문서는 네이티브한 렌더링인 UIKit, CoreAnimation을 다룬다.

Quartz는 메인 drawing 인터페이스이다.이는 경로기반 drawing, anti-aliased 렌더링, 그라데이션 패턴, 이미지, 색상, 좌표공간 변환, PDF 문서생성, display, parsing을 제공한다. 
UIKit은 line art, Quartz images, 색상 조작을 위해 Objective-C wrapper를 제공한다. 
Core Animation은 많은 UIKit view 프로퍼티들이 애니메이션하게 만들게 도와주고 커스텀 애니메이션을 구현할 수 있다.

이 챕터는 구체적인 drawing 기술과 함께 iOS app에서 drawing process의 개요를 다룬다. 

### The UIKit Graphics System
iOS에서 화면에 보여지는 모든 drawing은(이 drawing이 OpenGL, Quartz, UIKit, Core Animation을 수반하든 안하든) `UIView` 클래스의 인스턴스, 혹은 그것의 서브클래스 안에서 일어난다.
View들은 drawing이 발생하는 화면의 부분을 정의한다. 만약 시스템이 제공하는 뷰를 사용한다면 이 drawing은 자동으로 다뤄진다. 
하지만 만약 커스텀 뷰를 정의한다면 drawing code를 제공해야 한다. 만약 Quartz, CoreAnimation, UIKit을 사용한다면 Drawing Concept을 알고 사용해야한다.

화면에 직접적으로 그리는 것 외에 UIKit은 오프스크린 비트맵이나 PDF 그래픽 컨텍스트로 그릴 수도 있다. 
오프스크린 컨텍스트로 그릴 때 뷰 드로잉 사이클을 적용하는 게 아니기 때문에 view안에 그리는 것이 아니다. 

### The View Drawing Cycle
`UIView`의 서브클래스에 대한 기본 drawing 모델은 요청에 따라 내용을 업데이트하는 것을 수반한다. `UIView` 클래스는 업데이트 프로세스를 더욱 쉽고 효율적이게 만든다. 하지만 업데이트 요청을 모아서 가장 적절한 시간에 드로잉 코드에 전달한다. 

뷰가 처음 보여질때다 다시 그려질때 iOS는 그 뷰에게 `draw` 메서드를 호출함으로써 내용을 다시 그리도록 요청한다. 

뷰가 업데이트 될 수 있는 몇가지 액션이 존재한다. (하지만 1~3번은 실제로 뷰가 다시 그려지는 작업이 아니기 때문에 draw메서드가 호출이 되지 않는다.)
- 부분적으로 뷰를 막고있는 또다른 뷰를 옮기거나 제거함으로써
- `hidden` 프로퍼티를 `No`로 설정함으로써 숨겨진 뷰를 다시 보여지게 만들면서
- 화면 밖으로 스크롤 했다가 다시 돌아왔을 때
- 명시적으로 `setNeedsDisplay`, `setNeedsDisplayInRect:`메서드를 호출했을 때

`draw`메서드를 오버라이드해서 커스텀해서 이미지나 그림을 그리려고 한다면 추후에 변경사항이 있을 때 자동적으로 setNeedsDisplay를 호출해주지 않는다. 따라서 직접 커스텀해서 그렸다면 변경사항 후에 이 메서드를 호출해야 한다.(UIKit이 제공하는 기본적인 draw의 경우에는 다시 그리는 작업이 있을 때 자동적으로 호출해준다)

시스템 뷰들은 자동으로 다시 그려주지만 커스텀 뷰는 `draw`메서드를 재정의해서 수행해야 한다. `draw`메서드 안에서 native drawing 기술을 사용해서 shape, text, image, gradient, 시각적인 내용들을 그려준다. 

`draw`메서드 호출후에 뷰는 스스로 업데이트가 되었다는 것을 표시하고 다른 업데이트 사이클이 트리거될때까지 기다린다. 만약 뷰가 정적인 내용을 보여준다면 스크롤하거나 다른 뷰의 존재에 의해 가시성이 변경되는 경우에만 반응하면 된다.

하지만 만약 뷰의 내용이 변경되길 원한다면 뷰에게 다시 그리도록 요청해야 한다. 이를 위해 `setNeedsDisplay`, `setNeddsDisplayInRect`메서드를 호출해야 한다. 

### Coordinate System and Drawing in iOS
iOS에서 앱이 무언가를 그릴 때 좌표 시스템에 의해 정의된 두 개의 dimensional space에 취지되어야 한다. 이 개념은 간단해보일지 모르지만 그렇지 않다. iOS의 앱들은 때떄로 그릴 때 다른 좌표계 시스템을 다룰 필요가 있다.

iOS에서 모든 drawing은 graphic context에서 나타난다. 개념적으로 graphic context는 어디서 그리고 어떻게 drawing이 일어날지를 설명하는 객체이다. 이는 그려질때 사용하는 색상, 잘리는 영역, 선 width, style과 관련된 정보들을 퐇마하고 있다.

각 graphic context는 좌표계 시스템을 갖고있는데 정확하게는 3개의 좌표시스템을 갖는다.
- drawing (user) 좌표계 시스템. 이 좌표계시스템은 그리기 명령을 실행할 때 사용된다.
- view 좌표계 시스템. 이 좌표계 시스템은 뷰와 관계된 고정된 좌표계 시스템이다.
- device 좌표계 시스템. 이 좌표계 시스템은 물리적인 화면위의 픽셀을 나타낸다


## Drawing Cycle
드로잉 사이클은 View가 로드되거나 변경되었을 때 이를 화면에 적용시켜 그리는 과정을 뜻한다. 이들은 즉각적으로 처리되지 않고 Main Run loop에 의존하는 형태를 지닌다.

<img src="https://i.imgur.com/Bg79Qdk.png" width="500"/><br/>

업데이트 사이클은 앱이 모든 이벤트 처리 코드실행을 마치고 제어가 기본 실행루프로 돌아가는 지점이다. 이 업데이트 사이클 시점에 시스템은 레이아웃, 디스플레이, 제약조건을 업데이트하기 시작한다. 

사용자 상호작용과 레이아웃 업데이트 사이의 지연은 사용자가 인지할 수 없어야 한다. iOS 애플리케이션은 일반적으로 60fps로 애니메이션되는데 이는 1초에 60번 새로고침한다는 뜻이다. 이 작업이 엄청 빠르기 때문에 UI의 지연을 인지하지 못하는 것이다. 

업데이트 사이클에만 실질적으로 변화를 가지기 때문에 메인 런루프동안 "이 작업은 돌아오는 업데이트 주기에 꼭 실행해야 해"라고 말해주는 메서드가 필요하다. 

그래서 `UIView`에서는 실질적으로 뷰가 변하는 부분과 뷰가 변하도록 작업을 예약하는 부분 두 가지로 나뉜다. 

<img src="https://i.imgur.com/9PlEeDl.png" width="500"/><br/>

위쪽은 뷰가 실질적으로 변하는 메서드이고 아래 setNeeds를 prefix로 갖는 메서드들은 뷰가 변하도록 작업을 예약하는 메서드이다.

<img src="https://i.imgur.com/3lxg6li.png" width="500"/><br/>

View의 흐름만 본다면 `updateContraints` -> `layoutSubviews` -> `drawRect`순으로 메서드가 호출이 된다. 

하지만 모든 뷰들이 꼭 세 개의 메서드를 호출하는 건 아니고 constraint가 존재하지 않는 뷰라면 updateConstraint를 호출하지않고, layout을 그리는 부분이 존재하지않는다면 layoutSubview를 호출하지 않는다.(아래 그림 참조)

<img src="https://i.imgur.com/BnYFn6y.png" width="500"/><br/>

애플은 뷰를 변경하는 메서드들을 직접 호출하지 말라고 한다. 뷰를 업데이트하는 작업은 비용이 크기 때문에 하나의 뷰만 특정해서 직접 호출한다면 성능이 떨어질 수 있기 때문이다.

그래서 뷰를 변경하도록 작업을 예약하는 메서드들로 시스템의 다음 드로잉 사이클에 그리도록 해야 한다.

물론 이 작업을 예약하는 메서드들을 직접적으로 호출할 일은 거의 없다.
View의 크기가 변경되거나 배경색이 변하거나 디바이스 회전이 일어나거나, Constraint조정이 있을 때 등 자동으로 호출하기 때문에 특별한 경우가 아니면 신경쓰지 않아도 된다. 

`layoutIfNeed`메서드를 호출하게 된다면 업데이트 사이클까지 기다리지 않고 바로 `layoutSubviews`를 호출해서 변화를 적용시킨다. 이는 애니메이션처럼 자연스럽게 외형을 변화시키는 경우에 사용한다.

### Drawing 사이클 라이프사이클 메서드 확인

<img src="https://i.imgur.com/ITxN0Zn.png" width="300"/><br/>

해당 커스텀 뷰를 놓고 다음 메서드들을 오버라이드 했다.

```swift
class CustomView: UIView {
    override func updateConstraints() {
        super.updateConstraints()
    }
    
    
    override func layoutSubviews() {
        super.layoutSubviews()
    }
    
    
    override func draw(_ rect: CGRect) {
        super.draw(rect)
    }
}
```

그리고 뷰컨에서는 파란 뷰의 constraint를 잡아준 뒤에 특정 시간 후에 deactivate하고 새로운 constraint를 잡아주었다.

```swift
class ViewController: UIViewController {
    @IBOutlet weak var customView: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        self.customView.translatesAutoresizingMaskIntoConstraints = false
        
        let constraints1 = [
          self.customView.heightAnchor.constraint(equalToConstant: 300),
          self.customView.widthAnchor.constraint(equalToConstant: 300),
          self.customView.centerYAnchor.constraint(equalTo: self.view.centerYAnchor),
          self.customView.centerXAnchor.constraint(equalTo: self.view.centerXAnchor),
        ]
        NSLayoutConstraint.activate(constraints1)

        DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
          let constraints2 = [
            self.customView.heightAnchor.constraint(equalToConstant: 100),
            self.customView.widthAnchor.constraint(equalToConstant: 100),
            self.customView.centerYAnchor.constraint(equalTo: self.view.centerYAnchor),
            self.customView.leadingAnchor.constraint(equalTo: self.view.leadingAnchor),
            self.customView.trailingAnchor.constraint(equalTo: self.view.trailingAnchor),
            
          ]
          NSLayoutConstraint.activate(constraints2)
        }
    }
}
```

첫 번째 constraint를 잡아줄 때는 다음의 순서로 메서드들이 호출되었다.
> updateConstraint -> layoutSubviews -> draw

예상했던 결과이다. 
하지만 deactivate하고 새롭게 잡아주었을 때의 순서는 다음과 같았다.
> layoutSubviews -> draw

음...? deactivate하고나서 새로운 제약을 레이아웃 엔진에게 건너주었는데도 `updateConstraint`가 호출이 안된다고??

deactivate가 문제인가 싶어서 이 코드를 빼고 다시 실행해봤는데도 호출순서가 동일했다.

비활성화가 꼭 constraint를 다 없앤 후에 처음부터 시작하는 건 아닌 것 같다. (레이아웃 엔진의 특성상 그런 것 같다.)

결론적으로 만약 constraint를 업데이트했는데 레이아웃 변화가 즉각적으로 일어나지않았다면 `setNeedsUpdateConstraints()`를 호출함으로써 바로 감지할 수 있도록 만들 수 있다는 것이다.


## WWDC- High Performance Auto Layout
- 모든 constraint를 deactivate하고 activate하기 보다는 제약들을 여러그룹으로 나눠서 특정 제약만 activate, deactivate한다. 그래서 churning(엔진에서 반복적인 레이아웃 계산)을 줄여야 한다. 
- 요소들이 최대한 연관을 갖도록 한다. 서로 연관이 없어보이는 두 개의 뷰에 제약을 걸 때 보통은 더 복잡한 제약이기 때문에 오래 걸린다고 생각하지만 이는 엔진입장에서는 간단한 계산이다. 오히려 두 개의 뷰에 따로따로 제약을 줄 때 각각 연산을 해야하기 때문에 오래걸릴 수 있다. 따라서 최대한 뷰 간의 연관이 있는 제약을 주고 너무 복잡하게만 설정하지 않는다.(디버깅 시 힘들 수도 있다)
- removeFromSuperView보다 hide가 훨씬 싼 비용이기에 레이아웃 설정 시 고려한다.
- 렌더 루프는 1초에 120 프레임을 실행하기에 너무많이 호출되니까 민감한 코드는 사용하지 않는다. 


### 참고문서
-[Apple Docs - UIView](https://developer.apple.com/documentation/uikit/uiview#1652896)
-[Apple Archive - Drawing and Printing Guide for iOS](https://developer.apple.com/library/archive/documentation/2DDrawing/Conceptual/DrawingPrintingiOS/GraphicsDrawingOverview/GraphicsDrawingOverview.html)
-[Demystifying iOS Layout](https://tech.gc.com/demystifying-ios-layout/)
-[Drawing Cycle](https://inuplace.tistory.com/1137)
-[WWDC 18 - High Performance Auto Layout](https://developer.apple.com/videos/play/wwdc2018/220/)

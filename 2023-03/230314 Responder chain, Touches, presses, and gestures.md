230314 Responder chain, Touches, presses, and gestures
===
학습내용
---
- Using responders and the responder chain to handle events Article 읽기
- Touches, presses, and gestures

## Using responders and the responder chain to handle events

### ✏️ 개요

앱은 responder 객체를 사용해 이벤트를 받고 처리한다. 하나의 responder 객체는 `UIResponder` 클래스의 인스턴스이고, 일반적인 하위클래스에는 `UIView`, `UIViewController`, `UIApplication`이 포함된다.
`responder`는 원시 이벤트데이터를 받고 이벤트를 처리하거나 또 다른 responder 객체에게 보내야 한다. 앱이 이벤트를 받았을 때 자동으로 그 이벤트를 `first responder`라고 불리는 적절한 responder 객체에게 보낸다.

처리되지 않은 이벤트들은 활성화된 responder chain안에서 responder 객체끼리 전달된다. 이 chain은 앱 내의 responder 객체들의 동적 설정이다. 아래 이미지는 `label`, `textfield`, `button`, `background`를 보여주는 responders를 보여준다. 이는 어떻게 이벤트들이 다음의 responder 객체로 전달되는지 보여준다.

<img src="https://i.imgur.com/YT3H1gk.png"/><br/>

만약 텍스트필드가 이벤트를 다루고있지 않는다면 UIKit은 그 이벤트를 textfield의 부모인 UIView에 보낸다. 그리고 그 UIView에서도 다루지않으면 window의 루트뷰인 UIView에 전달이 된다. 이 루트뷰는 window에 이벤트를 전달하기전에 자신을 소유하고 있는 viewController로 responder chain이 전환이 된다. 그 이후 UIWindow에 전달하게 된다. 하지만 window도 이벤트를 처리할 수 없다면 UIKit은 UIApplication에 전달하고 만약 해당 객체가 UIResponder의 인스턴스이고 응답자 체인의 일부가 아닌 경우 app delegate에 전달할 수 있다.


### ✏️ 이벤트의 first responder 결정하기
UIKit은 이벤트의 타입에 기반해서 first responder에 맞는 객체를 지정한다.


<img src="https://i.imgur.com/AL2u21l.png" width="600"/><br/>

Control들은 action 메세지를 이용하여 연관된 타겟 객체와 직접 소통한다. 사용자가 컨트롤과 상호작용을 했을때 컨트롤은 엑션메세지를 타겟 객체에 보낸다. 엑션메세지는 이벤트가 아니지만 responder chain의 이점을 취할 수 있다. 


### Note
accelermeters, gyroscpes, magnetometer와 관련된 모션 이벤트들은 responder chain을 따르지 않는다. 대신에 Core Motion은 지정된 객체로 직접 전달한다. 하나의 control의 타겟객체가 nil이라면 UIKit은 responder chain을 타서 적절한 action method가 구현되어있는 객체를 찾는다. 
예를 들어 UIKit editing menu는 `cut(), copy(), paste()`라는 이름의 메서드가 구현되어있는 객체를 찾을 때까지 responder chain을 타게된다.

Gesture recognizer들은 touch, press이벤트를 받는다.(뷰가 그것들을 하기 전에) 만약 view의 gesture recognizer가 터치의 순서를 인지하는데 실패했다면 UIKit이 그 터치들을 view로 보낸다. 
만약 그 뷰가 touch를 처리하지 않으면 UIKit은 그걸 responder chain으로 보낸다. gesture recoginzer에 대한 자세한 정보는 다음을 참고한다. [Handling UIKit gestures](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/handling_uikit_gestures)

### ✏️ 어떤 responder가 터치 이벤트를 포함하는지 결정하기

UIKit은 touch이벤트가 일어난 곳을 알기위해 view에 기반한 hit-testing을 사용한다. 특히 UIKit은 터치위치와 뷰 계층 내의 뷰객체 bound와 비교한다. 
`UIView`의 `hitTest(_:with:)`메서드는 뷰 계층을 순회하여 터치 이벤트에 대한 첫 번째 응답자가 되는 지정된 터치를 포함하는 가장 깊은 하위 뷰를 찾는다.

터치가 발생했을 때 UIKit은 `UITouch` 객체를 생성하고 하나의 뷰와 연관짓는다. 그 터치 위치 또는 다른 파라미터들이 변할 때 UIKit은 같은 `UITouch`객체를 새로운 정보로 업데이트한다. 변하지 않는 유일한 프로퍼티는 view이다.(터치 위치가 원래 뷰의 바깥으로 이동하더라도 터치의 view 프로퍼티는 변하지않는다.). 그 터치가 끝났을때 UIKit은 `UITouch`객체를 해제한다.


### Note
만약 터치위치가 view bound 외부에 있다면 `hitTest` 메서드는 해당 뷰와 그 뷰의 모든 하위뷰를 무시한다. 결과적으로 view의 `clipsToBounds` 프로퍼티가 true라면 해당 뷰의 bound 외부에 있는 subview들은 터치를 포함하고 있어도 반환하지 않는다. 

### ✏️ responder chain 교체하기
responder 객체의 `next`프로퍼티를 override해서 responder chain을 변경할 수 있다. 

많은 UIKit 클래스들은 이미 이 프로퍼티를 override하고 있고 특정한 객체를 반환한다.
- UIView: 만약 view가 뷰 컨트롤러의 루트 뷰라면 next responder는 view controller이고 그렇지 않으면 view의 superview이다.
- UIViewController
    - 만약 뷰 컨트롤러의 뷰가 window의 루트 뷰라면 next responder는 window 객체이다.
    - 만약 뷰 컨트롤러가 또다른 뷰컨트롤러에 의해 보여진다면 next responder는 보여지는 view controller이다.
- UIWindow: window의 next responder는 UIApplication 객체다
- UIApplication: next responder가 app delegate이다. 하지만 만약 앱 델리게이트가 오직 하나의 UIResponder 인스턴스이고 view, viewcontroller, app object가 아닌 경우에만 사용가능하다.


### 참고자료
- [Reponder Chain](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/using_responders_and_the_responder_chain_to_handle_events)

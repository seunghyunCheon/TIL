230113 Auto layout, alignemnt, priority, Intrinsic size, CHCR
## 오토레이아웃 정복하기

### Alignment
- 왼쪽 view에 leading, top, bottom, leading에 20을 주고 오른쪽 view에 trailing의 20을 준 상태.
- 오른쪽 view는 leading이 왼쪽 view와 이미 관계되어있기 때문에(왼쪽 view에서 trailing을 설정해서) 알아서 leading이 결정된다. 

<img src="https://i.imgur.com/OcScxcn.png" width="300" height="600"/>

- 만약 오른쪽 view를 왼쪽 뷰와 동일하게 맞추고 싶다면 top과 bottom의 alignment를 맞추면 된다.

<img src="https://i.imgur.com/jYf73IA.png" width="300"/>

- 이를 코드로 표현하면 다음과 같다.
```swift
// Vertical Constraints
Green.top = 1.0 * Superview.top + 20.0
Superview.bottom = 1.0 * Green.bottom + 20.0
Green.top = 1.0 * Purple.top + 0.0 // Purple의 top을 Green에 맞춤으로써 Green크기에 유동적으로 변하게 만들 수 있다.
Green.bottom = 1.0 * Purple.bottom + 0.0
```


### Priority
> 같은 제약에 대해 conflict이 난다면 priority가 높은 것을 우선적으로 적용
<img src="https://i.imgur.com/MmIoGNO.png" width="300"/>
- 오른쪽 뷰의 leading이 왼쪽 뷰의 trailing에 20 떨어진 관계와 100 떨어진 관계 2개가 존재한다면 priority가 높은 것을 기준으로 배치된다. 
- 또는 100을 준 제약사항에 Less Than or Equal을 준다면 그보다 더 작은 20제약사항에 맞게 배치된다.

<img src="https://i.imgur.com/xXQ9BWk.png" width="250"/>


### Intrisic size
> 본질적인 컨텐츠 크기
- Intrinsic size는 컨트롤마다 존재하는 게 다르다.
- UIView의 경우 Intrinsic Size가 존재하지 않기 때문에 위치 제약조건 뿐만 아니라 size에 대한 제약조건도 걸어줘야 한다.
- 반면에 Label, buttons, switchs, textField는 width와 height에 대한 Intrinsic size를 가지기 때문에 컨텐츠의 양이 곧 Intrinsic size가 된다. 
- Slider는 height에 대해서만 고유 크기를 갖는다.
- Text view와 Image view는 상황에 따라 다르다.

#### UILabel
<img src="https://i.imgur.com/G4fsXSy.png" width="250"/><br>

> leading과 top만 잡혀도 size가 결정된다.

#### UIButton
<img src="https://i.imgur.com/qgefUUc.png" width="250"/><br>

> leading과 top만 잡혀도 size가 결정된다.

#### UISlider
<img src="https://i.imgur.com/phTTego.png" width="250"/><br>

> leading만 top만 잡으면 height만 Intrinsic size를 갖고 width는 잡히지 않아 설정해줘야 한다.

#### UIView
<img src="https://i.imgur.com/kIUTE0s.png" width="250"/><br>

> top과 trailing만 잡으면 width와 height둘 다 Intrisic Size를 갖지않아 둘 다 설정해줘야 한다.

- 스토리보드에서 UIView에 직접 width, height제약을 주는 방법말고 코드로도 제약을 줄 수 있다.
- 먼저 MyView라는 파일을 만들어 intrinsicSize를 오버라이딩 한다.
```swift
import UIKit

@IBDesignable
class MyView: UIView {
    override var intrinsicContentSize: CGSize {
        return CGSize(width: 50, height: 50)
    }
}
```
- 스토리보드에서 view의 클래스를 MyView로 설정해준다.

<img src="https://i.imgur.com/SAh6weL.png" width="250"/><br>

#### CHCR
> contents hugging과 contents compression resistance의 약어로 contents compression resistance외부에서 힘을 넣을때 intrinsic를 버티려고 하는 힘이고(잘리지 않기위해) contents hugging은 반대되는 개념은 아니고 intrinsic size에 딱 맞게 하려고 하는 힘이다. 즉 늘어나는 힘이 아닌 늘어나지 않으려고 버티는 힘인 것이다

#### CHCR 예제
<img src="https://i.imgur.com/HWMzMPC.png" width="250"/><br>
- leading과 trailing을 각각의 레이블에 20을 주게 된다면 컴퓨터입장에서 각 버튼의 사이즈를 어느정도 줘야할 지 몰라 빨간줄이 나온다. 
- 이 상태에서 첫 번째 레이블의 크기를 크게하면 다음화면처럼 두번째 레이블과 세번째 레이블이 덮여진다.
<img src="https://i.imgur.com/vY3M802.png" width="250"/><br>
- 만약 두 번째와 세 번째의 CR Priority를 첫 번째 레이블보다 높인다면 intrinsic Size를 지키려고 하기 때문에 다음 화면처럼 보여지게 된다.
<img src="https://i.imgur.com/aU293NK.png" width="350"/><br>
- 그런데 아직도 빨간줄이 보인다. 이는 CH를 설정해주지 않았기 때문이다. 즉 어떤 것이 intrinsic size에 딱 맞게 하려고 하는 힘이 강한지에 대해 우선순위를 설정해주지 않았기 때문이다. 이에 대해 두번째 레이블의 hugging을 가장 낮게 주면 intrinsic size가 가장 낮기 때문에 가장 넓은 width를 갖게 된다.
<img src="https://i.imgur.com/PmeP2a9.png" width="350"/><br>

#### 결론
- 글자가 잘리지않고 보여지게 만드려면 CR우선순위를 높게주고 컨텐츠가 커지지않고 고유 사이즈를 유지해야 한다면 CH우선순위를 높여주자

#### 필수 제약조건이 선행되어야 할 때
> 왼쪽View와 오른쪽View 두 개가 존재할 때 왼쪽 View는 반드시 최소 150point크기의 width를 가지고 오른쪽 View는 왼쪽 View의 두 배를 가지려고할 때의 상황
- 01 왼쪽View의 Width를 150을 준다. 
- 02 오른쪽 View가 왼쪽 View의 2배를 갖는 multiplier를 준다. 
- 03 왼쪽View는 150을 가지려하면서 오른쪽View의 1/2크기를 가져야하기에 충돌이 난다.
- 04 2:1비율을 준 제약사항의 priority를 낮춘다.
- 05 첫번째 view의 width제약사항을 준 곳을 Equal이 아닌 Equal than Greater로 준다.

마지막에 Equal than Greater로 준 이유는 최소150포인트만 충족시킨다면 오른쪽이 2배크기가 되는 것이 충족되어야 할 제약이기 때문이다. 이렇게 하면 가로로 방향전환을 해도 2배크기를 가질 수 있게 된다.

<img src="https://i.imgur.com/L0UMhVJ.png" width="350"/><br>

#### autoLayout 예제
<img src="https://i.imgur.com/6kX7ljW.png" width="400" height="800"/>

- imageView가 SuperView의 30%를 가지면서 최대 150point를 갖고자한다면(가로의 경우를 고려) 30%의 제약의 priority를 낮춘다.
- 각각의 레이블들은 이미지를 기준으로 vertical제약을 설정한다
- 레이블들은 leading과 trailing이 일치가 된다.

###### tags: `Auto layout`, `alignment`, `priority`, `Intrinsic size`, `CHCR`

### reference
[야곰닷넷 - 오토 레이아웃](https://yagom.net/courses/autolayout/)

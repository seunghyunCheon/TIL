221229 Enum, HIG(Layout, Action Sheet, Alert, Pull-down buttons, pop up)
===
TIL (Today I Learned)
===
학습내용
---
- Enum 고찰
- HIG 학습

----
## <span style="color:#007bff">Enum</span>
swift 공식문서의 Property부분을 읽던 중 아래와 같은 문장을 보게 되었다.
> Stored properties are provided only by classes and structures.

### 저장 속성은 왜 클래스와 구조체에만 제공되는 거지? 🤔
Enum은 속성은 계산속성과 타입저장속성만 존재가 가능하다. 저장 속성은 해당 인스턴스의 성격을 지칭하는 데이터라고 볼 수 있기에 열거형에서는 성격을 지니는 속성을 가지기를 거부하고 있는 것이다. 이는 열거형자체가 타입의 성격을 강하게 갖고있기 때문인 것 같다.

열거형을 다음과 같이 정의하고 사용한다.
```swift
enum Company {
    case Samsung
    case Apple
    case Google
}
let sam: Company = Company.Samsung
```
sam이라는 변수는 Company의 인스턴스이면서 특정 데이터라고 설명할 수 있다. 
하지만 클래스나 구조체에서 sam에 인스턴스를 생성시켰다면 특정 데이터라고 설명하기보다는 완전한 인스턴스라고 표현할 수 있을 것 같다.
열거형자체가 데이터를 뜻하기 때문에 데이터안에 또 다른 데이터를 넣는 논리가 어색하기 떄문에 열거형에는 저장속성을 갖지 못하는 것 같다.(추측)

##  <span style="color:#007bff">Layout</span>
### SafeArea
> 안전영역은 navigation, tool bar, tab bar에 포함되지 않는 영역으로 아이폰의 Dynamic Island나 일부 sensor housing과 같은 장치의 대화형 및 디스플레이 기능을 피하기 위해 사용이 된다.

쉽게 말해 홈 인디케이터나 노치와 같은 영역에 침범하지 못하게 시스템적으로 마진을 준 영역을 말한다.

### Best practices
- 가능하면 동일한 내용을 보여주는 화면에서 문맥이 바꼈을 때(회전하거나 화면 크기를 줄이는) 일관적인 레이아웃을 제공한다.
- 각 플랫폼의 핵심 디스플레이와 시스템 특징을 존중한다(플랫폼 마다 다른 모양의 sensor housing, display를 갖고 있는 것을 고려하라는 뜻으로 추측)
- 상대적인 중요도를 나타내기 위해 배치를 사용한다. 일반적으로 중요한 내용은 upper half of the screen, near the leading side에 작성한다.
- 충분한 공간을 통해 중요한 정보를 높인다.중요하지 않은 내용은 스크롤과 같은 행동을 통해 보여지도록 한다.
- 사람들이 원하는 정보를 찾을 수 있도록 visual grouping을 한다. (negative spcae, 구분선, 색 등을 이용하여)
- 정렬을 사용해서 시각적인 스캔을 용이하게 하고 조직 및 계층을 전달한다.
- 종횡비에 유의한다. 만약 artwork의 종횡비가 맞지 않을때 artwork의 비율을 바꾸는 것 대신에 확대해서 화면을 채운다.
- 텍스트 크기 변화에 준비한다.
- 가능하다면 스크린에 보이지 않는 내용들을 부분적으로 보여지게 함으로써 아래 있음을 암시하게 만든다.
- touch screen에서 상호작용할 수 있는 충분한 터치타겟을 제공한다. 최소 44x44 Point를 유지한다.
- 앱의 방향과 위치, 텍스트 사이즈를 변경해서 미리 확인해라.

### text size changed 실험

텍스트 사이즈에 변환해서 레이아웃을 대응하는 게 감이 잘 오지않아 크리스티와 같이 실제 아이폰 기기를 통해 설정에서 텍스트 크기를 바꿔 실험해봤다.
이를 대응하는 어플들은 youtube, slack, discord등 외국계 기업이 대응하고 있었다. 하지만 카카오톡, 네이버, 당근마켓 등 한국기업들은 잘 제공하지 않는 것처럼 보였다. (유일하게 토스만 텍스트 크기변화에 대응이 되었다.)
사용자가 설정한 텍스트 크기에 따라 여러가지 레이아웃을 만드는 것도 상당한 비용이 드는 일인 것 같다..😅 그래도 작은 글씨가 잘 보이지 않는 사람들을 위해 대응하게끔 조금씩 고려한다면 더 멋진 앱이 만들어 질 것 같다. 

##  <span style="color:#007bff">Action sheets</span>
> 사람들이 시작하는 액션과 관련된 선택들을 모달 뷰로 표시한다.

### Best practices
- 의도적인 수행과 관련있는 것은 alert가 아닌 action sheet를 사용한다.
- 사용자들이 현재 작업에 집중할 수 있게 action sheet를 드물게 사용한다
- 짧은 제목을 갖게한다.
- 필요한 경우에만 메세지를 제공한다
- 취소버튼을 제공한다.
- 삭제 버튼과 같은 선택들은 눈에 띄게 한다.

##  <span style="color:#007bff">Alerts</span>
> 문제발생에 대한 중요한 정보를 바로 제공한다.

### Best practices
- 단지 정보를 제공한다는 이유로 alert를 사용하지않는다.
- 평범한 삭제 행위에 대해서는 alert를 띄우지 않는다.(예상치 못한 부분에서 삭제행위가 일어날 때 사용)
- 앱 시작 시 alert를 띄우지 않는다.

### content
- 버튼은 2개를 사용하는 것을 선호한다.
- 정보를 전달해야 한다면 message에 작성한다.
- 텍스트 필드를 포함시킬 수 있다.

### Alert Buttons
- "Yes", "No"와 같은 표현을 보다는 "OK", "Cancel"을 사용한다.
- 일반적으로 사람들은 Cancel이 왼쪽, OK는 오른쪽에 오길 기대하기 때문에 이에 맞게 배치한다.
- 만약 alert가 정보성 alert라면 OK보다는 더 구체적인 "Erase", "Convert"같은 어휘를 사용한다.

### Action sheet VS Alert
- 단순히 정보를 전달해야하는 경우 Action Sheet을 사용하고 예상치 못한 문제에 대해서 경고를 알려주기를 원할때는 Alert를 사용한다.
- Action Sheet는 focused밖의 화면을 클릭하면 사라지지만 Alert는 사라지지 않는다.

<img src="https://i.imgur.com/5TTnYmE.jpg" width = "500"/>
<br><br>


왼쪽 사진은 alert, 오른쪽 사진은 action sheet이다.
사용자 설정을 통해 해당 앱에 정보를 제공하지 않는다면 사용하지 못하는 것을 알려줄 때는 alert를 현재 상태에 대해 추가적인 작업을 하거나 작업을 취소할 때는 action sheet를 사용한 것을 볼 수 있다.

##  <span style="color:#007bff">Pull-down buttons</span>
> 요소들의 메뉴나 버튼의 목적에 연관된 행동들을 표시한다.

### Best practices
- 하나의 풀 다운 버튼에 모든것을 넣지 않는다.
- 메뉴 길이를 고려한다.(최소 3개 아이템이 들어갔을때 사용하는 것이 좋다)
- 풀다운 메뉴를 통해 값이 삭제된다면 action sheet와 같은 모달 뷰를 띄워준다.
- SF Symbol과 같은 icon을 통해 값을 제공한다.


### 핵심<br>
메인 인터페이스안에서 꼭 눈에띄는 위치에 있을 필요가 없는 경우에 More pull-down button을 사용한다.<br>
<img src = "https://i.imgur.com/zYQKAc3.png" width = "300"/><br>

작업목록을 보여주거나(위 사진에서는 Select, NewFolder와 같은 부분) 정렬을 하는 등의 여러가지 일을 할 수 있다. 하위 메뉴도 제공 가능하다.

###  <span style="color:#007bff">Popup</span>
- 기본값을 제공한다
- 레이블을 통해 버튼을 누르지않고 내용값을 예상할 수 있도록 한다.
- 필요하다면 메뉴에 추가적인 값을 넣을 수 있도록 옵션을 포함한다.

### 결론
Pull-down buttons는 다중으로 선택하거나 작업목록을 넣어줘야할때 사용하고 pop-up buttons는 서로 베타적인 메뉴들을 제공할때 사용한다.

### Reference
- [Swift Language Guide - Property](https://docs.swift.org/swift-book/LanguageGuide/Properties.html)
- [HIG - Action Sheet](https://developer.apple.com/documentation/swiftui/actionsheet)
- [HIG - Alert](https://developer.apple.com/design/human-interface-guidelines/components/presentation/alerts)
- [HIG - Pull-down buttons](https://developer.apple.com/design/human-interface-guidelines/components/menus-and-actions/pull-down-buttons)
- [HIG - pop up](https://developer.apple.com/design/human-interface-guidelines/components/menus-and-actions/pop-up-buttons)
- [HIG - Action Sheet](https://developer.apple.com/documentation/uikit/windows_and_screens/getting_the_user_s_attention_with_alerts_and_action_sheets)

###### tags: `Enum`, `HIG`, `Layout`, `Action Sheet`, `Alert`, `Pull-down buttons`, `pop up`

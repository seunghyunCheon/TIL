230228 Self-Sizing Table View Cell - 2, WWDC - Accessibility Insepector, WWDC - Writing Great Accessibility Labels
===
학습내용
---
- 커스텀 셀 오토레이아웃 적용하기
- WWDC - Accessibility Insepector
- WWDC - Writing Great Accessibility Labels

## 커스텀 셀 오토레이아웃 적용하기
기존 프로젝트에서 스택 뷰 두개를 사용해서 커스텀 셀을 구현했는데 최대한 스택뷰 없이 오토레이아웃에 대한 이해를 높이고 싶어서 두개중 하나를 빼서 제약을 설정했다.

![](https://i.imgur.com/ByOqxA9.png)

기존에는 Item Image View와 Stack View를 하나로 묶어서 스택뷰를 만들었다. 이 때문에 내부 스택뷰의 content가 커지더라도 외부의 스택뷰가 그에 맞게 커지고 그로 인해 셀의 높이도 자동으로 커졌다.

### 외부 스택뷰를 지우고 첫 번째로 해결한 방법은 Item Image View의 Y좌표를 CenterY제약을 주는 것이다.

이미지 뷰를 상수로 준 것이아닌 contentView의 width의 0.2배, 이미지뷰 width와 height를 1:1로 주었다. 

처음에는 이미지 크기에 제약을 준 뒤 top과 bottom을 contentView에 constant 0을 주어 제약을 걸었는데 이렇게 되면 이미지 뷰가 contentView bottom에 딱 맞게된다. 이는 옆의 stackView의 내용이 짧다면 문제가 되지 않는다. 

<img src="https://i.imgur.com/fnQ173U.png" width="400"/><br/>

하지만 내용이 긴 stackView라면 bottom 제약이 ContentView의 bottom과 동일하기에 아래쪽의 내용만 보여지게 된다.(이미지뷰가 bottom에 딱 맞으면 그게 contentView의 bottom이 되고 스택뷰는 이 bottom제약에 맞게되어 짤린다)

<img src="https://i.imgur.com/JzwK8PT.png" width="400"/><br/>

음... top과 bottom을 줘서 해결해보려했는데 일단은 이 두가지 제약을 없애고 이미지뷰의 Y좌표를 CenterY를 줘서 해결했다. 이미지에 centerY 제약을 설정함으로써 bottom이 얼마나 떨어져있는지 몰라도 뷰의 크기를 그릴 수 있게 된다. 

### 외부 스택뷰를 지우고 두 번째로 해결한 방법은 Item Image View의 Bottom제약을 ContentView Bottom보다 작거나 같도록 설정하는 것이다.

처음에는 imageView의 bottom제약이 contentView의 Bottom과 동일하게 주어서 실패했다. (위 직지 사진처럼 옆의 스택뷰가 이미지뷰 bottom에 의존하는 고정된 bottom을 가지고 있다.)

#### 그럼 동일하게 말고 같거나 작도록 하면 어떨까?

<img src="https://i.imgur.com/yaCX8Ye.png" width="400"/><br/>
이미지 뷰의 Bottom 제약을 잡음과 동시에 contentView의 bottom은 고정시키지 않는 방법이다. 이렇게되면 스택뷰의 내용에 따라 contentView의 Bottom 제약이 설정되어 문제를 해결할 수 있다.

## WWDC - Accessibility Insepector

Accessibility Inspector는 앱 내에서 접근성 문제를 찾고 진단하고 수정할 수 있는 방법을 제공한다.

Xcode -> Open Developer Tool -> Accessibility Inspector로 들어갈 수 있다.

저시력자들이 요소를 식별할 수 있도록 VoiceOver기능이 잘 되어있는지 확인하거나 동적텍스트 변경인 Dynamic Type을 검사할 수 있다.

Accessibility Inspector에는 세 가지 탭이 존재한다.

먼저 가운데 탭인 Audit은 감사하는 탭이다. 이 탭에서 감사버튼을 누르면 잠재적인 접근성 문제들을 표시해준다.

첫번째 탭에서는 요소가 어떤 접근성레이빌을 갖고있고 어떤 타입을 갖고있는지 확인할 수 있는 탭이다. 이 곳에서 VoiceOver를 검사할 수 있다. 

만약 특정 이미지의 접근성 레이블이 camera.on.rectangle이라면 사용자에게 좋지않은 경험이 줄 것이다. 네모난 카메라 보다는 "사진을 찍다." 라는 버튼을 알려주는 label이라면 더 직관적으로 이해할 수 있을 것이다.
그래서 이를 다음의 코드로 바꿀 수 있다.

```swift
cameraButton.accessibilityLabel = "Take a photo"
```

UIkit을 이용하는 객체에 대해서는 이렇게 바로 접근성레이블을 사용할 수 있지만 CATextLayer와 같이 UIKit이 아닌 경우로 텍스트를 그릴 경우에는  자체적으로 접근성을 처리해야 한다. 

```swift
branTitleBackghroundView.isAccessibilityElement = true
if let brandName = textLayer.string as? String {
    brandTitleBackgroundView.accessibilityLabel = brandName
}
```

명암비에 대해서 접근성을 처리할 수도 있다. 명암비가 맞지않은 컬러의 경우 audit했을때 try adjusting either the text color or the background color so the contrast ratio is above 3.0.과 같은 경고문구를 볼 수 있을것이다. 

이는 정보를 명확하고 읽기 쉬운 방식으로 표시하는 것이 중요하기 때문에 알려주는 문구이다.

권장 명암비를 맞추기 위해 Accessibility Inspector에는 색상 대비 계산기라는 도구가 존재한다. 

Accessibility Inspector에서 Window -> Show Color Contrast Calculator에 들어가면 다음의 창이 보이게 된다.

<img src="https://i.imgur.com/HfyvoSo.png" width="500"/>

왼쪽에 텍스트컬러를 오른쪽에 백그라운드컬러를 넣어서 명암비가 3.0이상으로 맞게끔 수정해주고 수정한 색으로 코드에서 적용하면 해결이 된다.



## WWDC - Writing Great Accessibility Labels

접근성 레이블은 사람이 이해하고 읽을 수 있는 형태로, 앱의 요소에 대한 컨텍스트와 의미를 제공하는 레이블이다.

접근성레이블을 설정하는 Best practice는 다음과 같다.
### 앱 버튼에 레이블을 설정한다.

레이블을 추가하지 않은 경우 VoiceOver로 들었을때 Button, Button, Button만 들리게 될 것이다. 사용자가 잘 들을 수 있도록 레이블을 추가해준다.

### 레이블 안에 요소타입을 포함하지 않는다.

Accessibility Inspector는 요소의 타입을 알고있다. 그래서 아래 코드는
```swift
button.accessibilityLabel = "Add button"
```
"Add button button"이라고 읽는다.
그래서 아래처럼 수정해야 한다.
```swift
button.accessibilityLabel = "Add"
```
### UI를 변경하는 요소에는 접근성레이블을 Update한다.
VoiceOver가 올바른 상태를 말할 수 있도록 UI를 업데이트 해야한다.
```swift
addRemoveButton.accessibilityLabel = "Add"
```

### 충분한 문맥을 제공한다.
쇼핑몰에서 땅콩 버터를 추가하는 버튼이 있는 경우 아래와 같이 사용한다.
```swift
// worse
button.accessibilityLabel = "Add"

// better
button.accessibilityLabel = "Add peanut Butter"
```

### 중복을 피한다.
아래는 뮤직 앱 코드의 예시이다. song이라는 단어가 존재하지않아도 문맥을 충분히 이해할 수 있기 때문에 중복되는 레이블을 제거한다.
```swift
// warning or worse
prevButton.accessibilityLabel = "Previous song"
playButton.accessibilityLabel = "Play song"
nextButton.accessibilityLabel = "Next song"

// better
prevButton.accessibilityLabel = "Previous"
playButton.accessibilityLabel = "Play"
nextButton.accessibilityLabel = "Next"
```

### 의미있는 애니메이션에 레이블을 추가한다.
로딩중인 애니메이션의 경우에도 레이블을 추가하면 좋다.
```swift
spinner.accessibilityLabel = "Loading..."
```

### 장황하게 말하는 것을 피한다.
사용자가 알아야 할 레이블이 아니라면 제거하여 간략하게 나타낸다.
```swift
button.accessibilityLabel = "Delete item from the current folder and add it to the trash"
button.accessibilityLabel = "Delete"
```

### 장황하지만 적절한 경우에는 길게 사용한다.
스티치 앱에서는 웃긴 표현을 길게 사용해서 제공하고 있다.





### 참고자료
- [WWDC - Accessibility Inspector](https://developer.apple.com/videos/play/wwdc2019/257/)
- [WWDC - Writing Great Accessibility Labels](https://developer.apple.com/videos/play/wwdc2019/254/)

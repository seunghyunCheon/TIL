230105 UILabel, UIButton, UISlider, AttributedString, intrinsicContentSize, sizeToFit, sizeThatFits
===
TIL (Today I Learned)
===
학습내용
---
- UILabel, UIButton, UISlider 학습하기
- AttributedString, NSMutableAttributedString 학습하기
- UILabel 텍스트가 잘리는 현상 

## 고민한 점 / 해결 방법
## [boostCourse - ios]
- option + command + o를 이용하면 빠르게 찾는 파일에 접근이 가능하다.
### Control
UIkit에는 UIButton, UISwitch, UIStepper 등 UIControl을 상속받은 다양한 컨트롤 클래스들이 있다. 이들에게서 발생한 특정 이벤트 종류를 특정 액션 메서드에 연결시킬 수 있다. 이를 target-action mechanism이라고 함.

### UIButton
### 버튼의 상태
- 버튼의 상태는 default, highlighted, focused, selected, disabled 총 5가지 상태가 있다.
- 사용자가 버튼과 상호작용을 하면 버튼의 상태가 변한다.

### 버튼 주요 프로퍼티
- `enum UIButtonType`: 버튼의 유형
    - 버튼의 유형에 따라 버튼의 기본적인 외형과 동작이 달라진다.
    - 한번 생성된 버튼의 유형은 이후 변경이 불가하다.
    - 가장 많이 사용하는 유형은 system, custom 
- `titleLabel: UILabel?`: 버튼 타이틀 레이블
- `imageView: UIImageView?`: 버튼 이미지 뷰
- `tintColor: UIColor!`: 버튼 타이틀과 이미지의 틴트 컬러

### 버튼 주요 메서드
```swift
    // 특정 상태의 버튼의 문자열 설정
    func setTitle(String?, for: UIControlState)
    // 특정 상태의 버튼의 문자열 반환
    func title(for: UIControlState) -> String?
```

### UILabel
> 한 줄 또는 여러 줄의 텍스르를 보여주는 뷰로, UIButton등의 컨트롤 객체의 목적을 설명하는 용도로 주로 쓰인다.

#### UILabel 주요 프로퍼티
- 레이블 프로퍼티에 접근해 문자의 내용, 색상, 폰트, 문자정렬방식, 라인 수를 조정할 수 있다.
- `var text: String?`: text 프로퍼티에 값을 할당하면 attributed 프로퍼티에도 같은 내용의 문자열이 할당
- `var textAlignment: NSTextAlignment`: 문자열의 가로 정렬 방식(left, right, center, justified, natural 중 하나 선택)
    - justified: line이 변경되었을 때 행 맞추는 속성
    - natural: 지역화 기준으로 정렬
- `var baselineAdjustment: UIBaselineAdjustment`: 세로 정렬을 할 때 사용하는 프로퍼티. Align Baseline, Align Center, None이 존재한다.
- `var lineBreakMode: NSLineBreakMode`: 레이블의 경계선을 벗어나는 문자열에 대응하는 방식
    - Character wrap: 여러 줄 레이블에 적용. 글자 단위로 줄 바꿈
    - Word wrap: 여러 줄 레이블에 적용. 단어 단위로 줄 바꿈
    - Truncate (head, middle, tail): 한 줄에 적용하고 속성 해당 위치를 자르고 ...로 표시
- `var attributedText: NSAttributedString?`: 레이블이 표시할 속성 문자열
```swift
// NSAttributedString 사용예시
class ViewController: UIViewController {

    @IBOutlet weak var attributedStringLabel: UILabel!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        attributedStringLabel.attributedText = NSMutableAttributedString()
            .redHighLight("빨간색강조")
            .bold(string: "굵은 글씨", fontSize: 14)
    }
}

extension NSMutableAttributedString {
    var fontSize: CGFloat {
        return 14
    }
    
    var boldFont: UIFont {
        return UIFont.boldSystemFont(ofSize: fontSize)
    }
    
    var normalFont: UIFont {
        return UIFont.boldSystemFont(ofSize: fontSize)
    }
    
    func bold(string: String, fontSize: CGFloat) -> NSMutableAttributedString {
        let attributes: [NSAttributedString.Key: Any] = [.font: boldFont]
        self.append(NSAttributedString(string: string, attributes: attributes))
        return self
    }
    
    func redHighLight(_ value: String) -> NSMutableAttributedString {
        let attributes: [NSAttributedString.Key: Any] = [
            .font: normalFont,
            .foregroundColor: UIColor.white, // 글자 색
            .backgroundColor: UIColor.red // 배경 색
        ]
        self.append(NSAttributedString(string: value, attributes: attributes))
        return self
    }
}
```
#### 화면
<img src="https://i.imgur.com/HK4KAlT.png" width="300"/><br>

- AttributedString를 적용할 range를 정하는 방법이 아닌 `self.append`를 통해 유동적으로 NSMutableAttributedString를 만들어 붙이는 방법
- 빨간색으로 강조되는 부분과 굵은 글씨를 적용해야 하는 부분이 일정하지 않을때(서버에서 온 데이터가 다를때) 사용하면 좋을 것 같다. 

---------
> TextAlignment의 justified속성을 테스트하다가 레이블의 CGSize를 설정하지않은 상태에서 텍스트 크기를 늘렸더니 텍스트가 잘리는 현상이 발생했다.
### 왜 잘려서 출력되지? 🤔
- 일반적으로 Custom View(정의한 UILabel과 같은)는 layout system이 인식하지 못하는 내용을 표시한다. 자기 사이즈가 변경되었음을 동적으로 소통할 방법이 없기 때문인데 소통을 하는 방법이 intrinsicContentSize속성을 이용하는 것이다.

#### 1. intrinsicContentSize이용
```swift
var intrinsicContentSize: CGSize { get }
```
- intrinsicContentSize는 뷰의 컨텐츠 본질적인 사이즈를 나타내는 속성이다. 하지만 모든 view들이 이 속성을 가지고 있지는 않고 UILabel, UIButton, UISwitch 등 몇 가지 컴포넌트들만 가지고 있다.
- 결론은 상당한 양의 텍스트를 가지는 레이블의 intrinsicContentSize에 접근해 레이블의 size를 이 고유 사이즈로 설정하게 된다면 텍스트가 짤려서 보이는 일을 막을 수 있다.
#### 2. sizeToFit메서드 이용
- 또 다른 방법으로는 sizeToFit메서드를 이용하면 된다.
```swift
// UIView를 상속받은 메서드
extension UIView {
    open func sizeToFit()
}
```
> receiver view가 하위 view들을 둘러싸도록 크기를 조정하고 움직인다.
- UILabel view가 텍스트의 양 만큼 둘러싸도록 크기를 조정하고 움직이는 것과 같은 말로 해석된다.
- 하지만 numberOfLines를 0으로 설정한다면 가로로 길어지는 게 아닌 세로로 늘어난다.(numberOfLines를 0으로 설정하면 텍스트가 찰 때 줄바꿈이 계속 발생) -> 세로인지 가로인지 알려주지 않았기 때문

#### 3. sizeThatFits메서드 이용
```swift
func sizeThatFits(_ size: CGSize) -> CGSize
```
> 지정한 크기에 가장 적합한 크기를 계산하여 반환하도록 뷰에 요청한다.
- 파라미터로 넣은 사이즈에 맞게 CGSize를 새롭게 계산해서 리턴해주는 방식

```swift
// mainLabel의 size에 새롭게 계산된 size를 넣어 텍스트 짤림 현상을 방지한다.
@IBOutlet weak var mainLabel: UILabel!
mainLabel.text = "많은 레이블, 많은 레이블, 많은 레이블, 많은 레이블"
mainLabel.numberOfLines = 0
mainLabel.frame.size = mainLabel.sizeThatFits(view.frame.size)
```
- 재밌는 사실은 numberOfLines를 앞에 적어줘야 sizeThatFits함수를 적용할 때 줄바꿈이 일어난 다는 것이다. (줄바꿈 속성을 고려한 상황에서 레이아웃 계산을 하는 것으로 추측)

#### 4. adjustFontSizeToFitWidth 프로퍼티 이용
- 사실 이건 텍스트에 맞게 UILabel의 영역을 넓히는 것이 아닌 그 반대로 UILabel의 bounding rectangle안으로 텍스트 크기를 줄일지 여부를 결정하는 것이다. (나름 유용해보인다.)

### UISlider
> 연속된 값 중에서 특정 값을 선택하는데 사용되는 컨트롤러

- 값의 범위를 지정하고, 색상과 이미지를 이용해 모양 구성, 메서드를 슬라이더와 연결
#### 슬라이더 주요 프로퍼티
- `var minimumValue: Float, var maximumValue: Float`: 슬라이더 양끝단의 값
- `var value: Float`: 슬라이더의 현재 값
- `var isContinuous: Bool`: 슬라이더의 연속적인 값 변화에 따라 이벤트 역시 연속적으로 호출할 것인지의 여부
- `var minimumValueImage: UIImage?, var maximumValueImage: UIImage?:`` 슬라이더 양끝단의 이미지



### Reference
- [boostCourse - 인터페이스빌더](https://www.boostcourse.org/mo326/lecture/16847/?isDesc=false)
- [AttributedString](https://ios-development.tistory.com/359)
- [UILabel Size](https://doorganizedcoding.tistory.com/entry/UILabel-size%EB%A5%BC-Text-size%EC%97%90-%EB%A7%9E%EC%B6%94%EB%8A%94-%EB%B0%A9%EB%B2%95%ED%98%B9%EC%9D%80-%EA%B7%B8-%EB%B0%98%EB%8C%80)
- [Intrinsic Content Size](https://hyunsikwon.github.io/ios/iOS-AutoLayout-02/)
- [Apple Developer Documentation - SizeToFit](https://developer.apple.com/documentation/uikit/uiview/1622630-sizetofit)
- [Apple Developer Documentation - SizeThatFits](https://developer.apple.com/documentation/uikit/uiview/1622625-sizethatfits)


###### tags: `UIButton`, `UILabel`, `UISlider`, `intrinsicContentSize`, `sizeToFit`, `sizeThatFits`, `adjustFontSizeToFitWidth`, `AttributedString`

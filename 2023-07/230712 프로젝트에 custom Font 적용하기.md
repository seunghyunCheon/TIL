## 230712 프로젝트에 custom Font 적용하기

### 개요
폰트가 포함된 폰트파일을 앱 번들에 추가해서 사용한다.

### 폰트파일을 Xcode Project에 추가

File > Add Files로 파일을 추가하거나 Xcode Project에 드래그한다. ttf나 otf 파일을 추가할 수 있다. 폰트 파일은 앱에 대한 타겟 멤버십을 체크해야 한다. 

## 폰트 파일 등록

폰트 파일을 추가한 후에 iOS가 이 폰트를 알도록 해야 한다. 이를 위해 info.plist의 `Fonts provided by application` key를 추가한다. Xcode는 이 키를 위한 값으로 여러 개의 배열을 제공하는데 이곳에 폰트파일 이름을 적으면 된다. 파일 확장자를 적어야 하는 것을 인지하자.

## 코드에서 사용하기

```swift
extension UIFont {
    enum Family: String {
        case gmarketSans
    }
    
    enum CustomWeight: String {
        case light = "Light"
        case medium = "Medium"
        case bold = "Bold"
    }
    
    enum Size: CGFloat {
        case largeTitle = 33
        case title1 = 27, title2 = 21, title3 = 19
        case body = 16
        case callout = 15
        case subheadLine = 14
        case caption = 11
    }
    
    private class func stringName(_ family: Family, _ weight: CustomWeight) -> String {
        return "\(family.rawValue)TTF\(weight.rawValue)"
    }
}

extension UIFont {
    static func gmarketSans(size: Size, weight: CustomWeight) -> UIFont {
        guard let customFont =  UIFont(name: UIFont.stringName(.gmarketSans, weight), size: size.rawValue) else {
            return UIFont.systemFont(ofSize: size.rawValue)
        }
        
        return UIFontMetrics.default.scaledFont(for: customFont)
    }
}
```

### 참고문서
- [Apple Docs - Adding a Custom Font to Your App](https://developer.apple.com/documentation/uikit/text_display_and_fonts/adding_a_custom_font_to_your_app)

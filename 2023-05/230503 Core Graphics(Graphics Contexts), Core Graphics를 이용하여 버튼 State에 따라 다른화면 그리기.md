# 230503 Core Graphics(Graphics Contexts), Core Graphics를 이용하여 버튼 State에 따라 다른화면 그리기

학습내용
- CoreGraphics
- Graphics Context
- Core Graphics를 이용하여 버튼 State에 따라 다른화면 그리기

## CoreGraphics
Quartz기술의 힘으로 가벼운 2D렌더링을 수행하는 FrameWork. 경로 기반 그리기, antialiased 렌더링, 그라데이션, 이미지, 색상 관리, PDF 문서 등을 처리한다.

### OverView
Core Graphics 프레임워크는 drawing엔진을 사용하는 Quartz기반이다. 이는 저수준의 가벼운 2D렌더링을 제공한다. 
이 프레임워크를 사용해서 경로 기반 그리기, 변형, 색상관리 등을 관리할 수 있다.

macOS에서 코어 그래픽스는 저수준의 사용자 입력이벤트나 windowing system과 같은 하드웨어와 함께 동작하는 서비스를 포함한다.

## Graphics Contexts
그래픽 컨텍스트는 drawing 목적지를 표현한다. 여기에는 드로잉 시스템이 후속 드로잉 명령을 수행하는 데 필요한 드로잉 매개변수 및 모든 장치별 정보가 포함되어 잇다. 
그래픽 컨텍스트는 그릴 때, 영역을 자를때, 선의 width와 style을 정할때, 폰트 정보를 줄 때 등에 사용되는 기본 드로잉 속성을 정의한다.

Quartz contet 생성함수를 사용하거나 macOS 프레임워크나 iOS의 프레임워크인 UIKit프레임워크를 사용함으로써 제공되는 고수준의 함수를 사용함으로써 그래픽 컨텍스트를 얻을 수 있다.
Quartz는 bitmap, PDF, 커스텀 content를 만들고 사용할 때 쓰는 등을 포함하여 다양한 종류에 대한 기능을 제공한다. 

즉 다양한 목적지에 도달하는 그래픽스 컨텍스트를 만들 수 있는 것이다. 그래픽스 컨텍스트는 코드에서 `CGContextRef`에 의해 표현된다. 이는 불투명 타입이다. 
그래픽스 컨텍스트를 얻은후에 Quartz 2D함수를 이용하여 content를 만들고 graphics state 파라미터를 사용하여 선의 스타일과 같은 부분들을 수정할 수 있다.

### Drawing to a View Graphics Context in iOS

iOS에서 화면에 그리기위해서 `UIView`객체를 설정하고 `drawRect:`메서드를 구현해야 한다. 그 view의 `drawRect:`메서드는 뷰가 화면에 보여지거나 업데이트 될 때 호출이 된다. 
`drawRect:`메서드를 호출하기 전에 뷰 객체는 자동적으로 드로잉 환경을 설정해서 즉시 그리기를 시작한다. 이 설정의 부분으로 `UIView`객체는 현재 드로잉 환경에 맞춰 그래픽스 컨텍스트(`CGContextRef`라는 불투명 타입의)를 만든다. `UIKit`함수인 `UIGraphicsGetCurrentContext`를 호출함으로써 `drawRect`메서드 안에서 이 그래픽스 컨텍스트를 얻는다. 

UIKit의 좌표계는 Quartz좌표계와 다르다. 
UIKit은 왼쪽위부터 아래를 향할수록 양수가 된다. `UIView`객체는 Quartz 그래픽스 컨텍스트의 CTM을 수정하여 UIKit 컨벤션에 맞춘다. 

## Core Graphics를 이용하여 버튼 State에 따라 다른화면 그리기
- 뷰컨에서 버튼 클릭했을 때 selected 변경
- 변경된 selected는 tintColor를 변경시켜, `draw`메서드 호출.
- `isSelected`변수에 따라 `context`다르게 구성

```swift
// ViewController.swift
@IBAction func ButtonTapped(_ sender: Button) {
    if button.isSelected {
        button.isSelected = false
    } else {
        button.isSelected = true
    }
}

// Button.swift
class Button: UIButton {
    override var isSelected: Bool {
        didSet {
            self.tintColor = .white
        }
    }
    
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else {
            return
        }
    
        let circleRect = bounds.insetBy(dx: bounds.height * 0.1, dy: bounds.width * 0.1)
        
        //원 그리기
        context.beginPath()
        context.setLineWidth(5)
        context.setStrokeColor(UIColor.black.cgColor)
        context.setFillColor(UIColor.systemBackground.cgColor)
        context.addEllipse(in: circleRect)
        context.drawPath(using: .fillStroke)
        context.closePath()
        
        if self.isSelected {
            //체크 표시 그리기
            context.beginPath()
            context.setLineWidth(15)
            context.setLineJoin(.round)
            context.setLineCap(.square)
            context.move(to: CGPoint(x: bounds.width * 0.15, y: bounds.height * 0.5))
            context.addLine(to: CGPoint(x: bounds.width * 0.4, y: bounds.height * 0.8))
            context.addLine(to: CGPoint(x: bounds.width * 0.8, y: bounds.height * 0.3))
            context.drawPath(using: .stroke)
        }
    }
}
```

### 알게된 점
- `UIButton`에서 select되었을 때 배경이 systemBlue가 되는 것이 `BackgroundColor`인줄 알았는데 tintColor였다. 

### 참고문서
- [Apple Docs - drawRect](https://developer.apple.com/documentation/uikit/uiview/1622529-drawrect)
- [Apple Archive - Core Graphics](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/drawingwithquartz2d/Introduction/Introduction.html)
- [Core Graphics](https://green1229.tistory.com/73)

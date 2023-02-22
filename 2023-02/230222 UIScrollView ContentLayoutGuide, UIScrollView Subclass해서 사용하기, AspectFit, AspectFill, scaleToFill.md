230222 UIScrollView ContentLayoutGuide, UIScrollView Subclass해서 사용하기, AspectFit, AspectFill, scaleToFill
TIL (Today I Learned)
===
학습내용
---
- UIScrollView ContentLayoutGuide
- UIScrollView Subclass해서 사용하기
- AspectFit, AspectFill, scaleToFill
## UIScrollView ContentLayoutGuide
> 스크롤 뷰의 컨텐츠에 기반하는 레이아웃 가이드

```swift
var contentLayoutGuide: UILayoutGuide { get }
```
스크롤 뷰의 컨텐츠 영역과 관련된 오토레이아웃을 설정할 때 이 레이아웃을 사용한다.

스크롤 뷰의 ContentLayoutGuide를 사용해보기 전에 스크롤 뷰에서 가장 중요한 컨텐츠의 사이즈를 정하는 방법 4가지를 먼저 알아보자.

### ✏️ ContentSize Property 활용(외부에서 내부로)
UIScrollView 내부 contentSize 프로퍼티에 CGSize를 설정하면 UIScrollView의 내부 contents의 사이즈가 정해진다.

```swift
let scrollView: UIScrollView(frame: self.view.bounds)
scrollView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
self.view.addSubView(scrollView)
scrollView.backgroundColor = .white
let imageView = UIImageView()
imageView.contentMode = .scaleAspectFit
imageView.image = UIImage(named: "woookdev")
scrollView.addSubView(imageView)
imageView.frame = CGRect(x: 0, y: 0, width: 800, height: 2000)

// 직접 contentSize 정해서 주기
scrollView.contentSize = CGSize(width: 800, height: 2000)
```

### ✏️ 내부 autolayout을 통한 contentSize 설정
내부 사이즈가 autolayout을 통해 해당 뷰를 subView로 가지는 UIScrollView의 contentSize가 정해지는 방식이다.

간단한 내부를 통해 외부가 정해지는 in-out 방식이고 가장 많이 사용하는 방식이다. 내부에 다른 뷰를 추가해도 contentSize가 커지기 때문에 코드 수정에 용이하다. 

```swift
// scrollView autolayout 설정
scrollView.translatesAutoresizingMaskIntoConstraints = false
NSLayoutConstraint.activate([
    scrollView.topAnchor.constraint(equalTo:self.view.topAnchor),
    scrollView.bottomAnchor.constraint(equalTo:self.view.bottomAnchor),
    scrollView.leadingAnchor.constraint(equalTo:self.view.leadingAnchor),
    scrollView.trailingAnchor.constraint(equalTo:self.view.trailingAnchor),
])

// in-out 방식으로 layout 설정되도록 imageView autolayout 설정 (height, width 정해줘야 contentSize 정해짐)
imageView.translatesAutoresizingMaskIntoConstraints = false
NSLayoutConstraint.activate([
    imageView.topAnchor.constraint(equalTo:scrollView.topAnchor),
    imageView.leadingAnchor.constraint(equalTo:scrollView.leadingAnchor),
    imageView.bottomAnchor.constraint(equalTo:scrollView.bottomAnchor),
    imageView.trailingAnchor.constraint(equalTo:scrollView.trailingAnchor),
    imageView.heightAnchor.constraint(equalToConstant: 2000),
    imageView.widthAnchor.constraint(equalToConstant: 800)
])
```

### iOS 11부터 제공하는 contentLayoutGuide
내부의 autolayout을 통해 외부의 scrollView와 제약을 설정하다보니 이게 UIScrollVIew에 보여지는 frame과의 관계를 설정하는 건지 내부 bounds와 설정하는 건지 혼란스러울 수 있다. 이 문제를 해결하기 위해 iOS11이후로 contentLayoutGuide를 제공했다. 이는 보여지는 scrollView가 아닌 내부 사이즈와 관계를 맺는 layoutGuide으로 생각하고 사용하면 된다.

```swift
scrollView.translatesAutoresizingMaskIntoConstraints = false
NSLayoutConstraint.activate([
    scrollView.topAnchor.constraint(equalTo:self.view.topAnchor),
    scrollView.bottomAnchor.constraint(equalTo:self.view.bottomAnchor),
    scrollView.leadingAnchor.constraint(equalTo:self.view.leadingAnchor),
    scrollView.trailingAnchor.constraint(equalTo:self.view.trailingAnchor),
])

// 기존 autolayout 설정을 contentLayoutGuide를 활용하도록 수정
imageView.translatesAutoresizingMaskIntoConstraints = false
NSLayoutConstraint.activate([
    imageView.topAnchor.constraint(equalTo:scrollView.contentLayoutGuide.topAnchor),
    imageView.leadingAnchor.constraint(equalTo:scrollView.contentLayoutGuide.leadingAnchor),
    imageView.bottomAnchor.constraint(equalTo:scrollView.contentLayoutGuide.bottomAnchor),
    imageView.trailingAnchor.constraint(equalTo:scrollView.contentLayoutGuide.trailingAnchor),
    imageView.heightAnchor.constraint(equalToConstant: 2000),
    imageView.widthAnchor.constraint(equalToConstant: 800)
])
```
### ✏️ iOS 11부터 제공하는 frameLayoutGuide 활용
Floating 버튼과같은 요소를 오토레이아웃 제약을 설정할 때는 FrameLayoutGuide에 맞춘다.

## AspectFit, AspectFill, scaleToFill

### ✏️ Content Mode
> 컨텐츠 사이즈가 변경되면서 어떻게 뷰가 조절될지 명시하는 옵션

### ScaleAspectFit
> 비율 유지 O, 화면 꽉 채움O, 이미지 잘림 O


비율을 유지하면서 뷰의 사이즈에 맞게 이미지를 늘리는 옵션. 이미지가 뷰를 꽉채우지 못해 남는 부분은 투명처리 된다.

### scaleAspectFill
> 비율 유지 O, 화면 꽉 채움X, 이미지 잘림 X

비율을 유지하면서 뷰의 사이즈에 맞게 이미지를 꽉 채우는 옵션. 그래서 이미지의 어떤 부분은 잘려 보일 수 있다.

### scaleToFill
> 비율 유지 X, 화면 꽉 채움O, 이미지 잘림 X

전체 이미지가 다 나올 수 있도록 필요하다면 비율을 꺠뜨리면서 뷰의 사이즈에 맞게 이미지를 꽉 채우는 옵션

### 참고자료
- [WoookDev님 블로그 - UIScrollView contentSize, layout 설정 탐구하기](https://woookdev.github.io/programming/UIScrollView-contentSize,-layout-%EC%84%A4%EC%A0%95-%ED%83%90%EA%B5%AC%ED%95%98%EA%B8%B0-(+-contentLayoutGuide,-frameLayoutGuide-)/) 
- [nalJin님 블로그 - 사진으로 보는 AspectFit, AspectFill, ScaleToFill](https://sujinnaljin.medium.com/ios-%EC%82%AC%EC%A7%84%EC%9C%BC%EB%A1%9C-%EB%B3%B4%EB%8A%94-aspectfit-aspectfit-scaletofill-f41470d0191f)
- [Apple Delveloper Docuemntation - ContentMode](https://developer.apple.com/documentation/uikit/uiview/contentmode)

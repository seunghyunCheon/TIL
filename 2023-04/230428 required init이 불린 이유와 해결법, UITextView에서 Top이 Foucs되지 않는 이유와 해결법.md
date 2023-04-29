# 230428 required init이 불린 이유와 해결법, UITextView에서 Top이 Foucs되지 않는 이유와 해결법
===
학습내용
- required init이 불린 이유와 해결법
- UITextView에서 Top이 Foucs되지 않는 이유와 해결법

## required init이 불린 이유와 해결법

### 문제점

스토리보드로 UI를 구현한 화면을 코드를 통해 부른 뒤, push하려고 하니 `AddJokeViewController`에서 `required init`이 호출되어 `fatalError`가 불려서 앱이 크래시가 났다.

```swift
// ViewController.swift
guard let addJokeViewController = self.storyboard?.instantiateViewController(
    identifier: "AddJokeViewController") as? AddJokeViewController else {
    return
}

// AddJokeViewController.swift
init?(delegate: TableDataUpdatable) {
    self.delegate = delegate
}

required init?(coder: NSCoder) {
    fatalError("init(coder:) has not been implemented")
}
```

### 해결방법
위 코드는 스토리보드에 있는 뷰컨을 가져오는 코드이다. 때문에 `AddJokeViewController`파일에는 스토리보드에서 화면을 그리는 경우 `init`이 존재하면 안 된다. 하지만 `init`을 만들어놨기 때문에 예상치 못한 흐름이여서 크래시가 나는 것이다.
> AddJoke 뷰컨에 커스텀 지정생성자가 정의되어 있다는 건 코드로 UI를 그린다는 거 아니야? 근데 왜 메인스토리보드에서 xml파일을 꺼내려고 해? 뭔가 충돌이 있으니까 크래시 낼게😄

위의 이유로 iOS13이전에는 메인스토리보드를 사용하면서 커스텀 init을 사용할 수 없었다. (가능한 방법이 있다고는 하지만 기본적으로 제공하는 것은 아니다.) 

하지만 iOS13이후로 `instantiateViewController(identifier:creator:)`가 나오면서 화면을 스토리보드에서 그리면서 속성값을 커스텀 이니셜라이저를 추가할 수 있게 되었다. 

`creator`는 `NSCoder`타입으로 `AddJokeViewController`를 만들 때 해당클래스의 프로퍼티들을 전부 초기화한후에 `creator`를 `super.init`의 인자로 넣어줌으로써 커스텀이니셜라이저를 구성할 수 있게 만들었다.

```swift
// ViewController.swift
guard let addJokeViewController = self.storyboard?.instantiateViewController(
    identifier: "AddJokeViewController",
    creator: { creator in
    let viewController = AddJokeViewController(
        delegate: self,
        coder: creator)
    return viewController
}) else {
    return
}

// AddJokeViewController.swift
init?(delegate: TableDataUpdatable, coder: NSCoder) {
    self.delegate = delegate
    super.init(coder: coder)
}
```

그나저나 막상 `required init`에서 `fatalError`를 맞이하게 되니 부정적경험이라고 느꼈다. 

따라서 실수로 커스텀이니셜라이저가 있지만 스토리보드에서 부르는 경우에 크래시가 아닌 속성값은 초기화되지 않았지만 메인스토리보드에 있는 뷰를 가져와서 보여주는 것이 더 좋은 경험이라고 생각이 들어 다음과 같이 수정했다.

```swift
required init?(coder: NSCoder) {
    super.init(coder: coder)
}
```

## UITextView에서 Top이 Foucs되지 않는 이유와 해결법
### 문제점
텍스트가 충분히 긴 `textview`가 존재하는 화면에 진입했을때 맨위를 focus하지 않는 현상이 발생했다.

### 해결방법
[stackOvewFlow](https://stackoverflow.com/questions/26942861/uitextview-content-offset-changes-after-setting-frame)에 따르면 스크롤 뷰의 애니메이션과 frame setting이 동시에 일어나기 때문이라고 한다. 

이에 대한 해결방법은 두 가지이다.

### 1. contentOffset을 zero로
스크롤이 일어난 상태에서 frame을 잡기 때문에 아래로 보여지는 것이다. 따라서 `textView`의 `contentOffset`을 0으로 고정한다면 뷰가 그려질 때 내려간 스크롤을 계산하지 않을 것이다.

재밌는 사실은 `contentOffset`이 `viewDidLoad`에는 0이지만 `ViewDidLayoutSubviews`에서는 0이 아닌 것이다. 뷰가 실제로 그려지면서 내부적으로 계산되는 게 있는 것 같다. 내부 계산이 어찌되었든 `contentOffset`을 zero로 두면 스크롤이 되어도 맨 위를 고정시킬 수 있다.
```swift
diaryTextView.contentOffset = .zero
```

### 2. scroll을 비활성화
스크롤과 frame setting이 동시에 잡혀서 생기는 문제기 때문에 frame setting전에 스크롤을 비활성화 시키면 된다.
```swift
diaryTextView.isScrollEnabled = false
NSLayoutConstraint.activate([
    diaryTextView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
    diaryTextView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
    diaryTextView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
    diaryTextView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
])
diaryTextView.isScrollEnabled = true
```

### 참고문서
- [소들이 - required Initializer](https://babbab2.tistory.com/171)
- [UITextView ScrollTop에 고정되지 않는 이유](https://stackoverflow.com/questions/26942861/uitextview-content-offset-changes-after-setting-frame)

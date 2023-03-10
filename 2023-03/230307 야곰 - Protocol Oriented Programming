230307 야곰 - Protocol Oriented Programming
===
학습내용
---
- Protocol Oriented Programming

## Protocol Oriented Programming

Objective-C에서의 프로토콜은 단지 기능의 청사진 역할을 했다. 

하지만 Swift에서는 기본구현이 가능해지면서 많은 것이 달라졌다.

- 기본 구현: Protocol + Extension. 이 공간에서 연산프로퍼티나 기본메서드를 제공할 수 있게 되었다.

즉 특정 타입이 할 일과 구현을 동시에 할 수 있게 된 것

#### Protocol Default Implimentation

```swift
protocol LayoutDrawable {
    func drawSomeLayout()
}

extension LayoutDrawable {
    func drawSomeLayout() {
        
    }
}

extension UIView: LayoutDrawable { }
extension SKNode: LayoutDrawable { }
```

프로토콜을 채택한다고 명시만해도 구현이 다 되는 것!

이는 상속의 한계로, 값타입의 필요성으로부터 생겨났다.

- `UITableViewController`와 `UICollectionViewController`는 유사한 기능이 많기 떄문에 하나의 클래스를 상속받게 되는데 이는 동일한 기능을 구현하기 위해 중복코드를 사용하게 된다.
- Objective-C에서는 참조타입에만 카테고리 기능을 사용할 수 있었다. 이 떄문에 값타입인 enum, struct는 확장적인 기능을 사용하지 못했는데, 참조타입은 어디에서나 수정이 될 수 있기에 위험했다. 값 타입도 확장가능하고 구현이 있는 기능이 필요해진 것이다. (+ 참조타입은 동적할당과 참조 카운팅에 많은 자원을 소모함)


### ✏️ 프로토콜 기본구현 활용

테이블 뷰의 타임라인을 요구사항을 받는다면 다음과 같이 정의할 수 있다.

```swift
class TimelineTableViewCell: UITableViewCell {
    var mediaImageView: UIImage
    var note: UILabel
    var content: NSDictionary
}
class DetailViewController: UIViewController {
    var mediaImageView: UIImageView
    var note: UILabel
    var content: NSDictionary
}
class TimelineTableViewController: UITableViewController {
    var contents: [NSDictionary]
}
```

그런데 위 코드에서 사진첩처럼 볼 수 있는 모드를 추가하는 요구사항을 받게 되면 어떤 것을 변경해야 할 것인지를 먼저 고민해봐야 한다.

먼저 모델단 부분에서 프로토콜을 사용할 부분을 생각해보자. 

사진첩에 들어가는 내용과 기존의 테이블뷰컨트롤러에 들어가는 내용이 같다면 content에 대한 부분을 재사용할 수 있다.

```swift
struct Content {
    var URLString: String
    var note: String
}
```

위 모델을 각 뷰컨트롤러에서 프로퍼티로 갖는다.

```swift
class TimelineTableViewController: UITableViewController {
    var contents: [Content]
}
class TimelineCollectionViewController: UITableViewController {
    var contents: [Content]
}
```

위 상황에서 프로토콜 지향 프로그래밍을 위해 contents프로퍼티를 가져야한다는 것을 명시하는 용도로 `ContainsContents`프로토콜을 만들어 채택했다.

```swift
protocol ContainContents {
    var contents: [Content] { get }
}
```

여기서 기본구현을 사용할 수 있다! contents프로퍼티를 채택하는 모든 곳에서 직접 만드는 것대신에 기본구현을 사용하는 것이다. 다음과 같이 말이다.

```swift
class TimelineContentObject {
    static let shared = TimelineContentObject()
    var contents: [Content] = [Content]()
}

extension ContainContents {
    var contents: [Content] {
        return TimelineContentObject.shard.contents
    }
}
```

그래서 관계도는 다음과 같이 그려진다
<img src="https://i.imgur.com/DSHpjdR.jpg" width="500"/>

처음에는 왜 굳이 싱글턴 객체를 사용하는 걸까 생각을 해봤는데 관계로를 보니 책임을 명확하게 나누고 싶은 이유지 않을까 싶었다.

View단에서도 프로토콜을 사용해 재사용화할 수 있다.

```swift
protocol MediaContainer: class {
    var content: Content? { get set }
    var media: UIImageView { get }
    var note: UILabel { get set }
    
    func contentChanged()
}

extension MediaContainer {
    func contentChanged() {
        // update view...
    }
}
```
컨텐츠가 바뀔때 어떻게 변하는지에 대해서 기본 구현을 한 상태이다.
이 프로토콜을 채택해서 다음과 같이 사용할 수 있다.
```swift
class TimelineTableViewCell: UITableViewCell, MediaContainer {
    var media: UIImageView
    var note: UILabel
    var content: Content? {
        didSet {
            contentChanged()
        }
    }
}
```
content에 값이 들어왔을때 `contentChanged()`를 통해 필터 및 특정 기능을 하게끔 만든 것이다. 이때 `contentChanged`는 기본구현이 되어있기 때문에 정의를 하지않아도 된다.(정의한다면 기본구현이 아닌 정의한 부분을 따른다.)

Controller에서는 어떤 부분을 적용할 수 있을까?

테이블뷰와 컬렉션뷰 둘 다 다음화면으로 넘어갈 때 네비게이션 컨트롤러를 이용해 푸시한다는 점이 공통점이다.

이 부분을 프로토콜로 만들면 다음과 같다.

```swift
protocol CanShowDetailView {
    func showDetailView(withContent content: Content)
    var navigationController: UINavigationController? { get }
}

extension CanShowDetailView {
    func showDetailView(withContent content: Content) {
        // show detail view...
    }
}
```

위 프로토콜을 컨트롤러에서 채택만하면 함수명과 인자만 넣어서 커스텀하게 사용할 수 있다.

그런데 사진 말고 영상도 보여달라는 요구사항이 들어온다면 어떻게 프로토콜을 또 적용할 수 있을까?

먼저 이미지뷰가 있던 부분에 비디오도 올 수 있어야 한다.

```swift
protocol MediaContainer: class {
    ...
    var media: UIImageView { get }
}
```
media가 `UIImageView`로 특정되어있기 때문에 문제가 된다. 이를 이미지와 비디오를 받을 수 있는 프로토콜을 정의하면 문제가 해결된다

이를 컨텐츠를 보여줄 수 있는 이라는 뜻을 가진 `ContentPresentable` 프로토콜로 만들어보자. 

```swift
protocol ContentPresentable: class, Layout {
    var frame: CGRect { get set }
    var canPresentContent: Bool { get }
}

extension ContentPresentable {
    var canPresentContent: Bool {
        return true
    }
}

extension UIImageView: ContentPresentable { }
extension AVPlayerLayer: ContentPresentable { }
```

이제 `ImageView`대신에 `ContentPresentable`으로 대체하면서 이를 따르는 타입 아무거나 들어와도 되게 만들었다.

```swift
protocol MediaContainer: class {
    var content: Content? { get set }
    var media: ContentPresentable { get }
    var note: UILabel { get set }
    
    var videoLayer: AVPlayerLayer { get }
    var mediaImageView: UIImageView { get }
    
    func contentChanged()
}
```

그리고 media프로퍼티를 특정 객체(이미지나 비디오)로 만들기 위해 Content 구조체에 다음의 특성을 추가했다.

```swift
struct Content {
    enum MediaType {
        case image, video
    }
    
    var type type: Content.MediaType
    ...
}
```

마지막으로 MediaContainer의 extension으로 위 content의 mediaType에 따라 알맞은 객체를 주도록 만들었다.

```swift
extension MediaContainer {
    var media: ContentPresentable {
        switch Content!.type {
            case .image:
                return mediaImageView
            case .video:
                return videoLayer
        }
    }
}
```

### ✏️ 프로토콜 지향 프로그래밍의 장점
- 범용적인 사용
    - 클래스, 구조체, 열거형에 적용 가능
    - 제네릭과 결합하면 더욱 효과적
- 상속의 한계 극복
    - 특정 상속 체계의 종속되지 않음
    - 프레임워크에 종속적이지 않게 재활용 가능
- 적은 시스템 비용
    - Reference type cost > Value type cose
- 용이한 테스트
    - GUI 코드 없이도 수월한 테스트

### ✏️ 프로토콜 지향 프로그래밍의 한계
Objective-C 프로토콜과 Swift의 Extension을 결합시킬 수 없다. 즉 자주 사용되는 Delegate, DataSource에는 기본구현이 불가하다.

### 결론
- Value Type을 이용해 성능상의 이점을 취하자
- Protocol + Extension + Generic을 사용하면 좋다.
- OOP에서의 수직확장뿐만아니라 POP의 수평확장을 통해 기능추가를 고려한다.




### 참고자료
- [야곰 - 프로토콜 지향 프로그래밍](https://www.youtube.com/watch?v=9gkzHUsQiUc)

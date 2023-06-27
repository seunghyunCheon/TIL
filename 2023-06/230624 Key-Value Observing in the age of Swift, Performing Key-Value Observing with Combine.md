# 230624 Key-Value Observing in the age of Swift, Performing Key-Value Observing with Combine
### 학습내용
- Key Value Observing(with AVPlayer)

## Key Value Observing(with AVPlayer)

KVO패턴은 Cocoa의 개념이 생겨난 이래로 존재해왔다. 이는 객체의 프로퍼티 변화에 대해 어떤 객체에게 알려주기 위해 사용이된다.

앱 내에서 분리된 부분사이에서 변화에 대해 소통하는 것과 관련하여 많은 것이 존재한다. 주로 많이 사용되는 것은 `Delegate`패턴이다.

### 델리게이트 패턴의 이점
- 자체 문서화되어있다. 코드에서 어떻게 사용해야 하고 무엇을 하는 지를 볼 수 있다.
- 1대1 관계라면 재사용가능하다.

### 델리게이트 패턴의 한계
만약 시스템에서 제공되는 클래스에 대해 관찰하고 싶다면 이를 하지 못하는 경우가 존재한다. 예를 들어 `AVPlayer`에서 상태가 예이다. 이를 위해 `AVPlayerItem`을 수정할 수도 없다. 이는 오직 `addObserver`를 이용해야만 한다. 
과거문법은 다음과 같다.

```swift
class ViewCOntroller: UIViewController {
    let player = AVPlayer()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // 1. AVPlayerItem의 상태변화에 대한 관찰자를 추가.
        player.currentItem?.addObserver(self,
                                       forKeypath: #keyPath(AVPlayerItem.status),
                                       options: [.old, .new],
                                       context: nil)
        
        // 2. 알림처리.
        override func observeValue(forKeyPath: keyPath: String?,
                                  of object: Any?,
                                  change: [NSKeyValueChangeKey: Any]?
                                  context: UnsafeMutableRawPointer?) {
            if keyPath == #keyPath(AVPlayerItem.status) {
                let status: AVPlayerItem.Status
                if let statusNumber = change?[.newKey] as? NSNumber {
                    status = AVPlayerItem.Status(rawValue: statusNumber.intValue)
                } else {
                    status = .unkonw
                }
            }
            
            // Switch over status value
            switch status {
            case .readyToPlay:
                // Player item is ready to play.
                break
            case .failed:
                // Player item failed. See error.
                break
            case .unknown:
                // Player item is not yet ready.
                break
            }
        }
    }
    
     deinit {
        player.currentItem?.removeObserver(self, forKeyPath: #keyPath(AVPlayerItem.status))
    }
}
```

위 코드에서는 세 가지 아쉬운 부분이 존재한다.
- 관찰은 분리된 단계로 설정된다. 설정할 관찰이 여러 개인 경우 각 관찰은 하나의 addObserver를 사용하여 설정한다.
- 알림이 하나의 공간에 다뤄진다. 여러개의 관찰이 있을 때를 상상해본다면 각각의 알림이 `if keyPath == #KeyPath(key)`를 거쳐야 한다는 것을 알 수 있을 것이다. 그리고 유용한 정보를 꺼내기 위한 문법은 가독성이 좋지않다.
- 설정한 관찰사항은 크래시를 피하기위해 명시적으로 제거해야 한다.

### 새로운 KVO 방식
```swift
class ViewController: UIViewController {
    let player = AVPlayer()
    
    var timeControlStatusObserver: NSKeyValueObservation?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // AVPlayerItem 상태변화에 대한 옵저버 추가.
        timeControlStatusObserver = player.observe(\.timeControlStatus, options: [.old, .new] { (player, change) in
            if let newValue = change.newValue,
            if let oldValue = change.oldValue,
            newValue != oldValue {
                pint("Time Control Status has changed")
            }
            
        })
    }
}
```

`\`문법은 keyPath를 추출하는 스위프트 문법이다. 
observer를 추가하면서 동시에 `observe()`를 하는 역할을 한다. `options`파라미터는 `changeHandler`가 호출될 때 oldValue인지 newValue인지 혹은 둘 다인지를 알려주는 파라미터이다. 

observation은 변수로 저장이 되어야 한다. 왜냐하면 observer는 만약 더이상 참조하고있는 게 존재하지 않는다면 제거되기 때문이다. 

그러므로 `observe`호출의 결과를 변수로 담음으로써 관찰을 유지할 수 있다. 그리고 이로 인해 observer의 해제를 뷰컨트롤러의 라이프사이클에 맡기는 것이다. 따라서 deinit도 필요가 없어진다.

## Performing Key-Value Observing with Combine

### Overview
여러 프레임워크들은 kvo를 사용해 앱의 비동기적인 변경사항을 알린다. 콜백으로 구성된 kvo코드를 컴바인으로 변경함으로써 코드를 더욱 예쁘고 유지가능하게 만드는 것이 목표이다.

### Montioring Changes with KVO
다음 예제는 `UserInfo`타입을 `KVO`로 설정한 프로퍼티를 이용한 예제이다. `viewDidLoad`에서 `observe`메서드를 사용하여 프로퍼티의 변경사항에 대해 다룰 수 있도록 구성했다. 

```swift
class UserInfo: NSObject {
    @objc dynamic var lastLogin: Date = Date(timeIntervalSince1970: 0)
}
@objc var userInfo = UserInfo()
var observation: NSKeyValueObservation?


override func viewDidLoad() {
    super.viewDidLoad()
    observation = observe(\.userInfo.lastLogin, options: [.new]) { object, change in
        print ("lastLogin now \(change.newValue!).")
    }
}


override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    userInfo.lastLogin = Date()
}
```
### Converting KVO Code to Use Combine
KVO를 컴바인으로 변경하기 위해 `observe`메서드를 `NSObject.KeyValueObservingPublisher`로 변경해야 한다. 부모객체에서 `publisher(for:)`를 사용하여 접근할 수 있다.
```swift
class UserInfo: NSObject {
    @objc dynamic var lastLogin: Date = Date(timeIntervalSince1970: 0)
}
@objc var userInfo = UserInfo()
var cancellable: Cancellable?


override func viewDidLoad() {
    super.viewDidLoad()
    cancellable = userInfo.publisher(for: \.lastLogin)
        .sink() { date in print ("lastLogin now \(date).") }
}


override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    userInfo.lastLogin = Date()
}
```

KVO publisher는 관찰되는 타입의 요소를 만든다. 
이를 통해 처음 과정처럼 newValue를 풀어내는 과정을 제거할 수 있게된다.

### 참고문서
- [Key-Value Observing in the age of Swift](https://medium.com/nerd-for-tech/key-value-observing-in-the-age-of-swift-f509c54fa5e8)
- [Apple Docs - Performing Key-Value Observing with Combine](https://developer.apple.com/documentation/combine/performing-key-value-observing-with-combine)

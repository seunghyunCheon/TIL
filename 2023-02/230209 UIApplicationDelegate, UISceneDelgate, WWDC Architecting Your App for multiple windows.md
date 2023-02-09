230209 UIApplicationDelegate, UISceneDelgate, WWDC Architecting Your App for multiple windows
===
TIL (Today I Learned)
===
학습내용
---
- UIApplicationDelegate, UISceneDelegate 학습하기
- Managing your app’s life cycle Article 정독
- WWDC Architecting Your App for multiple windows 시청
- 활동학습 정리
## ✏️ UIApplicationDelegate
> 앱에 공유되는 행동들을 관리하는 메서드 집합
```swift
@MainActor protocol UIApplicationDelegate
```
`app delegate`객체는 앱의 공유되는 행동을 관리한다. `app delegate`는 앱의 루트 객체이고 `UIApplication`과 결합하여 시스템의 상호작용을 관리한다. `UIApplication` 객체처럼 `UIKit`은 앱이 시작할 때 `app delegate`를 만들기 때문에 항상 존재한다.

### `app delegate` 사용하는 경우
- 앱의 중심 데이터구조를 초기화할 때
- 앱의 `scene`을 설정할 때
- 앱 밖에서 온 이벤트에 응답할 때. (저전력 경고, 다운로드 완료 알림 등)
- `scene` `view`, `viewController`가 아닌 앱 자체를 타겟한 이벤트에 응답할 때
- `Apple Push Notification Service`와 같은 서비스를 초기에 등록할 때

### iOS12 이전의 Life-cycle 관리

iOS12이전에는 앱의 생명주기 이벤트를 관리하는데 `app delegate`를 사용했다. 구체적으로 앱이 `foreground`에 들어오거나 `background`로 이동할 때 앱의 상태를 업데이트하는데 사용했다.

## ✏️ UISceneDelegate
> scene안에서 발생하는 생명주기 이벤트에 응답하는데 사용하는 핵심 메서드
```swift
@MainActor protocol UISceneDelegate
```
앱 `user interface`의 하나의 인스턴스 안의 생명주기 이벤트를 관리할 때 `UISceneDelegate`를 사용한다. 이 `interface`는 신에 영향을 주는 상태 전환(active상태가 되거나 background로 이동하는)에 응답하는 메서드들을 정의한다.

`UISceneDelegate`를 직접 생성하지 않는다. 대신에 `custom delegate class`의 이름을 명시해서 사용한다. 이 정보를 `Info.plist`파일에서 명시하거나 `UISceneConfiguration`객체에 지정할 수 있다.

## ✏️ Managing your app’s life cycle Article
> 앱이 foreground나 background에 있을 때 시스템 알림에 응답하고 기타 시스템과 관련된 중요한 이벤트들을 다룬다.

앱의 현재 상태는 어떤 시점에 무엇을 할 수 있고 무엇을 할 수 없는지 결정한다. 예를 들어 `foreground`앱은 사용자의 주목을 가지고 있기 때문에 CPU를 포함한 시스템 리소스보다 우선순위를 가진다. 
반면에 `background`앱은 화면 밖이기 때문에 가능하면 적게 일을 하거나 아무것도 안하는 것이 좋다. 앱의 상태가 변함에 따라 행동들을 조정해야 한다.

앱의 상태가 변할 때 `UIKit`은 적절한 `delegate object` 메서드를 호출함으로써 알린다.
- iOS13 이후로는 `scene-based app`의 생명주기 이벤트에 응답하는데 `UISceneDelegate`를 사용한다.
- iOS12 이전으로는 생명주기 이벤트에 응답하는데 `UIApplicationDelegate`를 사용한다.

### 👏 Note
만약 앱에서 `scene`을 지원한다면 iOS13이후로는 항상 `scene delegate`를 사용한다. iOS12이전에는 `app delegate`를 사용한다.


### Respond to scene-based life-cycle events
만약 앱이 `scene`을 지원한다면 `UIKit`은 각각에 대해 분리된 생명주기 이벤트들을 전달할 것이다. 하나의 `scene`은 앱 UI의 하나의 인스턴스를 나타낸다. 만약 사용자가 여러개의 `scene`을 만들었다면 그것들을 분리해서 보여주고 숨길 것이다. 
각각의 `scene`들은 자체의 생명주기를 가지고 있기 때문에 각각 다른 실행상태에 있을 수 있다. 예를 들어 어떤 `scene`은 `foreground`에 있는 동안 어떤 `scene`은 중단되거나 `background`에 있을 것이다.

### 👏Note
`Scene support`는 옵션기능이다. 기본 support를 가능하게 하려면, `UIApplicationSceneManifest`키를 `Info.plist`파일에 추가하면 된다.

다음 그림은 `scene`의 상태변화를 보여준다. 사용자나 시스템이 앱에 새로운 `scene`을 요청하면 `UIKit`은 그것을 만들고 연결되지 않은 상태로 둔다. 유저가 요청한 `scene`은 빠르게 `foreground`로 이동하여 화면에 보여지고, 시스템이 요청한 `scene`은 일반적으로 `background`로 가서 이벤트를 처리한다. 
예를 들어 시스템은 `background`에서 위치 이벤트를 처리할 수 있다. 사용자가 앱의 UI를 `dismiss`할 때 `UIKit`은 관련된 `scene`을 `background`상태로 보내고 끝내 `suspended` 상태로 보낸다. `UIKit`은 언제든지 이러한 상태에 있는 `scene`들의 연결을 해제하고 리소스를 회수하여 연결 해제된 상태(`unattached`)로 되돌릴 수 있다.
<img src="https://i.imgur.com/ktFAvrQ.png" width="700"/>

다음의 작업을 수행하기위해 `scene transition`을 사용한다.
- `UIKit`이 앱에 `scene`을 연결할 때, `scene`의 초기 UI를 설정하고, `scene`이 필요한 데이터들을 로드한다.
- `forground-active state`로 전환할 때, UI를 설정하고 유저와 상호작용할 준비를 한다.
- `forground-active state`에서 빠져나올때, 데이터를 저장하고 앱의 행동을 중지한다.
- `background state`에 진입했을 때, 중요한 작업을 완료하고, 가능한 한 많은 메모리를 확보하며 스냅샷을 준비한다.
- `scene`연결이 해제될 때 `scene`과 관련된 공유 리소스들을 정리한다.
- `scene`과 관련된 이벤트 뿐만 아니라 `UIApplicationDelegate` 객체를 이용하는 앱의 시작에도 반응해야 한다.

### Respond to app-based life-cycle events
iOS12 이전에는 앱이 `scenes`를 지원하지 않았다. `UIKit`은 모든 생명주기 이벤트를 `UIApplicationDelegate` 객체를 이용해 다뤘다. 이 `app delegate`는 분리된 화면들에 보여지는 것들을 포함하여 모든 `app windows`를 관리했다. 결과적으로 앱 상태 전환은 앱의 전반적인 UI에 영향을 끼쳤다.

다음 그림은 `app delegate`객체를 수반하는 상태전환을 보여준다. 시작이후 시스템은 UI가 스크린에 보여지는지 여부에 따라 앱을 `inactive`한 상태 혹은 `background`상태로 놓는다. `forground`로 시작하면 시스템이 자동으로 앱을 `active state`로 전환한다. 그 후 앱이 종료될 때까지 `active`와 `background` 사이에서 변동한다.


<img src="https://i.imgur.com/EXWtOnQ.png" width="700"/>

다음의 작업을 수행하기 위해 `app transition`을 사용한다.
- 시작할때 앱의 데이터구조와 UI를 초기화한다.
- `active`상태로 갈 때 UI설정을 끝내고 유저와 상호작용할 준비를 한다.
- `deactivate`상태로 갈 때 데이터를 저장하고 앱의 행동을 끝낸다.
- `background`상태로 전환되면, 중요한 작업을 끝내고 메모리를 확보하고 앱의 스냅샷을 준비한다.
- 종료할 때, 모든 작업을 즉시 멈추고 공유 자원을 `release`한다.

## WWDC Architecting Your App for multiple windows 시청

### iOS 12 이전 버전의 App Delegate
1. 프로세스 수준의 이벤트를 알려준다.
2. 애플리케이션의 UI 상태를 알린다.


iOS13이후로는 여전히 하나의 프로세스만 공유하지만 여러 사용자 인터페이스 혹은 `scene sessions`를 가지기 때문에 유효하지 않다.

<img src="https://i.imgur.com/iLShwQA.png" width="500"/>

### iOS 13 이후 버전의 변화

App Delegate에서 수행했떤 모든 UI설정 또는 해제 작업이 Scene Delegate의 책임이 되었다. 그리고 scene하나당 하나의 session을 갖는 sesssion들을 다루게 되었다.

<img src="https://i.imgur.com/mUaE4Z1.png" width="500"/><br/><br/>


실행중인 scene을 잠시 위로 스와이프해서 화면에 보이지 않게 만들면 `active` 상태를 `resign`하고 백그라운드 상태로 진입하게 된다. 
그리고 이후 어느 시점에서 메모리 장면을 해제하게 되면서 `scene delegate`가 보유한 모든 `scene`과 `view`계층이 해제된다. 
하지만 이 `scene`이 나중에 다시 연결되고 돌아올 수 있으므로 데이터나 상태를 영구적으로 삭제하지 않는 것이 중요하다. (복원이 필요)

다른 예시로 만약 명시적으로 앱을 위로 스와이프해서 삭제하려고 한다면 `didDiscardSceneSessions`를 호출하게 될 것이다. 이 메서드에는 텍스트 편집 앱에 저장되지 않은 초안과 같은 `scene`들을 영구적으로 저장할 수 있게 만들어진 곳이다.

### 장면 기반 상태복원

iOS13이후로는 상태복원이 더 이상 중요하지않고 장면 기반 상태복원을 구현하는 것이 중요하다.

만약 화면 네개를 켜놓고 2개는 백그라운드 상태로 보냈다고 했을때 백그라운드의 `scene`은 시스템이 메모리가 부족하다고 느낄때 언제든지 날아가게 할 수 있다. 
이 상태에서 날아간 `scene`들을 불러올 때 초기 화면부터 시작하면 기존에 해놨던 작업을 다시 해야하니까 좋지않은 `UX`를 보여주게 될 것이다.

<img src="https://i.imgur.com/BIy5QD7.png" width="500"/><br/><br/>

`scene`에 대해서 이를 복원하기 위해 나온 것이 다음의 메서드

<img src="https://i.imgur.com/JMB3Mkz.png" width="500"/><br/><br/>

기존에는 상태 복원을 하기위해 뷰 계층구조를 encoding했지만 iOS13이후로는 창을 다시 만들 수 있는 **상태**를 인코딩하는 방식으로 작동한다.
먼저 현재 window에서 가장 활동적인 사용자 활동양식을 찾는 메서드를 호출하고 반환한다.
그리고 실제로 `scene`이 `foreground`로 들어가게 되면 상태 복원 활동이 포함되어있는지를 확인하고 포함되어 있다면 활동을 포함하고 그렇지않으면 상태 없이 완전히 새로운 창을 만든다.


<img src="https://i.imgur.com/RqoKCFy.png" width="500"/><br/><br/>

### 참고자료
[Apple Developer Documentation - UIApplicationDelegate](https://developer.apple.com/documentation/uikit/uiapplicationdelegate)
[Apple Developer Documentation - UISceneDelegate](https://developer.apple.com/documentation/uikit/uiscenedelegate)
[Apple Developer Documentation - Managing your app’s life cycle](https://developer.apple.com/documentation/uikit/app_and_environment/managing_your_app_s_life_cycle)
[WWDC - WWDC Architecting Your App for multiple windows](https://developer.apple.com/videos/play/wwdc2019/258/)

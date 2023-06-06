# 230605 WWDC - AVKit, kodeco - AVFoundation(비디오 녹화하고 실행)
### 학습내용
- WWDC - AVKit
- codeco - AVFoundation(비디오 녹화하고 실행)

## WWDC - AVKit

```swift
import UIKit

// AVPlayer 생성
let player = AVPlayer(url: "https://my.example/video.m3u8")

// AVPlayerController 생성
let playerViewController = AVPlayerViewController()
playerViewController.player = player

// show it
present(playerViewController, animated: true)
```

`AVKit`은 두 가지 장점이 존재한다.
1. media playback object를 완전히 제어할 수 있다.
2. UIKit 기반 API를 사용하여 표시하는 API이다.

iOS13용 `AVPlayerViewController` 및 기능을 살펴보자.

### AVPlayerViewController Delegate를 확장하여 전체화면 및 축소화면에 대한 상태를 알려줌.

전체 화면 프레젠테이션이 시작되거나 끝날 때 알림을 받는다. 
```swift
// ios 12이상
func playerViewController(_ playerViewController: AVPlayerViewController, willBeginFullScreenPresentationWithAnimationCoordinator coordinator: UIViewControllerTransitionCoordinatoor)

func playerViewController(_ playerViewController: AVPlayerViewController, willEndFullScreenPresentationWithAnimationCoordinator coordinator: UIViewControllerTransitionCoordinatoor)
```

구현을 살펴보자.

```swift
func playerViewController(_ playerViewController: AVPlayerViewController, willBeginFullScreenPresentationWithAnimationCoordinator coordinator: UIViewControllerTransitionCoordinatoor) {
    coordinator.animate(alongsideTransition: { (context) in 
        // Add coordinated animations            
    }) { (context) in 
        if context.isCancelled {
            // still embedded inline
        } else {
            // Presented full screen
            // Take strong reference to playerViewController if needed
        }
    }
}
```
`context.isCancelled`코드에서 전환이 성공했는지 또는 사용자가 취소했는지 알 수 있으며 이는 인라인 프레젠테이션에서 전체 화면으로 전환하는 방법을 알기위한 source of truth이다.

### AVPlayerViewController Best Practices

### Modal방식을 사용한다.
자식 뷰컨트롤러에 추가하는 방식이 아닌 모달방식을 사용한다. 
기본적으로 모달 프레젠테이션 스타일은 전체이기 때문에 만질 필요가 없다.

### videoGravity를 설정하지 않는다.

### viewWillAppear나 그 외에것에 의존하지 않는다.



## codeco - AVFoundation(비디오 녹화하고 실행)

`AVFoundation`은 iOS, macOS, watchOS 및 tvOS에서 시간 기반 시청각 미디어 작업을 위한 완전한 기능을 갖춘 프레임워크이다. 이를 사용하여 쉽게 `movie`나 `MPEG-4` 파일 등을 앱 안에서 생성, 실행, 편집할 수 있다. 

`macOS`에서는 10.7이후로, `iOS`에서는 4버전 이후로 중요한 역할을 해왔다. 그 이후로 계속 발전하여 현재는 100개 이상의 클래스를 가지고 있다.

`AVFoundation`을 사용하여 다음의 기능을 사용할 수 있다.
- 미디어 라이브러리로부터 비디오를 선택하고 실행한다.
- 비디오를 녹화하고 미디어 라이브러리에 저장한다.
- 여러 클립을 사용자 지정 사운드트랙이 포함된 하나의 결합된 비디오로 병합한다.

### 비디오를 선택하고 실행하기

<img src="https://hackmd.io/_uploads/ryd5NzjIh.png" width="300"/><br/>
첫 번째 버튼은 비디오를 보여주는 뷰 컨트롤러인 `PlayVideoController`와 segue가 되어있는 상태이다. 이 버튼을 누르는 순간 비디오를 선택하고 실행하는 코드를 작성해야 한다.

먼저 `PlayVideoController`에 `AVKit`과 `MobileCoreServices`를 import한다.

`AVKit`을 import하는 것은 선택된 비디오를 실행시키는 `AVPlayer`에 대한 접근을 제공한다. 
`MobileCoreServices`는 후에 필요해지는 `kuTypeMovie`와 같은 미리 정의된 상수를 포함한다. 

다음으로 파일 끝에 클래스 확장을 추가한다. 

```swift
// MARK: - UIImagePickerControllerDelegate
extension PlayVideoViewController: UIImagePickerControllerDelegate {
}

// MARK: - UINavigationControllerDelegate
extension PlayVideoViewController: UINavigationControllerDelegate {
}
```

시스템이 제공하는 `UIIMagePickerController`를 사용하여 사용자가 photo library에서 비디오를 볼 수 있도록 만든다. 해당 클래스는 이러한 델리게이트 프로토콜을 통해 앱과 다시 통신한다. 비록 이름은 이미지이지만 비디오도 잘 동작한다.

다음 `PlayVideoViewController`로 돌아와 메인 클래스 정의부에 `playVideo(_:)`에 다음의 코드를 추가한다.

```swift
VideoHelper.startMediaBrowser(delegate: self, sourceType: .savedPhotosAlbum)
```

이 코드는 `VideoHelper`객체에 정의되어있는 `startMediaBroswer`라는 메서드를 호출하는 코드이다. 이 호출에서 다음의 일이 발생한다.
- image picker를 open하고 대리자를 self한다.
- `savedPhotosAlbums`라는 source type을 통해 카메라 롤에서 이미지를 선택할 수 있도록 한다.

자세히 살펴보고자 `VideoHelper.swift`를 열면 다음의 역할을 하는 것을 볼 수있다.
1. source가 디바이스에서 이용가능한지 확인한다. source는 카메라 롤, 카메라 그 자체, 사진 라이브러리를 포함한다. 이 확인은 `UIImagePickerController`를 사용할 때마다 필수적이다. 만약 이를 하지않으면 존재하지 않는 소스로부터 media를 선택해 크래시를 발생시킬 것이다.
2. 만약 소스가 이용가능하면 `UIImagePickerController`를 만들고 소스와 미디어 타입을 설정한다. 그리고 비디오를 선택할 때까지 코드는 타입을 `kUTTypeMovie`로 제한할 것이다. 
3. 마지막으로 `UIImagePickerController`가 정상적으로 보여진다.

비디오를 선택하고 `Choose`버튼을 누르더라도 그 앱은 단지 화면만을 반환한다. 이는 선택된 비디오를 처리하는 델리게이트 메서드를 만들지 않았기 때문이다. 

다시 `PlayVideoViewController`로 돌아와서 `UIImagePickerControllerDelegate`를 작성해보자. 

```swift
extension PlayVideoViewController: UIImagePickerControllerDelegate {
  func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
    guard let mediaType = info[UIImagePickerController.InfoKey.mediaType] as? String,
          mediaType == (kUTTypeMovie as String),
          let url = info[UIImagePickerController.InfoKey.mediaURL] as? URL else { return }
    
    dismiss(animated: true) {
      let player = AVPlayer(url: url)
      let vcPlayer = AVPlayerViewController()
      vcPlayer.player = player
      self.present(vcPlayer, animated: true)
    }
  }
}
```

1. 선택된 미디어타입과 URL를 얻는다.
2. image picker를 dismiss한다.
3. `AVPlayerViewController`를 만들고 미디어를 실행한다.

### 비디오 녹화하기

비디오 녹화를 할 컨트롤러인 `RecordVideoViewController`파일을 열고 위에서 했던 import와 델리게이트 메서드들을 구현해놓는다.

```swift
import MobileCoreServices
// MARK: - UIImagePickerControllerDelegate
extension RecordVideoViewController: UIImagePickerControllerDelegate {
}
// MARK: - UINavigationControllerDelegate
extension RecordVideoViewController: UINavigationControllerDelegate {
}
```

그리고 `PlayVideoViewController`와 동일한 메서드를 갖는데 `sourceType`을 `camera`로만 변경해준다.
```swift
class RecordVideoViewController: UIViewController {
  @IBAction func record(_ sender: AnyObject) {
    VideoHelper.startMediaBrowser(delegate: self, sourceType: .camera)
  }
}
```

녹화에 성공한 비디오를 저장하는 코드를 마저 작성해보자.

```swift
func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
    dismiss(animated: true, completion: nil)
      
      guard
        let mediaType = info[UIImagePickerController.InfoKey.mediaType] as? String,
        mediaType == (kUTTypeMovie as String),
        // 1
        let url = info[UIImagePickerController.InfoKey.mediaURL] as? URL,
        // 2
        UIVideoAtPathIsCompatibleWithSavedPhotosAlbum(url.path)
        else { return }
      
      // 3
      UISaveVideoAtPathToSavedPhotosAlbum(
        url.path,
        self,
        #selector(video(_:didFinishSavingWithError:contextInfo:)),
        nil)
  }
```

1. 전처럼 델리게이트 메서드는 비디오를 가리키는 URL을 제공.
2. 앱이 디바이스의 포토 앨범에 파일을 저장할 수 있는지 검증
3. 만약 가능하다면 저장.

`UISaveVideoAtPathToSavedPhotosAlbums`는 디바이스의 포토 앨범에 비디오를 저장하기 위해 `SDK`에 의해 제공되는 기능이다. 저장하려는 비디오의 경로와 콜백할 대상 및 작업을 전달하여 저장 작업의 상태를 알려준다.

마지막으로 콜백에 대한 구현을 추가해보자.

```swift
@objc func video(
  _ videoPath: String,
  didFinishSavingWithError error: Error?,
  contextInfo info: AnyObject
) {
  let title = (error == nil) ? "Success" : "Error"
  let message = (error == nil) ? "Video was saved" : "Video failed to save"

  let alert = UIAlertController(
    title: title,
    message: message,
    preferredStyle: .alert)
  alert.addAction(UIAlertAction(
    title: "OK",
    style: UIAlertAction.Style.cancel,
    handler: nil))
  present(alert, animated: true, completion: nil)
}
```

### 참고문서
- [Apple Docs - AVFoundation](https://developer.apple.com/av-foundation/)

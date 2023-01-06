230106 Class, Struct, 싱글톤, 딕셔너리, zip, AVFoundation, Timer, Cocoa, UIKit, Foundation
===
TIL (Today I Learned)
===
학습내용
---
- Class, Struct 학습하기
- 싱글톤 학습
- 딕셔너리, zip
- AVFoundation
- Timer
- Cocoa

## 고민한 점 / 해결 방법
## Class & Struct
> 클래스와 구조체는 프로그램 코드를 조직화 하기 위해 사용한다.
#### 클래스와 구조체 특징
- 값을 저장하기 위한 프로퍼티를 정의
- 기능을 제공할 메서드를 정의
- subscript 문법을 이용해 값에 접근하는 subscript 정의
- 초기 상태를 설정해주는 initializer정의
- 기본 구현에서 기능 확장
- 특정 종류의 표준 기능을 제공하기 위한 프로토콜 순응

#### 클래스만 가지고 있는 특징
- 상속: 클래스의 여러 속성을 다른 클래스에 상속함
- 타입 캐스팅: 런타임 시간동안에 인스턴스 타입을 확인
- Deinitializer: 인스턴스에 할당된 자원을 해제
- Reference counting: 클래스 인스턴스에 하나이상 참조 가능

#### Structure와 Enumeration은 값 타입이다.
- 값타입이 상수나 변수로 할당이 되었을 때, 혹은 함수로 전달될 때는 값이 복사가 된다.
- Swift의 기본 타입인 integer, floating-point number, Boolean, string, array, dictionaries는 전부 값타입이다.

> array, dictionaries, string과 같은 Collections들은 copying 퍼포먼스 비용을 줄이는 최적화를 사용한다. 복사본을 바로 만드는 것이 아닌 원본 인스턴스와 복사된 인스턴스 사이에 메모리를 공유하는 형태인 것이다. 만약 복사본이 수정된다면 그때 실제 주소가 copy가 된다. 이를 Copy-on-write 메커니즘이라고 부른다. 

#### Class는 참조 타입이다.
- 참조 타입은 값 타입처럼 복사해서 만들어내는 것이 아니라 생성된 인스턴스를 참조하여 동일한 인스턴스를 가리키는 타입이다.

#### 식별 연산자(Identity Operators)
- 클래스는 참조 타입이기 때문에 여러 상수와 변수에서 같은 인스턴스를 가리키고 있는지 확인할 수 있다.
> `===`: 두 상수나 변수가 같은 인스턴스를 참조하는 경우 참
> `==`: 두 상수나 변수가 같은 값을 참조하는 경우 참. class에서 사용하기 위해서는 equatable 프로토콜을 채택해야 한다.

#### 클래스와 구조체 선택
- 일반적으로 다음의 조건 중 1개이상을 만족하면 구조체 사용을 고려한다.
    - 주 목적이 관계된 값을 캡슐화하는 것일 때
    - 인스턴스, 내부 프로퍼티가 참조보다는 복사되기를 원할 때
    - 프로퍼티나 메소드 등을 상속할 필요가 없을 때
    - 작은 값을 처리할 때.(스택은 메모리의 값이 크지않기 때문)
    - Objective-C와 상호 운용성이 없을 때
    - 고유 identity가 존재하지 않을 때

#### Swift의 String, Array, Dictionary의 할당과 복사 동작
- String, Array, Dictionary는 구조체로 NSString, NSArray, NSDictionary는 클래스로 되어있다. 하지만 구조체로 이루어져 있다고 바로 값이 복사되는 것이 아닌 실제로 데이터가 필요할 때 복사되는 형식이다.(copy-on-write)

## 싱글톤
> 응용 프로그램이 요청하는 횟수에 관계없이 하나의 인스턴스만 반환한다.
- 일반적인 서비스나 리소스를 제공하는 단일 제어 포인트가 바람직할때 사용한다.
- Cocoa framework class의 예시로는 NSFileManager, UIApplcation등이 있다.
- 싱글톤 객체를 반환하는 fatory method이름은 앞에 shared를 붙인 `sharedClassType`이다. 

#### 사용방법
```swift
class Singleton {
    static let shared = Singleton()
}
```
- 여러 스레드들이 동시에 접근해도 느리게 한번만 초기화되는 것을 보장하는 static type으로 선언한다.
```swift
class Singleton {
    static let shared: Singleton = {
        let instance = Singleton()
        // 추가적인 setup 코드
        return instace        
    }()
}
```
- 추가적인 setup 코드를 작성하려면 클로저 형태로 작성할 수 있다.

## 딕셔너리
- 키와 값이 쌍으로 이루어진 자료형이며 순서가 없다. (key는 중복 가능, value는 불가)
#### 기본적인 사용방법
```swift
let camper: [String: Int] = ["Rowan": 1, "goatt": 2, "송준": 3]
camper["Brody"] = 3
```
- 배열에 value를 추가해서 Dictionary로 변경. 인자로 Sequence프로토콜을 따르는 타입을 넣어주면 된다. 아래 zip의 경우 두 개의 Sequence를 받아 하나의 튜플 쌍의 Sequence로 만든다.
```swift
let camper = ["Rowan", "goatt", "송준", "Brody"]
print(Dictionary(uniqueKeysWithValues: zip(camper, 1...4)))
// ["Brody": 4, "송준": 3, "goatt": 2, "Rowan": 1]
```
#### zip 예제
```swift
let words = ["one", "two", "three", "four"]
let numbers = 1...4

for (word, numbers) in zip(words, numbers) {
    print("\(word): " \(number))
}
// Prints "one: 1"
// Prints "two: 2
// Prints "three: 3"
// Prints "four: 4"
```
- 만약 zip의 인자로 들어온 Sequence의 length가 다르다면 더 짧은 것을 선택해서 튜플 쌍을 만든다.


## [boostCourse - ios]

### AVFoundation
> 다양한 Apple 플랫폼에서 사운드 및 영상 미디어의 처리, 제어, 가져오기 및 내보내기 등 기능을 하는 프레임워크

### AVAudioPalyer
> 파일이나 buffer로부터 온 오디오 데이터를 실행시키는 객체
```swift
class AVAudioPlayer: NSObject
```
- 파일 재생 시간 길이의 제한 없이 사운드 실행
- 오디오의 볼륨, 이동, 루프 동작을 제어
- 앞으로 감기, 뒤로 감기 지원.
- 동시에 여러 사운드 실행
- 반복재생 기능
- 파일 또는 메모리에 있는 사운드 실행(네트워크에 있는 것은 불가)

#### AVAudioPlayer 생성자
```swift
public init(contentsOf url: URL) throws
public init(data: Data) throws
```
- URL이나 Data타입으로 생성 가능

#### AVAudioPlayer 사용
```swift
func intializePlayer() {
    guard let soundAsset: NSDataAsset = NSDataAsset(name: "sound") else {
    print("음원 파일을 가져올 수 없습니다.")
    return
    }    
}
do {
    // rawData를 넣어 AVAudioPlayer 인스턴스 생성
    try self.player = AVAudioPlayer(data: soundAsset.data)
} catch let error as NSError {
    print("플레이어 초기화 실패")
    print("코드: \(error.code), 메시지: \(error.localizedDescription)")
}
// duration은 audio의 총 시간
self.progressSlider.maximumValue = Float(self.player.duration) 
self.progressSlider.minimumValue = 0
// audio 타임라인의 현재 시간
self.progressSlider.value = Float(self.player.currentTime)

self.player.play() // 오디오 재생
// self.player.pause() 오디오 멈춤
```
- 음악재생에 따른 슬라이더 value의 변화와 Timer의 역할은 주안점이 아니기 때문에 디테일하게 다루진 않았다.

#### NSDataSet
> asset 카탈로그에 저장된 data set타입의 객체
- 만약 앱에서 Json, XML, Audio와 같은 데이터 파일을 외부 네트워크로부터 받을 때 기능이 작동하게끔 설계되어 있다면 불리한 네트워크를 사용하는 사용자들에게는 좋지않은 UX를 줄 수 있다. 이를 대안으로 asset 카탈로그에 넣어 앱 번들에 포함시킨다면 네트워크와 관계없이 기능을 이용할 수 있게 된다.
- 앞서 말한 Json, XML, Audio는 Data Set이라는 타입으로 만들 수 있다.

#### 카탈로그 데이터 불러오기
```swift
guard let asset = NSDataSet(name: "NamedColors") else {
    falalError("Missing data asset")
}
let data = asset.data // data프로퍼티는 rawData가 담긴다.
print(data)
// 19382 bytes
```

## Timer
> Timer는 특정 시간이후에 target 객체에 메세지를 보내는 객체
```swift
class Timer: NSObject
```
- 타이머는 런루프와 같이 실행된다. 런 루프는 타이머에 대해 강한 참조를 유지하므로 런루프에 타이머를 추가한 이후에 강한참조를 유지할 필요가 없다.(무슨 말이지...?😂) 타이머를 효과적으로 사용하기 위해 런 루프 작동원리를 알아야 한다.(다음 주제는 런 루프다..)
- 타이머는 실시간 메커니즘이 아니다. 만약 타이머의 호출 시간이 긴 런루프 시간에 발생하거나, 런루프가 타이머를 check하고 있지 않을때 발생한다면 timer가 호출되지 않을 것이다. 그러므로 실제 시간은 훨씬 나중일 수 있다. (Timer Tolerance를 확인하라고 나와있는데 아마 허용오차범위에 대한 얘기인 것 같다)

- Timer 사용방법
```swift
class HurryUpGame {
    var timer: Timer?
    var leftCount = 60
    func timerSetting() {
        self.timer = Timer.scheduledTimer(timeInterval: 1,
                                          target: self,
                                          selector: #selector(timerExection),
                                          userInfo: nil,
                                          repeats: true)
    }
    
    @objc private func timerExection() {
        if self.leftCount == 0 {
            self.timer?.invalidate()
        }else {
            leftCount -= 1
            print("\(leftCount)초 남았습니다.")
        }
    }
}

let game = HurryUpGame()
game.timerSetting()
```
- playGround환경에서 타이머를 돌리려면 특정 옵션을 설정해야 한다. 


## Cocoa, Cocoa Touch
> 코코아와 코코아 터치는 OS X, iOS 애플리케이션을 개발하는 환경이다. 둘 다 objective-C 런타임을 이용하고 두개의 코어 frameWork가 존재한다.
- 코코아는 Foundation과 Appkit framework를 포함한다.(for OS X)
- 코코아 터치는 Foundation과 UIKit framework를 포함한다.(for iOS)
- 코코아라는 용어는 현재 Objective-C 런타임을 이용하고 NSObject를 상속받은 것을 가리킨다.

#### Founcation Framework
> iOS 애플리케이션의 운영체제 서비스와 기본 기능을 포함하는 프레임워크

- Foundation에서 데이터 타입 및 컬렉션 타입의 대부분은 Objective-C 언어 기능에서 지원되지 않아서 생긴 것. Swift를 사용하는 경우 Swift 표준라이브러리에서 사용이 가능하다.
- 데이터 타입 및 컬렉션 타입 사용
- 날짜와 시간을 계산하거나 비교
- 물리적 차원을 숫자로 표현 및 관련 단위 간 변환 가능
- 숫자, 날짜, 측정값을 문자열로 변환 혹은 반대 작업
- 컬렉션의 요소를 검사하거나 정렬
- 에셋과 번들 데이터 접근 지원
- Notification 기능 지원
- File System 기능 지원
- JSON, 바이너리 파일들을 객체로 변환 또는 반대 작업
- iCloud기능을 이용해 데이터를 동기화하는 작업
- 표준 인터넷 프로토콜을 이용해 URL과 상호작용하고 서버와 통신하는 기능
- 로컬 네트워크를 위한 작업



#### UIKit
> iOS 애플리케이션 개발시 사용자에게 보여질 화면을 구성하고 사용자 액션에 관련된 다양한 요소들을 포함한다.

- 제스처 처리, 애니메이션, 그림 그리기, 이미지 처리, 텍스트 처리를 한다.
- UIResponder에서 파생된 클래스나 사용자 인터페이스에 관련된 클래스는 메인 스레드, 메인 디스패치 큐에서만 사용한다.




### Reference
- [Class와 Struct 차이점](https://icksw.tistory.com/256)
- [NSDataAsset](https://nshipster.co.kr/nsdataasset/)
- [Apple Developer Documentation - NSDataAsset](https://developer.apple.com/documentation/uikit/nsdataasset)
- [Apple Developer Documentation - Timer](https://developer.apple.com/documentation/foundation/timer)
- [Apple Developer Documentation - UIKit](https://developer.apple.com/documentation/uikit)
- [Apple Archive - Cocoa](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/Cocoa.html)
- [boostcourse - cocoa](https://www.boostcourse.org/mo326/lecture/17994?isDesc=false)
- [boostcourse - Foundation](https://www.boostcourse.org/mo326/lecture/17996?isDesc=false)
- [boostcourse - UIKit](https://www.boostcourse.org/mo326/lecture/17995?isDesc=false)



Class, Struct, 싱글톤, 딕셔너리, zip, AVFoundation, Timer, Cocoa, UIKit, Foundation

###### tags: `Class`, `Struct`, `싱글톤`, `딕셔너리`, `zip`, `AVFoundation`, `Timer`, `Cocoa`, `UIKit`, `Foundation`

230405 Keychain Services, Storing Keys in the Keychain , UserDefaults
===
학습내용
- Keychain Services
- Storing Keys in the Keychain 아티클
- UserDefaults

## Keychain Services
> 사용자를 대신해서 소량의 데이터를 안전하게 저장한다

### Overview
컴퓨터 사용자는 안전하게 저장할 필요가 있는 데이터들을 자주 가진다. 예를들어 대부분의 사용자는 수많은 온라인 계정을 가진다. 고유하고 복잡한 패스워드를 기억하는 것은 불가능하다. 하지만 그것들을 쓰는 것은 안전하지 않다. 사용자들을 일반적으로 단순한 패스워드를 재사용하는데 이 방법은 역시 안전하지 않다.

키 체인 서비스 API는 keychain이라고 불리는 암호화된 데이터베이스 안에 사용자 데이터를 저장하는 메커니즘을 앱에게 제공함으로써 이 문제를 해결한다. 안전하게 비밀번호를 기억하게되면 사용자는 복잡한 비밀번호를 설정할 수 있게 된다.

키체인은 비밀번호에만 적용되는 게 아니다. 사용자가 명시적으로 신경써야 할 신용카드나 Note와 같은 것들을 저장할 수 있다. 그리고 사용자가 필요하지만 인지되지는 않는 것들도 저장할 수 있다. 
예를들어 신뢰 서비스로 관리하는 암호화키와 인증서를 통해 사용자는 보안 통신에 참여할 수 있다. 

<img src="https://i.imgur.com/oPN6PFX.png" width="500"/>

## Storing Keys in the Keychain 아티클
> 키 체인안에 암호화된 키들을 저장하고 접근한다

### Overview
키체인은 패스워드나 암호화된 키를 저장하는데 최적의 장소이다. 키체인 서비스 API를 이용해 추가하고 회수하고 삭제하고 수정할 수 있다.

### Create a Query Dictionary
키들을 생성할 때 프로세스의 암시적인 부분으로써 키체인에 그것들을 저장할 수 있다. 만약 다른 방법으로 키를 얻은경우에도 키체인에 그것을 저장할 수 있다. 이를 하기위해 item을 포함하고 설명하는 query dictionary를 만듦으로써 시작한다.

```swift
let key = <# a key #>
let tag = "com.example.keys.myKey".data(using: .utf8)!
let addquery: [String: Any] = [kSecClass as String: kSecClassKey,
                               kSecAttrApplicationTag as String: tag,
                               kSeecValueRef as String: key]
```
이 query dictionary는 `kSecClass` entry를 위한 `kSecClassKey`값을 사용한다. 이는 증명서, 패스워드와는 반대로 key item을 나타낸다. 또한 후에 검색할 때 다른 것으로부터 키를 구별할 수 있게 만드는 tag를 적용할 수도 있다.

### Store the Item
`SecItemAdd(_:_:)`함수를 사용해서 item을 저장한다.
```swift
let status = SecItemAdd(addqury as CFDictionary, nil)
guard status == errSecSuccess else { throw <# an error #> }
```

### Retrieve the Item
key를 회수하려고 할 때 동일한 application tag를 사용해 또다른 query dictionary를 구성해야 한다.
```swift
let getquery: [String: Any] = [kSecClass as String: kSecClassKey,
                               kSecAttrApplicationTag as String: tag,
                               kSecAttrKeyType as String: kSecAttrKeyTypeRSA,
                               kSecReturnRef as String: true]
```

위 dictionary는 key가 `kSecAttrKeyType`이여야 한다는 것을 나타낸다. 그리고 이는 위에 예시에서 사용된 tag를 가져야 한다. 마지막 line은 회수가 key reference를 반환해야 한다고 말한다. (실제 key data가 아니다)

이 쿼리를 `SecItemCopyMatching(_:_:)`함수와 이용해 검색하고 빈 reference를 구성할 수 있다.
```swift
var item: CFTypeRef?
let status = SecItemCopyMatching(query as CFDictionary, &item)
guard status == errSecSuccess else { throw <# an error #> }
let key = item as! SecKey
```

만약 호출이 성공적이면 반환된 key reference를 사용해서 암호화된 작업을 수행할 수 있다. Objective-C에서는 이러한 방식으로 회수한 키를 사용한 후에 메모리를 비워줄 필요가 있다. 스위프트에서는 시스템이 이를 관리한다.

### Refine the Search

위 예시의 query는 특정 key class(public, private, symmetric)에 검색제한을 주지 않는다. 이는 오직 tag와 type에만 매칭이 되고 첫번재로 성공적으로 매치된 reference를 반환한다. 고유한 태그를 사용하지 않으면 key를 얻지 못할 것이다. 이는 매치가 반환되는 첫번째 키이기 때문에 원하는 값일 수도있고 아닐 수도 있다.

이를 다루기위해 다음의 방법이 존재한다
- 검색범위를 넓힌다: 만약 단일 키 대신`kSecMatchLimit` entry를 `kSecMatchLimitAll`과 함께 query dictionary에 추가한다면 key reference배열을 생상하기 때문에 원하는 key를 찾을 수 있다.
- 검색범위를 좁힌다: 만약 키를 구별짓는 다른 attribute가 존재하면(key size, key class와 같은) 그에 상응하는 entry를 추가한다.
- 처음부터 태그를 재사용하지 않는다. 주어진 태그와함게 키와 타입을 추가하기 전에 키체인에서 동일한 특징을 가진 존재하는 키를 읽는다. 만약 중복적인 걸 찾았다면 기존의 키를 재사용하고 새로운 것을 만드는 것을 skip하거나 다른 tag의 새로운 키를 만든다. 또는 `SecItemDelete(_:)`를 사용하여 이전의 것을 지운다.

## UserDefaults
user defaults database 인터페이스로 이곳에 key-value 쌍을 영구적으로 저장할 수 있다. 

```swift
class UserDefaults: NSObject
```

### Overview
`UserDefaults` 클래스는 defaults system과 함께 상호작용하는 코드 인터페이스를 제공한다. default system을 사용하면 앱이 사용자의 선호도에 맞게 행동을 customize할 수 있게 만들 수 있다.
예를 들어 사용자가 선호하는 측정 단위 또는 미디어 스피드를 지정하도록 허용할 수 있다. 앱들은 user's defaults data안에 파라미터 set에 값을 할당함으로써 이러한 선호도들을 저장할 수 있다. 그 파라미터들은 시작 시 앱의 기본상태나 기본적으로 작동하는 방식을 결정하는데 일반적으로 사용되기 때문에 `defaults`로 추론된다. 

런타임에 `UserDefaults` 객체를 사용해 앱이 user's defaults database로부터 불러와 사용하는 defaults를 읽을 수 있다. `UserDefaults`는 default value에 접근할 때마다 user's default database를 open하는 것을 피하기 위해 cache를 사용한다. default value를 설정할 때 프로세스 내에서 동기적으로 변경되고 영구적인 젖아소나 다른 프로세스에는 비동기적으로 변경된다. 

교육기관에서 관리되는 디바이스들을 제외하고 user's defaults는 단일 디바이스에 지역적으로 저장되며 백업 및 복원을 위해 유지된다. preference와 사용자의 디바이스에 있는 다른 데이터들을 동기화하기 위해 `NSUbiqutiousKeyValueStore`를 사용한다.

### 중요
preference subsystem에 직접적으로 접근하지 않는다. preference property list file을 수정하는 것은 변경손실, 반영 딜레이, 앱 크래시를 초래한다. preference를 수정하기 위해서 macOS 내부의 `defaults` command line 유틸리티를 사용한다. 

### Storing Default Objects
`UserDefaults` 클래스는 float, double, integer, bool, URL과 같은 common type에 접근할 수 있는 편의 메서드를 제공한다. 

default object는 property list가 되어야 한다. 이것들은 `NSData`, `NSString`, `NSNumber`, `NSDate`, `NSArray`의 인스턴스이다. 만약 다른 타입의 객체를 저장하길 원한다면 `NSData`의 인스턴스를 생성해서 저장해야 한다.

비록 mutable 객체를 값으로써 설정했을지라도 `UserDefaults`로부터 반환된 값들은 immutable하다. 예를들어 만약 "MyStringDefault"라는 value로써 mutable String을 설정했다면 그 문자열은 후에 `string(forKey:)`메서드로 회사할 때 immutable할 것이다. 만약 변경가능한 기본 값으로 설정하고 후에 그 String을 변경하려고 한다면 `set(_:forKey:)`를 반영하지 않는 한 그 기본값은 변경된 string값을 반영하지 않을 것이다.

더 자세한 것은 [Preferences and Settings Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/UserDefaults/Introduction/Introduction.html#//apple_ref/doc/uid/10000059i)를 살펴본다.

### Persisting File References

파일URL은 파일시스템에서의 위치를 명시한다. 만약 `set(_:forKey:)`메서드를 사용해서 특정 파일의 위치를 저장하고 사용자가 그 파일을 이동한다면 앱은 다음 launch에 그 파일을 찾지 못할 수도 있다. 
파일 시스템에의한 파일 참조를 저장하기 위해 `bookmarkData(options:includingResourceValuesForKeys:RelativeTo:)`메서드를 사용해 `NSURL` bookmark data를 만들어야 하고 `set(_:forKey:)`메서드를 이용해 지속해야 한다. 그리고 ` URLByResolvingBookmarkData:options:relativeToURL:bookmarkDataIsStale:error:`를 사용해서 user defaults에 저장된 파일 URL에 대한 boork mark data를 확인할 수 있다.

### Responding to Defaults Changes
특정 기본 값이 업데이트되었을 때 알림을 받을 수 있는 key-value observing을 사용할 수 있다. 또한 local defaults database의 모든 업데이트에 대해 알림을 받을 수 있게 `default` notification center에서 `didChangeNotification` 옵저버로 등록할 수 있다.

### Using Defaults in Managed Environments
만약 앱이 Managed environments를 지원한다면 `UserDefaults`를 사용해서 사용자의 이점을 위해 어떤 preference가 관리자에 의해 관리되면 좋을지를 결정한다. 
computer lab과 classroom같은 managed environment안에서 관리자나 선생님은 default preference의 세팅을 설정하여 시스템을 수정할 수 있다. 만약 preference가 이러한 방식으로 관리된다면 앱은 사용자가 preference를 수정하는것을 막을 수 있다.(숨기거나 제어못하도록 만들어서)

교육기관에의해 관리되는 앱은 iCloud key-value 저장소를 사용해 사용자의 다른 디바이스와 함께 작은양의 데이터를 공유할 수 있다. 예를 들어 textbook app은 현재 페이지숫자를 저장할지도모른다. 그럼 이 앱이 다른 인스턴스에서 시작이 되었을 때 같은 페이지를 열 수 있을 것이다.

### Sandbox Considerations
sandbox된 앱은 다른 앱의 preference에 접근, 수정을 하지 못한다. 다음의 예외를 제외하곤 말이다
- App Extension on macOS and iOS
- macOS위에 애플리케이션 그룹안의 앱들

`addSuite(named:)`메서드를 사용해 third-party 앱의 도메인을 추가하는 것은 그 앱의 preference에 접근하는 것을 혀용하지 않는다. 접근과 수정을 시도하려면 에러를 보게 될 것이다. 대신에 macOS는 다른 애플리케이션의 실제 preference file에 접근하기 보다는 앱의 container안에 위치한 파일들을 저장하고 쓴다.

### Thread Safety
`UserDefaults`클래스는 thread-safe하다



### 참고문서
- [Apple Docs - clipsToBounds](https://developer.apple.com/documentation/uikit/uiview/1622415-clipstobounds)
- [Apple Docs - adjustsFontSizeToFitWidth](https://developer.apple.com/documentation/uikit/uilabel/1620546-adjustsfontsizetofitwidth)


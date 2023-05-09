# 230509 Core Location, Getting the current location of a device, Configuring your app to use location services, Requesting authorization to use location services

### 학습내용
- Core Location
- Getting the current location of a device
- Requesting authorization to use location services
## Core Location
> 디바이스의 지리학적 위치와 방향을 얻는다.

### Overview
Core Location은 기기의 지리학적 위치, 고도, 위치, iBeacon 디바이스 근처의 상대적인 위치를 제공한다. 이 프레임워크는 Wi-Fi, GPS, Bluetooth, magnetometer, barometer, cellular hardware를 포함한 디바이스에 존재하는 컴포넌트들을 사용하여 데이터를 얻는다. 

`CLLocationManager`클래스의 인스턴스를 이용해서 `Core Location` 서비스를 수정, 시작, 멈출 수 있다. 이 객체는 다음과 같은 위치와 연관있는 행동들을 지원한다.
- 표준 및 중요한 위치 업데이트. 수정가능한 정확성을 기반으로 사용자의 현재 위치의 크고작은 변화를 추적한다.
- 지역 모니터링. 관심있는 지역을 모니터링하고 사용자가 그 지역에 갈 때와 나올 때 위치이벤트를 생성할 수 있다.
- Beacon ranging. beacon근처를 감지하고 위치한다.
- 나침반. 방향 변경을 보고한다.

위치 서비스를 사용하기 위해 앱은 인증을 요청하고 시스템은 그 요청을 승인하거나 거절할 메세지를 표시한다. 

iOS 디바이스에서 사용자들은 Setting app에서 어느때나 위치서비스 세팅을 변경할 수 있다. 이는 각각의 앱에 주거나, 디바이스 전체로 줄 수 있다. 앱은 인증변화를 포함하여 위치 매니저의 델리게이트 객체가 이벤트를 수신한다. 이 객체는 `CLLocationManagerDelegate`를 채택한다.

## Getting the current location of a device
> 위치 서비스를 시작하고 이 서비스를 사용하는데 있어 전력사용을 최적화 할 필요가 있는 정보를 제공한다.

### Overview
Core Location은 위치와 관련된 데이터를 얻는 많은 서비스를 제공한다. 하지만 대부분의 서비스들은 기기의 현재 위치를 제공한다. 이 정보를 다음에 사용할 수 있다.
- 네비게이션 역할에 사용. 걷거나, 차로 가는 등
- 근처 흥밋거리 찾기
- 사람과의 근접성을 기준으로 결과 제공
- 지도에서 현재 위치를 보여줌
- 위치를 친구와 공유
- 사진에 위치를 태그
- 소셜 미디어에서 확인
- 운동하는 동안 경로를 추적

Core Location은 Wi-Fi, cellular, GPS radios 등 다양한 하드웨어를 이용하여 현재 위치를 결정할 수 있다. 위치를 결정하기 위해 이러한 것들 모두를 사용할 필요는 없다.
대신 전력을 효율적으로 사용하는 방법안에서 선택적으로 골라 위치데이터를 수집한다. 
`CLLocationManager`객체의 설정은 어떤 하드웨어를 시스템이 사용할지와 이에 따라 앱이 전력소비에 쓰는것에 영향을 준다.

### 필요한 위치 데이터를 전달하는 서비스를 시작한다.
항상 앱의 요구를 충족시키는 가장 전력이 효율적인 위치서비스를 선택하는 것이 좋다.
Core Location은 위치 데이터를 얻는데 있어 다음의 서비스를 제공한다.
- `Visit` 위치 서비스는 위치데이터를 얻는 가장 전력효율적인 방법이다. 이 시스템은 누군가가 방문하는 장소와 얼마나 있는지를 모니터링하고 데이터를 후에 전달한다. `startMonitoringVisits()`메서드를 호출하여 서비스를 시작한다.
- `Significant-change`위치 서비스는 위치를 업데이트하는데 있어 저전력 방법을 제공한다. 이 서비스는 cellular와 Wi-Fi를 사용하여 오직 상당히 먼 거리를 초과하는 변경사항에 대해서만 보고한다. `startMonitoringSignificantLocationChanges()`메서드를 호출해서 서비스를 시작한다.
- `Standard` 위치 서비스는 가장 정밀하고 정기적인 위치 데이터를 제공한다. 하지만 다른 서비스들보다 많은 전력을 사용한다. 만약 앱이 네비게이션 역할을하거나 큰 정밀도를 필요로할 때 사용한다. `startUpdatingLocation()`을 호출하여 서비스를 시작하고 `requestLocation()`을 호출하여 단일 위치 이벤트를 얻는다.

위치 서비스를 바로 시작하는 앱은 거의 없으며 긴 기간동안 서비스를 구동하는 앱은 더 없다.
사용자가 앱에서 위치 서비스가 꼭 필요해서 이용하기 전까지는 이를 지연시켜야 한다. 그런다음 필요한 데이터를 얻자마자 서비스를 중단시켜 베터리 수명을 유지한다. 예를들어 만약 검색 결과를 필터링하기 위해 현재 위치만 얻고자 한다면 이후 바로 서비스를 중단시킨다.

위치 서비스를 위한 지원을 추가하기 위해 그 서비스와 관련된 메서드를 델리게이트 객체 안에서 전부 구현해야 한다. `Standard`와 `Significant-change` 위치 서비스는 같은 델리게이트 메서드를 사용하지만 `Visit` 서비스는 visit에 특화된 데이터를 받기에 분리된 메서드를 가진다. 

각각의 서비스 특징들에 대한 정보와 어떻게 시작하고 멈추는지를 확인하려면 [CLLocationManager](https://developer.apple.com/documentation/corelocation/cllocationmanager)을 확인한다. 

### 전력절약 기능

Core Location은 가능한한 최대한 전력 사용을 최적화하지만 여전히 이를 개선할 수 있다. 
가장 베스트 최적화는 앱이 이를 위치 서비스를 사용하지 않을 때 이를 끄는 것이다. 다른 최적화는 location manager 객체의 설정값을 조정하는 것이다.
- `distanceFilter`를 설정하여 필요한 정보에 대한 가장 큰 값을 제공한다. 값이 높을수록 시스템이 빈번하게 하드웨어를 끄도록 한다. -> 이는 업데이트가 일어나기전의 장치가 이동해야 하는 최소거리로 이 값이 클수록 서비스를 끄고있는 기간이 길어지니까 높게 설정하라는 뜻 같다.
- `desiredAccuracy`를 설정하여 필요한 정보에 대한 가장 작은 값을 제공한다. 더 낮은 정확성 값은 시스템이 저전력 하드웨어를 사용하도록 한다. 더 낮은 값은 또한 시스템이 더 빨리 하드웨어를 끄도록 한다.
- `activityType`을 적절한 값으로 설정하고 `pausesLocationUpdatesAutomatically`프로퍼티를 `true`로 한다. Core Location은 activity type을 사용하여 이 유형에 해당될 때 하드웨어를 끈다. 예를 들어 만약 activity type이 `CLActivityType.automotiveNavigation`이고 위치를 변경하지 않는다면 그 시스템은 이동을 감지할 때까지 하드웨어를 끌 것이다. `pausesLocationUpdatesAutomatically`프로퍼티는 true라면 자동으로 업데이트를 멈출 수 있도록 한다.
- 백그라운드 위치 업데이트가 필요한게 아니라면 `allowsBackgroundLocationUpdates`를 `false`로 설정한다.

전력효율을 개선하는 또다른 방법은 `NSLocationDefaultAccuracyReduced` key를 Info.plist파일에 추가하는 것이다.
만약 정확성이 낮은 위치 데이터로 충분하다면 이 키를 포함시킨다. 
예를 들어 운전 거리내에 있는 음식점 리스트들은 정밀한 위치가 필요하지 않을 수 있다. 필요에 따라 location manager를 사용하여 후에 정확한 데이터를 요청할 수 있다. 하지만 시스템은 디바이스 소유자에게 각 요청에 대한 승인을 띄워줘야 한다.

## Configuring your app to use location services
> 위치 데이터를 수집하기전에 위치 서비스가 이용가능한지와 앱에서 수정가능한지를 확인한다.

### Overview
대부분의 애플 디바이스에서 이용가능한 위치 데이터는 추가적인 컨텍스트와 정보를 제공해 앱 내용에 통합되도록 할 수 있게 만든다. 이 데이터를 사용하여 누군가에게 물리적인 위치를 지도위에 보여줄 수 있고 그들의 환경에 따라 네비게이션을 도와줄 수도 있다. 

`Core Location`을 앱에 추가했을 때 위치 서비스 데이터를 사용할 수 없는 상황을 대비한다. `Airplane` 모드에서의 디바이스는 필요한 하드웨어가 꺼지기 때문에 위치 업데이트를 전달할 수 없다. 그 시스템은 또한 앱이 위치서비스 데이터를 사용한다는 허가를 받아야해서 앱이 허가없이 서비스를 사용하는 것을 막아야 한다. 문제 없는 경험을 보장하려면 위치 데이터를 검색하기 전에 항상 다음을 수행해야 한다.
- 사용하려고 하는 위치 서비스가 이용가능한지를 검증한다.
- 앱이 위치 데이터에 접근하는 허가가 있는지를 검증한다.

만약 위치 데이터가 어떠한 이유로 이용이 불가능하다면 그것 없이도 할 수 있는 최고의 경험을 주는 앱을 만든다. 위치 데이터에 의존하는 것을 비활성화하고 필요한 동작을 얻기위한 대안을 제공한다.

### Important
위치 데이터는 민감한 정보이다. 그래서 안전하게 위치 데이터를 수집하고 있는 게 중요하다. 디스크에 저장하거나 네트워크로 보내는 위치 정보를 암호화한다. 추가적으로 어떻게 위치 데이터를 사용할 지에 대한 사용자에게 명확한 privacy policy를 제공한다.

### 앱에서 사용하는 서비스의 이용가능성을 확인
위치 서비스를 사용하기 전에 항상 이용가능한지를 확인해야 한다. 서비스는 다음과 같이 많은 이유로 이용가능하지 않을 것이다. 
- Airplane 모드일 때
- 디바이스가 필요한 하드웨어를 갖고 있지 않을 때
- 디바이스가 서비스를 지원하지 않을 때
- 서비스에 대한 허가를 받지 못했을 때

만약 서비스 이용이 불가능하다면 그 서비스에 의존하는 app-specific한 기능을 비활성화한다. 사전에 기능을 비활성화하는 것은 더욱 서비스를 사용하고 에러에 응답하는 것보다 더욱 신뢰할만한 접근법이다.

`CLLocationManager`클래스는 각 서비스의 이용가능성을 결정하기위해 메서드를 제공한다. 그 서비스를 사용하려고 시도하기 전에 주어진 서비스를 위한 적절한 메서드를 호출한다. 
예를 들어 나침반 방향 서비스를 시작하기 전에 이용가능한지를 확인하는 `headingAvailable()`메서드를 사용한다. 만약 앱이 여러개의 서비스를 사용한다면 적절한 메서드를 각 서비스에 호출해준다.

```swift
if (CLLocationManager.headingAvailable()) {
    // 앱의 나침반 기능이 가능
} else {
    // 불가능
}
```

만약 앱이 위치서비스 없이는 기능할 수 없다면 그러한 요구사항들을 사전에 `Info.plist`파일에 명시해 놓는다. 이에 대한 내용은 [Declare any location services your app requires](https://developer.apple.com/documentation/corelocation/configuring_your_app_to_use_location_services#3384900)를 참고한다.

### 앱의 location-manager 객체를 설정
`CLLocationManager` 객체는 모든 위치서비스에 접근하는 중앙 접근 지점이다. 하나이상의 객체를 사용하고자 하는 서비스 근처에 만들고 생성하는 각 객체에 대한 강한참조를 저장한다. 
예를 들어 위치 데이터를 보여주는 뷰 컨트롤러 안에 그것을 만들고 그 객체를 적절한 파라미터와 함께 설정하고 서비스를 시작한다.

위치 데이터를 수집하는 것은 시간이 꽤 소요되기 때문에 대부분의 위치 서비스들은 비동기로 동작하고 결과를 델리게이트 객체에 알려준다. 델리게이트 객체는 `CLLocationManagerDelegate`프로토콜을 채택하고 이 프로토콜의 메서드를 사용해서 위치 값을 받고 에러를 핸들링한다. 앱에 존재하는 객체 중 하나에 이 프로토콜을 채택하거나 위치 업데이트를 관리하는데 사용하는 커스텀 객체에 이를 추가한다. 각 location manager 객체는 그것과 연관된 델리게이트 객체에 업데이트를 보낼 것이다.

location manager 객체를 생성한 이후에 대리자를 즉시 할당한다. 대리자는 서비스를 시작하기 전에 설정이 되어야 한다. 다음 예제는 `location-manager` 객체를 만들고 즉시 설정하는 예제이다.

```swift
class LocationDataManager: NSObject, CLLocationManagerDelegate {
    var locationManager = CLLocationManager()
    
    override init() {
        super.init()
        locationManager.delegate = self
    }
    
    // 위치와 관련된 프로퍼티와 델리게이트 메서드
}
```

델리게이트 객체안에서 관련된 서비스를 핸들링할 필요가 있는 모든 메서드를 구현한다. 이와 관련된 에러도 덤으로 말이다. 만약 서비스가 어떠한 이유로 위치데이터를 전달할 수 없다면 적절한 에러와 관련된 메서드를 호출한다.

### 위치 서비스 접근에 대해 허가를 요청한다.
위치 서비스를 시작하기전에 기기 소유자에게 허가를 요청해야 한다. 위치 데이터는 민감한 개인정보이기에 기기의 소유자는 어떤 앱이 접근가능하게 할 지 제어가 가능해야 한다. 허가하거나 그렇지 않을 수 있고 추후 변경할 수도 있는 것이다.

접근을 요청하기 전에 해당 요청이 현재 필요한지 아닌지를 확인하기 위해 앱의 현재 인증 상태를 확인한다.
델리게이트 객체에 location manager를 할당했을 때 location-manager 객체는 대리자의 `locationManagerDidChangeAuthorization(_:)`메서드를 호출해서 앱의 현재 인증 상태를 보고한다. 만약 앱의 인증 상태가 `CLAuthorizationStatus.notDetermined`라면 서비스를 시작하기 전에 인증을 요청한다.

### Tip
앱에서 인증요청이 필요한 부분에서 이를 요청한다. 위치와 관련된 데이터를 보여주는 뷰컨과 같은 쪽에서 말이다. 시작시점에 요청하지말고 위치와 연결되지않은 앱의 부분에서 요청하지않는다. 누군가는 앱에서 왜 인증이 필요한지에 대해 이해하지 못해서 요청을 거절할 수가 있다.

Core Location은 다양한 수준의 인증을 제공하기에 어떤 레벨을 사용할 지 골라야 하고 추가적인 정보를 제공해야 한다. 더 자세한 정보는 [Requesting authorization to use location services.](https://developer.apple.com/documentation/corelocation/requesting_authorization_to_use_location_services)를 사용한다.

### 앱이 필요로 하는 위치 서비스를 선언한다.
Core Location은 Wi-Fi, cellular, GPS 하드웨어를 사용하여 위치 업데이트를 한다. 그리고 magnetometer를 사용하여 나침반 업데이트를 한다. 위치 업데이트를 위해 Core Location은 매번 모든 하드웨어를 사용하지는 않는다. `CLLocationManager`안에 정확성 수준을 명시하고 Core Location은 가장 전력효율적인 방법을 사용한다.

만약 앱이 특정 하드웨어없이는 동작할 수 없다면 `UIRequiredDeviceCapabilities`key를 `Info.plist`파일에 추가한다. 이 키의 존재는 앱스토어에게 명시된 하드웨어나 능력 없이는 디바이스 설치를 막아달라고 요청하는 것이다. 이 키의 값은 문자열 배열이고 다음과 같은 것들을 포함할 수 있다.
- location-services
- gps
- magnetometer
만약 고수준레벨의 정밀도가 필요한 위치데이터가 필요한 경우에만 `gps` key를 추가한다. 일반적으로 오직 네비게이션 앱들은 이 `gps`가 필요하지만 다른 앱도 정밀한 위치가 필요할 때 사용할 수 있다. 만약 앱이 방향정보를 알아야 한다면 `magnetometer`key를 추가한다.

만약 사람들이 위치 데이터 없이 앱을 사용할 수 있다면 `UIRequiredDeviceCapabilities`에 key를 포함시키지 않는다. 예를 들어 만약 앱이 위치 데이터를 사용하여 근처 식당의 결과를 검색하는 것이라면 키를 포함하지 않는다. 위치 데이터가 이용가능하지 않을 때 그 데이터 없이 동작할 대안을 찾을 수 있다. 예를 들어 만약 위치에 의해 결과를 필터링하고 싶다면 누군가에게 우편 번호나 기타 지리적 정보를 명시적으로 입력하라는 메시지를 표시할 수 있다.

### 필요한 위치 서비스를 시작한다
위치 서비스 이용가능여부를 체크한후에 위치 서비스를 시작한다. Core Location은 다양한 위치와 연관된 정보를 제공한다.
- 현재 위치를 얻는다: 방향 지시를 제공하고 위치에 기반해서 데이터를 필터링하고 친구에게 위치를 공유하거나 누군가의 현재 위치로 어떤 작업을 시작한다. 자세한 건 [Getting the current location of a device.](https://developer.apple.com/documentation/corelocation/getting_the_current_location_of_a_device)를 확인한다.
- 디바이스가 지리학적인 지역에 진입되거나 나와질 때를 감지한다: 관심있는 부분을 제공하고 위치에민감한 알림을 제공한다. 자세한 건 [Monitoring the user's proximity to geographic regions.](https://developer.apple.com/documentation/corelocation/monitoring_the_user_s_proximity_to_geographic_regions)를 참고한다.
- 현재 나침반 방향을 결정한다: 코스기반 네비게이션을 제공하거나 화면에 나침반 방향을 띄운다. 자세한 건 [Getting heading and course information](https://developer.apple.com/documentation/corelocation/getting_heading_and_course_information)를 참고한다.
- iBeacon 하드웨어 근처를 감지한다: 블루투스 디바이스와의 근접성을 결정한다. 자세한 건 [Determining the proximity to an iBeacon device](https://developer.apple.com/documentation/corelocation/determining_the_proximity_to_an_ibeacon_device)를 참고한다.

## Requesting authorization to use location services
> 위치 서비스를 사용하는 허가를 얻고 앱 인가 상태의 변경사항을 관리한다.

### Overview
위치 데이터는 민감한 정보로 이 사용은 앱을 사용하는 사람들에게 privacy implication이다.
사람들이 자신의 정보를 제어할 수 있도록 하기위해 시스템은 그들이 사용허가를 할 때까지 사용을 못하도록 한다. 
이 허가과정은 소유주에게 허가를 요청하는 알림을 보여주는 한번의 interruption을 수행한다. 이 interruption이후에 시스템은 앱 인가상태를 저장하고 다시 알림을 보여주지 않는다.

사람들이 왜 위치 데이터가 필요한지를 알게하기위해 인가요청은 앱이 데이터를 필요로 할 때 해야 한다. 

### 접근레벨을 선택한다.
인가 요청을 하기전에 앱이 필요한 접근레벨을 선택한다. Core Location은 두 가지 접근레벨을 지원한다.
- `When in Use`인가는 앱을 사용할 때만 위치 업데이트가 가능하게 만든다. 이 인가는 더 privacy하고 베터리 수명도 절약하기에 선호되는 선택이다.
- `Always`인가는 어느때나 위치를 업데이트 하도록 한다. 그리고 시스템이 일부 업데이트를 하기 위해 조용히 처리하도록 한다. 이 접근레벨요청은 오직 필요할 때만 한다. 예를 들어 만약 앱이 위치 변경에 대해 자동으로 시간에 민감한 데이터를 응답해야 하는 경우 혹은 위치 푸시 서비스 앱 확장을 구현하려고 할 때 사용한다.

iOS에서 앱은 포그라운드에 있을 때와 포그라운드에서 백그라운드로 이동할 때 사용 중이다. 만약 background location updates를 활성화한다면 `When in Use`상태일 때 백그라운드에서 계속 동작한다. 만약 위치 서비스가 동작하지않으면 정지 규칙이 적용된다. 만약 시스템이 앱을 종료하거나 앱이 동작하지 않는다면 새 업데이트를 제공하기 위해 `When in Use` 권한이 있는 앱을 실행하지않는다. 

### Note
만약 앱이 이미 `When in Use`인가를 가지고있다면 `Always`인가에 대해 후에 요청할 수 있다. 하지만 신청은 1번만 가능하다. 

선택한 접근레벨과는 관계없이 현재 디바이스에 이용가능한 위치 서비스를 시작할 수 있고 같은 결과를 얻을 수 있다. 접근레벨은 주로 앱이 어떻게 업데이트를 받을지를 결정하는 것이다. 
백그라운드에서 위치 업데이트를 어떻게 처리할 지에 관한 내용은 다음을 참고한다.[Handling location updates in the background](https://developer.apple.com/documentation/corelocation/handling_location_updates_in_the_background)

### 어떻게 위치 서비스를 사용할지에 대한 설명을 제공한다.
처음 인가요청을 받을 때 시스템은 alert을 띄워서 사람들이 승인 혹은 거절을 하도록 한다. alert은 사용설명서를 표시하여 왜 이 데이터에 접근해야 하는지를 알려주도록 한다. 앱의 `Info.plist`파일을 통해 문자열을 제공할 수 있고 그 문자열을 사람들에게 알리는 것이다.

Core Location은 각 접근레벨에 대해 다양한 문자열을 지원한다. `When in Use`접근에 대한 사용 설명서를 포함해야 한다. 만약 앱이 `Always`를 지원한다면 왜 상승된 권한을 얻고 싶어야하는지에 대한 추가적인 문자열을 제공한다. 다음의 테이블은 `Info.plist`에 키로 포함되는 것들이다.

<img src="https://hackmd.io/_uploads/rk80RLPN2.png" width="500"/><br/>

모든 사용 설명서 키들을 `Info.plist`에 추가한다. 만약 요구된 키가 존재하지않으면 인가요청은 즉시 실패할 것이다.

### 인가요청을 하고 상태 변화에 대해 반응한다.
위치 서비스를 시작하기 전에 앱의 현재 인가상태를 확인하고 만약 필요하다면 인가요청을 한다. 현재 앱의 인가상태를 location-manager 객체의 `authorizationStatus`프로퍼티를 통해 얻을 수 있다. 하지만 새롭게 수정된 `CLLocationManager`객체 또한 `locationManagerDidChangeAthorization(_:)` 메서드를 통해 자동적으로 앱의 현재 인가상태를 보고받을 수 있다. 이 델리게이트 메서드를 사용해서 인가요청에 대한 제어를 한다. 다음 예시는 델리게이트 메서드는 위치 기능을 활성화할지 비활성화할지 정하는 메서드이다.

```swift
func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) { 
    switch manager.authorizationStatus {
    case .authorizedWhenInUse:  // Location services are available.
        enableLocationFeatures()
        break
        
    case .restricted, .denied:  // Location services currently unavailable.
        disableLocationFeatures()
        break
        
    case .notDetermined:        // Authorization not determined yet.
       manager.requestWhenInUseAuthorization()
        break
        
    default:
        break
    }
}
```

`locationManagerDidChangeAuthorization(_:)`메서드는 인증과관련된 변화를 처리하는 중앙공간이다. 사람들은 앱의 인가상태를 시스템 설정에서 언제든 변경할 수 있다. 만약 설정이 변경되었을 때 앱이 동작중이라면 `CLLocationManager`객체가 변화를 델리게이트 메서드에 보고한다. 그 location manager는 또한 앱의 현재 인가를 보고한다. 예를 들어 location manager는 중단된 iOS앱이 다시 시작될 때 이 메서드를 호출한다.


### 참고문서
- [Apple Docs - Core Location](https://developer.apple.com/documentation/corelocation)
- [Apple Docs - Getting the current location of a device](https://developer.apple.com/documentation/corelocation/getting_the_current_location_of_a_device)
- [Apple Docs - Configuring your app to use location services](https://developer.apple.com/documentation/corelocation/configuring_your_app_to_use_location_services)
- [Apple Docs - Requesting authorization to use location services](https://developer.apple.com/documentation/corelocation/requesting_authorization_to_use_location_services#3385302)

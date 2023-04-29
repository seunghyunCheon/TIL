# 230426 localization, Region, Language, Locale, Localization 적용하기
===
학습내용
- Localization
- Region, Language, Locale 
- Localization 적용하기

## Localization
다양한 언어와 지역을 지원함으로써 앱을 확장하는 기능

### Overview
`Localization`은 앱을 다양한 언어와 지역에 번역하고 적용하는 작업을 한다. 앱을 지역화해서 다양한 언어를 사용하는 사용자들에게 편리한 접근을 제공한다.

첫번째로 자동으로 format하고 문자열을 그 언어와 지역에 맞게 올바르게 번역해주는 API를 사용해서 국제화 시킨다. 그 후 언어 복수형 규칙에 따라 복수형 명사와 동사를 포함하는 콘텐츠에 대한 지원을 추가하여 번역의 정확성을 높인다. 

### Translate and adapt your app
Xcode에서 지역화는 특히 지원하는 언어와 지역에 대한 자원의 집합이다. 

프로젝트에 지역화를 추가하고 포함하길 원하는 지역과 언어를 고를 수 있다. 지역화를 내보내고 파일을 사용자가 보는 텍스트를 번역하고 특정 문화와 지역에 대한 자원을 적용시켜주는 `localizers`에게 보낸다.마지막으로 localized 파일들을 import하고 그 언어와 지역 환경으로 앱을 직접 테스트한다.

지역화된 버전을 출시했을 때 앱 스토어 내의 앱스토어정보에 지역화도 할 수 있다. 

## Region, Language, Locale 

### Locale
Information about linguistic, cultural, and technological conventions for use in formatting data for presentation.

* Language: 지역화에 사용될 언어를 나타내는 indentifier.
* Region: 지역화할 문화권에 대한 identifier.
* Locale: 데이터 형식 지원에 사용되는 언어, 문화, 기술 규칙에 대한 정보
</br>

### 앱에서의 작용

```swift
var components = Locale.Components(languageCode: "en", languageRegion: "GB")
components.region = Locale.Region("US")
let en_GB_US = Locale(components: components)
```

- Language: Locale의 language를 사용해 두 locale에서 동일한 언어를 사용하는지 또는 한 언어가 다른 언어의 상위 언어인지 비교할 수 있다.
- Region: 문화나 통화와 같은 항목에 규칙을 적용하기 때문에 앱에서 언어는 영어를 사용하지만 문화나 통화는 중국을 사용하는 등 더 다양하게 적용시킬 수 있다.
- Locale: Data formatting API들에게 데이터를 표시할 적합한 포멧/방식을 알려주는 객체로 사용된다.

## Localization 적용하기

### Export
- Project -> Info -> Localization + 버튼클릭 -> 지원하고자 하는 언어 선택 -> Localization적용 파일 설정
- Product -> Export Localization을 눌러 `xloc`확장자 타입으로 내보낼 수도 있다.
 
<img src="https://i.imgur.com/XrVrAC8.png" width="400"/>

### Import
- Export Localization한 파일을 지역화할 언어에 맞게 수정을 한 뒤 import를 한다. 이 방식은 해당 언어가 어떤 언어로 변경되는지 한 눈에 볼 수 있다는 장점이 있다.
- 또는 프로젝트 내부에 Export된 파일에 가서 직접 수정해준다.(Localization적용 시 자동으로 프로젝트 내부에 Export가 된다.)

### 코드로 작성한 뷰의 경우
1. `strings`파일을 만든 뒤 오른쪽 인스펙터에서 `localize`하고 지역화할 언어를 적어준다. 
```swift
// 왼쪽글자를 오른쪽으로 로컬라이징 할 거야
"Touch the button below!" = "아래 버튼을 눌러주세요!";
"Touch Me!" = "클릭!";
```
2. 사용자 지역설정을 테스트 해 볼 언어로 설정한 뒤에 NSLocalizedString으로 변환. (테스트 언어 변경은 Edit scheme -> Options에서 변경)
```swift
bottomLabel.text = NSLocalizedString("Touch the button below!", comment: "nothing")
```

### 사진 로컬라이제이션
asset의 사진을 클릭한 뒤 오른쪽 인스펙터에서 `localize`하고 해당 국가에 맞는 사진을 넣어주면 된다.




### 참고문서
- [Apple Docs - Localization](https://developer.apple.com/documentation/xcode/localization)
- [Apple Docs - Locale](https://developer.apple.com/documentation/foundation/locale)
- [Apple Archive](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPInternational/LocalizingYourApp/LocalizingYourApp.html)

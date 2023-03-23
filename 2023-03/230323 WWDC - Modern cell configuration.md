230323 WWDC - Modern cell configuration
===
학습내용
- Modern cell configuration

## WWDC - Modern cell configuration

iOS14에서 컬렉션 뷰는 데이터를 채우는 방식, 레이아웃을 정의하는 방식, 콘텐츠를 표시하는 방식이 존재한다.

<img src="https://i.imgur.com/YTEJyBT.png" width="700"/><br/>

셀의 내용과 스타일을 구성하는 새로운 API에 대한 글이다.

content configuration을 이용해서 셀의 내용을 구성하는 코드는 다음과 같다.
```swift
let cell: UITableView:Cell = ...

var content = cell.defaultContentConfiguration()
content.image = UIImage(systemName: "Start")
content.text = "Hello WWDC"

cell.contentConfiguration = content
```

가장 먼저 할 일은 cell에 `defaultContentConfiguration`을 요청하는 것이다. 이는 항상 content가 설정되지 않은 새로운 configuration을 반환한다. 
이 구성에는 table view 스타일을 기반으로 한 기본적인 스타일이 존재한다.

그리고 프로퍼티에 설정해주고 cell에 `contentConfiguration`에 이를 넣어준다.

이를 사용하기 전에는 cell의 `UIImageView`나 `UILabel`에 직접접근했지만 이를 사용함으로써 직접 건드리지 않고 새로운 configuration을 만들어서 설정된다. 

이 사용의 이점은 뭘까?

위의 코드는 테이블뷰의 셀 내용 뿐만 아니라 어떤 셀에도 적용이 가능하다. 즉 헤더나 푸터, 컬렉션 뷰 셀에도 적용이 가능한 것이다.
이는 configuration이 구성가능한 조각이기 때문이기에 표준 셀 레이아웃 모양을 지원하는 모든 셀, 뷰에 연결할 수 있는 형태로 사용될 수 있다.

다른 예시로 다른 이점을 알아보자.

<img src="https://i.imgur.com/IzFFHXB.png" width="800"/><br/><br/>

multicolumn을 이용하는 iPad App 내에 왼쪽에 Sidebar형태의 컬렉션 뷰가 존재하고 그 안에 셀이 존재한다. 셀들은 유저와의 상호작용을 통해 내용과 상태들이 다양해질 수 있는데 이를 편리하고 유연하게 업데이트 시킬 수 있는 것이 지금 알아보고 있는 content configuration의 이점이다. 

위의 예시에서는 사이드 바에서의 configuration을 사용하고 있기에 자동으로 사이드바의 기본 스타일을 상태에 맞게 구성할 수 있게 된다. 만약 사이드바의 기본적인 configuration을 사용하지 않았으면 일일히 사이드바에 해당하는 state의 색 지정과 blur처리 등을 해줘야할 것이다.

configuration은 속성들의 모음이며 뷰 또는 렌더링 할 셀에 적용하기전까지는 아무작업도 하지않는다.

### configuration types

configuration에는 배경과 내용 두 종류가 존재한다.

배경 구성의 경우 일반적인 배경모양을 빠르게 지정할 수 있는 여러 속성들이 존재한다.

#### Background Configuration

<img src="https://i.imgur.com/n2Xxdzj.png" width="400"/><br/>

- 흐림효과나 둥근 모서리를 추가하는 등의 작업이 이 구성에서 사용된다.

#### List Content Configuration
<img src="https://i.imgur.com/Bqceiyy.png" width="400"/><br/>

- UITableViewCell 스타일과 유사한 셀, 헤더, 푸터 등에 대한 표준 레이아웃을 제공한다.
- 이미지, 텍스트, 사용자 정의할 수 잇는 많은 속성을 노출하고 많은 양의 텍스트를 표시할 수 있는 유연한 레이아웃과 접근성 텍스트를 위한 특별한 레이아웃 모드와 같은 높은 수준의 동작을 제공한다. 

이 두가지 구성 유형은 다음과 같은 공통된 설계원칙을 가진다.
- 만들기 쉽다.
- 항상 새로운 구성으로 시작한다. 구성은 값 타입이기 때문에 다른 곳에서 공유될 수 없다. 따라서 직접 셀에 설정할 때까지 영향을 주지않는다.
```swift
var content = cell.defaultContentConfiguration()
```
위의 코드로 항상 새로운 구성을 시작해야 하는데 이는 이미 구성된 셀을 업데이트 할때에도 적용이 된다. 매번 새로운 구성으로 시작해서 새로운 상태를 설정하는 것이 핵심이다.

셀에 구성을 적용하면 UIKit이 어려운 작업을 대신 수행해준다. 무엇이 변경되었는지 파악하고 필요에 따라 뷰를 효율적으로 업데이트한다. 

### ✏️ Configuration의 이점 정리

### 다른 상태에 대한 다른 외형
사이드바 코드예시를 다시 살펴보았을 때, 사이드바에 대한 구성을 기본적으로 제공하고 있었기에 구성은 사이드바 뿐만 아니라 다른 경우에도 기본적인 스타일이 있다는 것을 암시할 수 있다.

### 자동적인 애니메이션과 전환
몇줄의 코드로 고급동작에 대한 접근을 제공한다.
배경색 변경에 대한 애니메이션을 주고 싶다면 애니메이션 코드 안에 background configuration만 설정해주면 된다.

### 복잡한 버그 제거
복잡한 상태 및 전환을 처리할 때 전체 종류의 버그를 제거한다. 현재 적용하고자 하는 구성이 사실이며 이를 설정하면 새 구성이 한번에 적용되기 때문이다.

### 성능 최적화
처음부터 성능을 위해 구축되었으며 부드러운 스크롤을 보장하는데 특히 중요하다. UIkit은 뷰와 렌더링을 관리하기에 내부적으로 많은 내부 성능 최적화를 구현할 수 있고 우리는 이 이점을 무료로 얻을 수 있는 것이다.

### ✏️ Configuration State(구성 상태)

구성 상태는 셀과 뷰를 구성하는데 사용하는 다양한 입력을 나타낸다. 

뷰를 설정하는 가장 일반적인 입력 중 하나는 `UITraitCollection` 특성이다.

<img src="https://i.imgur.com/74FroYV.png" width="600"/><br/>
size class, user interface style, content size category와 같은 것들이다.

이러한 특성 외에도 셀과 뷰는 다양한 상태로 존재한다.
<img src="https://i.imgur.com/tzeV8yp.png" width="600"/><br/>

위의 상태는 UIKit에서 흔히 볼 수 있는 다양한 상태이고 이것 외에도 Custom State가 존재한다.

<img src="https://i.imgur.com/tqEIFuD.png" width="600"/><br/>

shared, archived, flagged, completed, view model 등 해당 앱과 도메인에서 사용하는 상태이다. 

예를 들어 셀을 구성할 때 메세지 앱은 잘 보관되었는지, 플래그가 지정되었는지 여부를 알아야하고 결제 앱은 처리된 트랜잭션을 표시할 수 있어야 한다. 

그렇다면 구성상태는 어디에서 찾을 수 있을까?

<img src="https://i.imgur.com/n8uMDrM.png" width="600"/><br/>

컬렉션, 테이블 뷰에서 헤더는 `View Configuration State`를 cell들은 각각 다른 `Cell Configuration State`를 가지고 있다.

### View Configuration State
View의 Configuration State의 상태는 다음과 같다.

<img src="https://i.imgur.com/6amfPyH.png" width="500"/><br/>

강조된 상태, 선택된 상태 등 기본적인 상태들이 있고 사용자 지정 상태가 존재한다.

### Cell Configuration State

<img src="https://i.imgur.com/q8tcIhu.png" width="500"/><br/>

View의 구성상태를 전부 가져온 뒤 셀의 특성인 편집하거나 스와이프하는 등의 상태가 포함되어있다.

### ✏️ Automatic configuration updates(자동으로 구성을 업데이트)

```swift
var automaticallyUpdatesContentConfiguration: Bool { get set }
var automaticallyUpdatesBackgroundConfiguration: Bool { get set }
```

기본적으로 위 프로퍼티가 활성화되어있기 때문에 셀에 배경이나 컨텐츠 구성을 설정하면 셀의 구성상태가 변경될때마다 자동으로 업데이트된 버전을 반환한 다음 새 구성을 다시 셀에 적용하도록 한다.

이러한 자동 업데이트는 기본 스타일의 상태를 얻는데에는 유용하지만 다른 상태에 대한 모양을 사용자 지정으로 하는 경우에는 이 속성을 비활성화하고 구성을 직접 업데이트할 수 있다.

셀의 `cellForRow` 델리게이트 메서드에서는 셀의 데이터들을 넣어놓고, 위 자동으로 configuration을 업데이트하는 프로퍼티가 true로 되어있다면 `updateConfiguration(state)`가 호출이 되어 현재 상태가 인자로 들어가 스타일을 설정할 수 있다.

즉 셀의 데이터를 설정하는 부분과 외관을 설정하는 부분이 분리된 것이다. 

`updateConfiguration`은 셀의 하위 클래스에서 재정의해서 사용된다.

```swfit
class CustomCell: UITableViewCell {
    override func updateConfiguration(using state: UICellConfigurationState)
}
```

이 메서드는 셀이 처음 표시되기 전에 항상 호출되며 구성 상태가 변경될때마다 다시 호출이 되므로 새 상태에 맞게 외관을 수정할 수 있다.

그리고 만약 셀의 Configuration을 완전히 재구성 해야하는 경우 `setNeedsUpdateConfiguration`을 호출할 수 있다.

### 다른 상태에 대한 외관 커스터마이즈
```swift
override func updateConfiguration(using state: UICellConfigurationState) {
    var content = self.defaultContentConfiguration().updated(for: state)
    
    content.image = self.item.icon
    content.text = self.item.title
    
    if state.isHighlighted || state.isSelected {
        content.imageProperties.tintColor = .white
        content.textProperties.color = .white
    }
    
    self.contentConfiguration = content
}
```

1. 셀에 기본 컨텐츠 구성을 요청해서 새로운 구성을 얻는다. 
2. 현재 상태에를 기반으로 셀의 스타일을 변경한다.
3. contentConfiguration에 변경한 스타일을 넣어준다.

처음 봤던 사이드바의 코드로 다시 돌아와서 Default Configuration을 살펴보자.

컬렉션 뷰의 셀, 테이블 뷰 셀, 헤더, 푸터의 기본 background Configuration은 포함하는 테이블 뷰의 스타일을 기반으로 자동으로 구성한다. 그렇기에 원하는 배경모양을 얻기 위해 아무것도 할 필요가 없다.

반면에 ContentConfiguration은 `defaultContentConfiguration`을 이용해서 셀의 스타일을 기반으로 새로운 구성을 가져올 수 있다.

그래도 배경 과 컨텐츠 구성 모두에 대해 기본구성을 다음의 코드처럼 쉽게 요청할 수 있다.
```swift
var background = UIBackgroundConfiguration.listSidebarCell()
var content = UIListContentConfiguration.sidebarCell()
```

Configuration을 사용하면 Dynamic type에 대응되는 것에 더하여 텍스트가 이미지 주위를 감싸는 특수 모드를 사용할 수도 있다. 가장 큰 접근성 텍스트 크기를 켜면 셀 레이아웃이 콘텐츠에 사용할 수 있는 공간을 최대화하기 때문이다.

<img src="https://i.imgur.com/RogI2FM.png" width="700"/><br/>

이게 가능한 이유는 Content Configuration은 환경에 따라 높이가 유연할 수 있는 self-sizing cell과 함께 사용되도록 설계되었기 때문이다. 

<img src="https://i.imgur.com/EZjkl9h.png" width="700"/><br/>

Content Configuration을 통해 파란색으로 표시된 레이아웃 마진과 주황색으로 표시된 다양한 패딩 속성을 제어할 수 있다.

고정된 높이를 제공하는 것 보다는 동적으로 레이아웃에 영향을 줄 수 있는 것이 중요하다.

레이아웃 개념을 하나 더 알아보자

<img src="https://i.imgur.com/nISDXR6.png" width="400"/><br/>

위 사진은 안에 이미지의 trailing으로부터 padding을 주고 있기 때문에 텍스트들이 정렬이 되지 않고 있다. 이 상태에서 올바른 정렬을 얻기 위해 각 이미지에 대해 예약된 레이아웃 크기를 지정하면 된다.

예약된 레이아웃의 너비를 설정하면 이 공간안에서 이미지를 가로로 중앙배치한다.

<img src="https://i.imgur.com/f0ULv63.png" width="400"/><br/>

빨간색 점선 사이의 거리는 예약된 레이아웃 너비이며 이 크기는 이미지의 실제 크기에 영향을 주지않는다. 그리고 이미지가 영역보다 큰 경우 영역 밖으로 나간다. 
그리고 레이아웃 영역을 기준으로 padding이 설정되기에 텍스트도 정렬되는 것을 볼 수 있다.


Configration과 기존 셀에 있는 속성을 베타적이기에 구성을 설정하면 기존의 속성이 nil이 된다. 그 반대도 마찬가지이다. 그렇기에 이 둘을 혼합해서 사용하지 않아야 한다.


### 참고자료
- [WWDC - Modern cell configuration](https://developer.apple.com/videos/play/wwdc2020/10027/)

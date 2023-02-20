230220 HIG(List and tables), 부스트코스 야곰 테이블뷰, 테이블 뷰 Grouped Style
TIL (Today I Learned)
===
학습내용
---
- HIG(List and tables) 정독
- 부스트코스 야곰 테이블뷰 학습
- 테이블 뷰 Grouped Style

## HIG(List and tables)
> 여러개의 행들의 단일, 다중 열 데이터를 보여준다.

table, list는 계층별로 정돈된 데이터들을 표현하고 CRUD와 같은 사용자 인터렉션을 가능하게 한다. 

때떄로 복잡한 데이터를 다룰때는 다중 열 테이블이나 스프레드시트를 사용해야 한다. 

### ✏️ Best practices

#### list or table안에서 텍스트를 보여주는 것을 선호한다. 
한 테이블은 어떤 타입의 내용을 포함하지만 row-based 포멧의 테이블은 더 쉽게 스캔하고 읽을 수 있다. 만약 다양한 사이즈를 가지는 아이템들을 화면에 보여주고 싶거나 많은 양의 이미지를 보여주고 싶다면 `Collection`을 고려한다.

#### 테이블을 편집할 수 있도록 한다.
아이템들을 추가하거나 지우지는 못하더라도 정렬할 수 있도록 만든다.

#### item을 선택할 때 적절한 피드백을 제공한다.
피드백의 종류는 새로운 화면을 보여준다거나 item상태를 변경시키는 등 다양하다. 일반적으로 계층구조를 navigate하는 테이블뷰의 경우 사람들의 현재 경로를 명확히 하기위해 영구적으로 선택된 행을 강조한다. 반면에 옵션들을 열거한 테이블뷰의 경우 체크표시처럼 짧게만 강조한다. 

### ✏️ Content

#### 행 내용이 편하게 읽히도록 간결한 텍스트를 작성한다.
짧고 간결한 텍스트는 짤림과 줄 바꿈을 최소화한다. 만약 아이템마다 큰 양의 텍스트를 가질 경우 제목만 나열하고 더 많은 부분은 디테일 뷰와같은 공간으로 위임을 넘긴다.

#### 텍스트가 짤릴 수 있는 경우 가독성을 보존하는 방법을 고려한다.
예를 들어 텍스트의 크기가 다양한 경우 텍스트들을 전부 보존하길 원한다면, 가운데 텍스트에 ...을 주어 시작과 끝을 보존할 수 있도록 만든다.

#### 다중 열 테이블에는 column heading을 사용한다.
명사형태의 대문자 타이틀을 사용하고 문장부호는 사용하지않는다. 만약 단일 column 테이블뷰에서 column heading을 사용하지않는다면 label이나 header를 사용해 사람들이 이해하기 쉽도록 만든다.

## 부스트코스 야곰 테이블 뷰

### ✏️ 테이블 뷰란?

테이블 뷰는 하나의 열과 여러 줄의 행을 가지고 있으며 수직으로만 스크롤이 가능하다.
각 행은 하나의 셀에 대응하며 섹션을 이용해 행을 시각적으로 나눌 수 있다. 
만약 행에 대한 추가적인 정보를 원한다면 헤더와 푸터를 이용해 정보를 보여줄 수 있다.

### 테이블뷰 스타일

테이블 뷰의 style 프로퍼티
```swift
var style: UITableView.Style { get }
```

style 프로퍼티의 타입
```swift
enum Style: Int, @unchecked Sendable
```

`init(frame:style:)`을 통해 테이블 스타일을 정할 수 있다. 후에는 수정할 수 없다.

- plain: 순수 테이블 뷰
- grouped: 섹션들이 그룹별로 되어있는 테이블 뷰
- insetGrouped: 섹션들이 둥근 모서리로 되어있는 테이블 뷰

보통 plain과 grouped를 사용하고 plain의 경우 색인을 이용한 빠른 탐색을 하거나 옵션을 선택할 때 사용하고, grouped는 정보를 특정 기준에 따라 개념적으로 구분할 때 적합하다.

### 테이블뷰 생성
테이블뷰를 생성하고 관리하는 좋은 방법은 스토리보드에서 UITableViewController 객체를 이용하는 것이다. 스토리보드에서 테이블뷰의 특성을 지정할 때 동적 프로토타입이나 정적 셀 중 선택할 수 있으며 보통은 동적 프로토타입으로 구현을 한다.

- 동적 프로토타입
    - 셀 하나를 디자인해 이를 다른 셀의 템플릿으로 사용
    - 같은 레이아웃에 여러 개를 표시할 경우
    - 데이터소스 인스턴스에 의해 관리하며 셀의 개수가 상황에 따라 동적일 때 사용
- 정적 셀
    - 고유 레이아웃과 고정된 수의 행을 가질 때 사용
    - 셀의 개수가 변하지 않음


### ✏️ 테이블뷰 셀이란?
테이블뷰를 이루는 개별적인 행으로 `UITableViewCell`클래스를 상속받는다. 

### 테이블뷰 셀의 구조
기본적으로 테이블뷰 셀은 콘텐츠 영역과 액세서리뷰 영역으로 구조가 나뉜다.
- 콘텐츠 영역: 문자열, 이미지, 고유 식별자 등
- 액세서리 뷰: 상세보기, 재정렬, 스위츠 등 컨트롤 객체

편집모드를 하는 경우에는 편집 컨트롤이 생기는데 이는 아래 사진의 세번째 사진처럼 컨텐츠 왼쪽에 Delete하는 컨트롤이 있거나 오른쪽처럼 재정렬하는 컨트롤을 하는 역할을 한다.<br/>
<img src="https://i.imgur.com/ZxnvEZr.png" width="500"/>

### 테이블뷰 셀의 기본 기능

- `UITableViewCell`클래스를 상속받는 기본 테이블뷰 셀은 표준 스타일을 이용할 수 있다. 표준 스타일은 이미지, 문자열, 액세서리뷰로 되어있으며 이미지가 오른쪽으로 확장됨에 따라 문자열이 오른쪽으로 밀려난다.
- `UITableViewCell` 클래스는 셀 콘텐츠에 세 가지 프로퍼티가 정의되어 있다.
    - `textLabel`: 주제목 레이블
    - `detailTextLabel`: 추가 세부사항 표시를 위한 부제목 레이블
    - `imageView`: 이미지 표시를 위한 뷰

### 커스텀 테이블뷰 셀
셀을 커스텀하는 방법에는 두 가지가 존재한다.
- 셀의 콘텐츠뷰에 서브뷰 추가
- `UITableViewCell`의 커스텀 서브클래스 만들기

`UITableViewCell`의 서브클래스를 이용해 커스텀 이미지뷰를 생성하는 경우, 이미지뷰의 변수명을 `imageView`로 명명하면 원하는 대로 동작되지않을 수 있으니 좀 더 구체적인 네이밍을 한다.

### ✏️ DataSource, Delegate

### DataSource
- `UITableViewDataSource` 프로토콜을 채택
- 테이블 뷰를 생성하고 수정하는데 필요한 정보를 테이블 뷰에게 제공
- 테이블뷰의 시각적 모양에 대한 최소한의 정보를 제공
- 섹션의 수와 행의 수를 알려주며, 행의 삽입, 삭제 및 편집이 가능한지, 재정렬하는지, 행을 다른 위치에 옮기는지에 대한 기능을 구현할 수 있다.

### Delegate
<img src="https://i.imgur.com/bt41znD.png" width="600"/>


- `UITableViewDelegate` 프로토콜을 채택
- 테이블뷰의 시각적인 부분 수정, 행의 선택 관리, 액세서리 뷰를 지원. 그리고 개별 행 편집을 도와준다.
- 테이블뷰의 세세한 부분을 조정할 수 있다.

데이터소스는 특정 기능을 가능하게 만들지에 대한 여부를 묻는 것이 많고 델리게이트를 이에 대해 어느 수준을 사용할지 구현하는 부분에 대한 메서드들이 많다.

그리고 데이터소스는 테이블뷰의 시각적인 셀을 만드는 것에 초점을 많이 두고, 델리게이트는 사용자와의 상호작용에 반응하는 메서드들이 많다.

### 테이블 뷰 새로운 셀 삽입

기존 테이블뷰에서 새로운 셀을 삽입하려면 `numberOfSections`와 `numberOfrawsInSection` 메서드를 수정해야 한다. 그리고 셀을 반환하는 코드에서 처리를 해줘야 한다.

#### 테이블 뷰에서 특정 섹션만 애니메이션을 줘서 reload

```swift
self.tableView.reloadSections(IndexSet(2...2), with: UITableViewRowAnimation.automatic)
```

### ✏️ 뷰의 재사용이란?

iOS기기는 한정된 메모리를 가지고 애플리케이션을 구동한다. 만약 많은 양의 데이터에 해당하는 뷰를 전부 그리게 된다면 많은 메모리 낭비가 일어날 것이다. 그래서 테이블뷰에서는 뷰를 재사용하여 메모리를 절약하고 성능을 향상할 수 있다.(ex. `UITableViewCell`, `UICollectionViewCell`)

### 재사용 원리
1. 테이블뷰 및 컬렉션뷰에서 셀을 표시하기 위해 데이터 소스에 뷰 인스턴스를 요청한다.
2. 데이터소스는 새로운 셀을 만드는 대신 재사용 큐에 대기하고 있는 셀이 있는지 확인 후 있으면 그 셀에 새로운 데이터를 설정하고, 없으면 새로운 셀을 생성합니다.
3. 테이블뷰 및 컬렉션뷰는 데이터 소스가 셀을 반환하면 화면에 표시한다.
4. 사용자가 스크롤하면 일부 셀들이 화면 밖으로 사라지면서 다시 재사용 큐에 들어간다.
5. 1~4번 반복

<img src="https://i.imgur.com/Mi9Cjza.png" width="500"/>


## 테이블 뷰 Grouped Style

테이블 뷰의 스타일은 `plain`과 `grouped`를 많이 사용하는데 `plain`을 사용할 경우 헤더를 추가한다면 이상하게 스크롤 시 헤더가 따라오게 된다. 이에 대해 스타일을 `grouped`로 바꿔준다면 스크롤 시에 헤더가 따라오지 않게 된다. 

하지만 헤더에만 어떤 색을 주고싶은데 푸터까지 view가 생기게 된다.(희미하지만 footerView에 공간이 있다.)

<img src="https://i.imgur.com/XjHDin9.png" width="200"/><br/>

어떤 이유에서 생기는지는 모르겠지만 테이블 뷰의 footerView의 height를 0으로 준다면 문제가 해결된다.

```swift
tableView.sectionFooterHeight = 0
```




### 참고자료
- [HIG - List and Tables](https://developer.apple.com/design/human-interface-guidelines/components/layout-and-organization/lists-and-tables/)
- [부스트코스 야곰 - 테이블뷰](https://www.boostcourse.org/mo326/lecture/16860?isDesc=false)
- [Apple Developer Documentation - UITableView.style](https://developer.apple.com/documentation/uikit/uitableview/1614913-style)
- [UIKit - UITableView 를 Grouped Style 로 지정했을 때 생기는 Footer 없애기](https://kasroid.github.io/posts/ios/20200905-uikit-deleting-unwanted-spaces-in-grouped-style-uitableview/)

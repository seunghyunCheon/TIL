230219 Table views, Filling a table with data, JSON, Codable, JSONDecoder, Using JSON with Custom Types, Encoding and Decoding Custom Types 아티클
===
TIL (Today I Learned)
===
학습내용
---
- 테이블뷰 학습하기
- Filling a table with data 아티클 읽기
- JSON
- Codable, JSONDecoder
- Using JSON with Custom Types 샘플 코드
- Encoding and Decoding Custom Types

## Table views
> 커스터마이징한 행들의 단일 열안에서 데이터를 보여주는 View

테이블 뷰는 섹션별로 나누어져 단일 열의 수직으로 스크롤링이 가능한 내용들을 보여준다.
각 테이블의 행은 앱과 관련된 단일 조각을 보여주고 테이블의 섹션은 그 행들을 그룹화시킨다. 예를 들어 연락처앱은 사용자 연락처 이름을 보여주기위해 테이블뷰를 사용한다.


<img src="https://i.imgur.com/zjRmMaf.png" width="700"/>

테이블 뷰들은 다른 객체들 사이의 협동이다. 객체들은 다음과 같다.
- Cells. 하나의 셀은 시각적인 내용을 제공한다. UIKit에 의한 기본 셀을 사용하거나 필요에따라 커스텀해서 사용한다.
- Table view controller. 일반적으로 테이블뷰를 관리하기 위해 `UITableViewController`를 사용한다. 다른 뷰컨트롤러를 사용할 수도 있지만 테이블과 관련된 특정 작업들이 필요해진다.
- Data Source object. 이 객체는 `UITableViewDataSource` 프로토콜을 채택하고 테이블에 데이터를 제공한다. 
- Delegate object. 이 객체는 `UITableViewDelegate`프로토콜을 채택하고 테이블 내용과 연관된 유저 인터렉션을 관리한다.

## Filling a table with data
> data source객체를 사용하여 동적으로 셀을 만들고 설정하거나 스토리보드에서 정적으로 제공한다.

테이블 뷰들은 인터페이스의 데이터 기반 요소이다. data source 객체를 사용하여 각 화면 렌더링에 필요한 뷰들과 함께 데이터를 제공한다. 그 테이블 뷰는 뷰들을 관리하고 데이터를 최신으로 유지하는 역할을 한다.

테이블 뷰들은 데이터를 행과 섹션으로 정리한다. 행들은 item들의 각각 데이터를 보여주고 섹션들은 관련된 행들을 그룹화시킨다. 섹션은 필수가 아니지만 데이터를 계층적으로 만드는데 좋은 방법이다. 예시로 연락처앱은 하나의 행에서 각 내용을 보여주고 그룹화된 행들은 사용자의 성 첫글자를 기준으로 정렬되어있다.

<img src="https://i.imgur.com/Yn0X0A3.png" width="300"/>

### ✏️ Provide the numbers of rows and sections
화면에 보여지기전에 테이블 뷰는 행과 섹션들의 총 숫자를 명시하라고 요구한다. Data source 객체는 이 정보를 제공한다.
```swift
func numberOfSections(in tableView: UITableView) -> Int // Optional
func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int
```

이 메서드들의 구현들안에서 가능한한 빠르게 행과 섹션들의 수를 반환해야 한다. 이렇게 하려면 정보를 쉽게 검색할 수 있는 방식으로 데이터를 구조화해야 할 수도 있다. 예를 들어 테이블 뷰의 데이터를 관리하기 위해 array를 고려해야 한다. Array는 테이블뷰 그 자체의 본질 구조와 매치가 되기 때문에 각 행들과 섹션들을 관리하기에 좋은 자료구조이다. 

아래 예시는 다중섹션에서의 행과 섹션들의 수를 반환하는 data source 메서드의 예시를 보여준다. 이 테이블안에서 각 행은 문자열을 보여준다. 그래서 각 섹션들에 전달되는 문자열 배열을 담고 있다. 섹션들을 관리하기 위해 2차원 배열을 사용한다. 섹션들의 수를 얻기위해 data source는 `hierarchicalData` 배열의 수를 반환하고, 구체적인 섹션안에서 행의 수를 반환하기위해 data source는 자식 배열에서 item들의 수를 반환한다. 

```swift
var hierarchicalData = [[String]]() 
 
override func numberOfSections(in tableView: UITableView) -> Int {
   return hierarchicalData.count
}
   
override func tableView(_ tableView: UITableView, 
                        numberOfRowsInSection section: Int) -> Int {
   return hierarchicalData[section].count
}
```

### ✏️ Define the appearance of rows
테이블뷰의 행들의 외관을 스토리보드 파일에서 셀을 사용하여 정의한다. 하나의 셀은 `UITableViewCell`객체이며 이는 테이블에서 행들의 템플릿의 역할을 한다.
셀들은 view이며 내용을 보여줄 라벨이나 이미지 뷰와 같은 어떠한 서브뷰들을 포함한다. 그리고 이러한 뷰들은 제약을 통해서 배치가 된다.

테이블뷰를 앱 인터페이스에 추가할 때 설정할 하나의 셀을 포함해야 한다. 더많은 프로토타입 셀들을 추가하기 위해서는 테이블뷰를 선택하고 프로토타입 셀 attribute를 업데이트해야 한다. 각 셀들은 외관 정의하는 스타일을 가지고 있다. 

다음 그림은 표준 셀 스타일인 두개의 프로토타입 셀을 보여준다.<br/> 
<img src="https://i.imgur.com/Jfee8fW.png" width="300"/><br/>

스토리보드 파일안에서 각 프로토타입 셀은 다음의 행동을한다.
- 셀 스타일을 커스텀해서 설정하거나 표준 셀 스타일을 설정한다.
- 셀의 식별자 attribute에 비어있지 않은 문자열을 할당한다.
- 커스텀 셀들을 위해 셀에 뷰들과 제약을 추가한다.
- Identity 인스펙터에 커스텀 셀들의 클래스를 명시한다.

커스텀 뷰 셀을 만들 때 `UITableViewCell`을 서브클래싱해서 정의한다. 그리고 커스텀뷰를 스토리보드 파일에 연결시켜야 한다. 

### ✏️ Create and configure cells for each row

화면에 테이블뷰가 보여지기 전에 data source는 행에 필요한 셀들을 제공한다. data source 객체의 `tableView(_:cellForRowAt:)`메서드는 빠르게 반응해야 하며 이 메서드는 다음의 패턴으로 구현한다.
1. 셀 객체를 회수하기 위해 테이블 뷰의 `dequeueReusableCell(withIdentifier:for:)`를 호출한다.
2. 앱의 커스텀 데이터와 함께 셀의 뷰를 설정한다.
3. 테이블뷰의 셀을 반환한다.

표준 셀 스타일의 경우 설정할 필요가 있는 view들을 담은 프로퍼티를 포함한다. 커스텀 뷰의 경우에는 디자인 타임에 뷰들을 추가하고 그것들을 연결시켜야 한다.

아래 예시코드는 단일 텍스트 라벨을 포함하는 셀을 설정하는 data source 메서드의 예시이다. 셀은 표준 스타일중 하나인 Basic Style을 사용하고 이 스타일에는 `textLabel`프로퍼티가 라벨 뷰를 포함하고 있다.

```swift
override func tableView(_ tableView: UITableView, 
                        cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    // Ask for cell of the appropriate type.
    let cell = tableView.dequeueReusableCell(withIdentifier: "basicStyleCell", for: indexPath)
}
    // Configure the cell's contents with the row and section number.
    // The Basic cell style guarantees a label view is present in textLabel
    cell.textLabel!.text = "Row \(indexPath.row)"
    return cell
```
테이블 뷰는 테이블의 각행들에 대해 셀을 만들도록 요구하지 않는다. 대신에 테이블뷰는 셀을 lazy하게 관리한다. 오직 시각적으로 보이는 부분만을 요청하는 것이다. 셀을 lazy하게 만드는 것은 테이블이 사용하는 메모리의 양을 줄여준다. 하지만 이것은 data source 객체가 셀을 빠르게 만들어야 한다는 것을 의미한다. 
그러므로 `tableView(_:cellForRowAt:)`메서드를 테이블 데이터를 로드하거나 길이 작업을 수행할 때 사용하지 않아야 한다. (로드하는 부분은 다른 부분에서 처리되어야 함.)

### 👏 Note
표준 셀 스타일 사용에 더하여 커스텀 셀을 원한다면 다음의 아티클을 살펴보자. [Configuring the cells for your table](https://developer.apple.com/documentation/uikit/views_and_controls/table_views/configuring_the_cells_for_your_table)

## JSON(JavaScript Object Notation)

메모리 위에 올라와있는 0과1의 객체들은 현재 자신의 시스템(macOS,iOS)에서만 사용할 수 있는 모양이기 때문에 다른 컴퓨터에서도 그대로 해석할 수 없다. 

그래서 현재 시스템에 있는 데이터들을 하드디스크에 저장하거나 다른 컴퓨터로 전송하려고 할 때 이해할 수 있도록 하기위해 나타난 것이 `JSON`이다. 이는 약속된 0과1의 형태로 저장을 하는 것으로 여러 시스템에서 이 약속을 지킨 형태로 데이터가 오기만 한다면 해석할 수 있게 만드는 것이다. 

<img src="https://i.imgur.com/Ls4HVAr.png" width="500"/><br/>

위의 그림을 보면 스위프트에서 생성된 객체와 전달된 Java의 객체에 해당하는 비트가 다른 것을 볼 수 있다. 이는 스위프트의 객체를 JSON형태로 변환했을때 또다른 비트를 가지게 되고 자바에서 이 JSON을 자바시스템에 맞게 변경을 할 때 달라지기 때문이다.

JSON은 컴퓨터와의 연결을 하는 장점 뿐만 아니라 위 그림처럼 사람이 보기 쉬운 형태로되어있다는 장점도 있다.

### JSON 규칙
{ }: 객체(딕셔너리)
[ ]: 배열
" ": 문자열
문자열 외: 숫자

### Codable
> 외부표현으로 변환하거나 외부표현으로부터 변환될 수 있는 타입
```swift
typealias Codable = Decodable & Encodable
```

`Codable`은 `Encodeable`과 `Decodable`을 묶은 별칭이다. `Codable`을 하나의 타입으로 사용하거나 제네릭 제약으로 사용한다면 두 개의 프로토콜을 따르는 어떤 타입과도 매치가 된다.

### JSONDecoder
> JSON 객체인 데이터 타입 인스턴스를 decode하는 객체

아래 예시는 JSON객체로부터 `GroceryProduct`를 어떻게 decode하는지 보여준다.
그 타입은 `Codable`을 채택하고 있기 떄문에 `JSONDecoder`인스턴스를 사용해서 decode할 수 있게 된다.

```swift
struct GroceryProduct: Codable {
    var name: String
    var points: Int
    var description: String?
}
let json = """
{
    "name": "Durian"
    "points": 600,
    "description": "A fruit with a distinctive scent."
}
""".data(using: .utf8)!

let decoder = JSONDecoder()
let product = try decoder.decode(GroceryProduct.self, from: json)

print(product.name) // Prints "Durian"
```

#### Decoding
```swift
func decode<T>(T.Type, from: Data) -> T
```
JSON객체로부터 decode해서 명시한 타입으로 반환을한다.

## Using JSON with Custom Types
> 스위프트의 JSON 지원을 사용해 구조와 관계없이 JSON 데이터를 Encode하고 Decode한다.

### OverView
다른 앱, 서비스, 파일로부터 받은 JSON 데이터들은 다른 형태와 구조로 올 수 있다. 이러한 JSON 데이터와 앱의 모델 데이터간의 차이를 다루기 위해 이를 사용해야 한다.

<img src="https://i.imgur.com/xedCvy1.png" width="500"/>

이 샘플은 간단한 데이터 유형을 정의하고 여러가지 다른 JSON 형식에서 해당 유형의 인스턴스를 구성하는 방법을 보여준다.

```swift
struct GroceryProduct: Codable {
    var name: String
    var points: Int
    var description: String?
}
```

### Read Data from Arrays
스위프트의 표현타입시스템을 이용해서 수동으로 동일한 구조의 객체컬렉션을 순회하는 것을 피한다. 

```swift
[
    {
        "name": "Banana",
        "points": 200,
        "description": "A banana grown in Ecuador."
    }
]
```

### Change Key Names

JSON 키는 커스텀타입의 이름 프로퍼티와는 관계없이 JSON key와 매핑이 된다. 아래 플레이그라운드에서는 `product_name`이 `GroceryProduct`에서 `name`와 매핑이 된다.

```swift
{
    "product_name": "Banana,
    "product_cost": 200,
    "description:": "A banana grown in Ecuador." 
}
```
Custom 매핑은 스위프트 모델에서 `API Design Guidelines`를 따를 수 있게 만든다.

### Access Nested Data
앱에서 필요하지 않는 JSON코드의 데이터와 구조는 무시한다. 아래 플레이그라운드는 중간 타입을 사용하여 원치않는 데이터와 구조를 건너뛰는 것을 볼 수 있다.

```swift
[
    {
        "name": "Home Town Market",
        "aisles": [
            {
                "name": "Produce",
                "shelves": [
                    {
                        "name": "Discount Produce",
                        "product": {
                            "name": "Banana",
                            "points": 200,
                            "description": "A banana that's perfectly ripe."
                        }
                    }
                ]
            }
        ]
    }
]
```

### Merge Data at Different Depths
`Encodable`과 `Decodable`프로토콜의 요구사항을 커스텀 구현함으로써 다른 depth의 JSON구조의 데이터를 결합하거나 분리한다. 아래 플레이그라운드는 `GroceryProduct` 인스턴스가 어떻게 구성되는지 보여준다.

```swift
{
    "Banana:" {
        "points": 200,
        "description": "A banana grown in Ecuador." 
    }
}
```

## Encoding and Decoding Custom Types
> JSON과 같은 외부표현과 함께 호환될 수 있도록 데이터타입을 encodable하고 decodable하게 만든다.

### Overview
많은 프로그래밍 작업은 네트워크 연결을 통해 데이터를 전송하고, 디스크에 데이터를 저장하고, API 서비스에서 데이터를 전송하는 등의 작업을 포함한다. 이러한 작업들은 데이터가 전송될 때 중간의 특정 포멧형식이 되어 데이터를 encode, decode될 수 있기를 요구한다. 

스위프트 표준 라이브러리는 이에 대해 표준화된 접근법을 정의한다. `Codable`프로토콜만 채택한다면 외부 표현으로부터 encode하거나 decode할 수 있다. Swift4 이후로 decode하는 방식이 훨씬 간단하고 편리해졌다.

### Encode and Decode Automatically
타입을 codable하게 만드는 가장 간단한 방법은 그것의 프로퍼티들이 이미 `Codable`인 것을 사용하는 것이다. 이들은 스위프트 표준 라이브러리 타입인 `String`, `Double` 그리고 Foundation타입인 `Date`, `Data`, `URL`이 있다. 프로퍼티들이 Codable을 따른다면 단지 프로토콜을 선언만해도 준수하게 된다.

```swift
struct Landmark: Codable {
    var name: String
    var foundingYear: Int
    
    // Landmark now supports the Codable methods init(from:) and encode(to:), 
    // even though they aren't written as part of its declaration.
}
```

`Codable`을 채택하면 내장된 데이터 포맷이나 커스텀한 encoder, decoder에 의해 제공되는 형태로 serialize할 수 있다.
예를 들어 `Landmark` 구조체는 `PropertyListEncoder`나 `JSONEncoder`클래스에서 encode될 수 있다. `Landmark`가 구체적으로 propertylist나 json으로 다루는 코드를 포함하지 않는데도 말이다.

`codable`프로토콜을 따르는 커스텀타입을 프로퍼티를 갖는 경우에도 같은 원리가 적용된다. 결국 프로퍼티가 `Codable`프로토콜을 따르기만 하면 된다.

```swift
struct Coordinate: Codable {
    var latitude: Double
    var longitude: Double
}

struct Landmark: Codable {
    // Double, String, and Int all conform to Codable.
    var name: String
    var foundingYear: Int
    
    // Adding a property of a custom Codable type maintains overall Codable conformance.
    var location: Coordinate
}
```

내장된 타입인 `Array`, `Dictionary`, `Optional`들이 codable type을 포함할때마다 `Codable`을 따른다. 

```swift
struct Landmark: Codable {
    var name: String
    var foundingYear: Int
    var location: Coordinate
    
    // Landmark is still codable after adding these properties.
    var vantagePoints: [Coordinate]
    var metadata: [String: String]
    var website: URL?
}
```

### Encode or Decode Exclusively
몇가지 케이스에서 encoding과 decoding 둘 다 지원할 필요가 없는 경우가 있다. 예를 들어 네트워크 API를 호출만을 하는 앱은 decode할 필요가 없고, 데이터를 읽기만 하려는 경우 encode할 필요가 없다. 이 경우 `Codable`이 아닌 `Encodable`, `Decodable`을 사용한다.

```swift
struct Landmark: Encodable {
    var name: String
    var foundingYear: Int
}

struct Landmark: Encodable {
    var name: String
    var foundingYear: Int
}
```

### Choose Properties to Encode and Decode Using Coding Keys

codable 타입은 `CodingKey`프로토콜을 준수하는 `CodingKeys`라는 중첩 열거형을 선언할 수 있다. 이 열거형이 있을때 그것의 케이스들은 codable 타입이 encode, decode될 때 포함되어야하는 프로퍼티 리스트들의 역할을 한다.

`decoding` 인스턴스나 `encoding` 표현에 포함시키지 않을 것이라면 `CodingKeys` case에서 생략해도 된다. 

만약 serialized된 데이터 포맷의 프로퍼티(JSON 프로퍼티)가 앱의 데이터타입과 매치되지 않는다면 `CodingKeys`에 raw-value의 형태로 `String`을 명시할 수 있다. 이 raw-value는 encoding, decoding하는 경우 실제 키 이름으로 사용된다. 그렇기 때문에 rawValue를 JSON data로 하고 case의 이름은 `Swift API Design Guidelines`를 따르게 만들 수 있다. 
아래 예시는 `CodingKeys`를 이용한 예시이다.

```swift
struct Landmark: Codable {
    var name: String
    var foundingYear: Int
    var location: Coordinate
    var vantagePoints: [Coordinate]
    
    enum CodingKeys: String, CodingKey {
        case name = "title"
        case foundingYear = "founding_date"
        case location
        case vantagePoints
    }
}
```


### 참고자료
- [Apple Developer Documentation - TableViews](https://developer.apple.com/documentation/uikit/views_and_controls/table_views)
- [Apple Developer Documentation - Fiiling a table with data](https://developer.apple.com/documentation/uikit/views_and_controls/table_views/filling_a_table_with_data)
- [Apple Developer Documentation - JsonDecoder](https://developer.apple.com/documentation/foundation/jsondecoder)
- [Apple Developer Documentation - Using JSON with Custom Types](https://developer.apple.com/documentation/foundation/archives_and_serialization/using_json_with_custom_types)

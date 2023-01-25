230125 SOLID(SRP, OCP, LSP, ISP, DIP)
===
TIL (Today I Learned)
===
학습내용
---
- Solid 학습하기
## 고민한 점 / 해결 방법
## Solid
> 유지보수하기 쉽고 유연하고 확장이 쉬운 소프트웨어를 만들기 위한 입증된 5가지 객체지향 디자인 원리들

## SRP (단일책임의 원칙: Single Responsibility Principle)
> 작성된 클래스는 하나의 기능만 가지며 클래스가 제공하는 모든 서비스는 그 하나의 책임을 수행하는 데 집중되어 있어야 한다는 원칙
- 책임 영역이 확실해지기 때문에 한 책임의 변경에서 다른 책임으로의 연쇄작용에서 자유롭다.
- 책임 분배에 따른 코드의 가독성 향상, 유지보수 증가

### SRP 적용사례
- SRP 적용 전. instagramService내에서 여러가지 책임이 존재하는 상태. login, post, decode 등의 책임이 분리가 되어있지않아 재사용성이 감소하며 어떤 특성에 변화가 생겼을때 클래스에서 찾아서 수정해야 하기 때문에 유지보수가 까다롭다.
```swift
class instagramService {
    var posts: [String] = []
    
    func login(id: String, password: String) -> Data {
        // Call API
    }
    
    func postArticle(element: String) {
        posts.append(element)
        // Save Data on dataBase
    }
    
    func decodeUserProfile(data: Data) -> User {
        // Decoding Data
        return User(name: "brody", hobby: "game")
    }
    
    func writeComment(of: String) {
        // Save Data on dataBase
    }
}
```
- SRP 적용 후. 책임에 따라 분리가 되어있으며 이 덕분에 해당 기능을 다른 객체에서도 사용이 가능하게 되었다. 프로토콜마다 특정 책임이 분명하다는 것에 집중하자. 요청을 통해 다른 객체와의 협력을 하는 것이 주 목적이다.
```swift
protocol LoginHandlerProtocol {
    func login(id: String, pasword: String) -> Data
}

protocol ArticleHandlerProtocol {
    func postArticle(element: String)
    func writeComment(of: String)
}

protocol DecodingHandlerProtocol {
    func decode<T>(data: Data) -> T
}

class InstagramService {
    let loginHandler: LoginHandlerProtocol
    let articleHandler: ArticleHandlerProtocol
    let decodingHandler: DecodingHandlerProtocol
    
    init(loginHandler: LoginHandlerProtocol,
         articleHandler: ArticleHandlerProtocol,
         decodingHandler: DecodingHandlerProtocol) {
        self.loginHandler = loginHandler
        self.articleHandler = articleHandler
        self.decodingHandler = decodingHandler
    }
    
    func login() {
        // loginHandler에게 "요청"해서 책임을 전가. 반환을 통해 "응답"
        let loginData = loginHandler.login(id, password)
        // decodingHandler에게 "요청"해서 책임을 전가. 반환을 통해 "응답"
        let user: User = decodingHandler.decode(from: loginData)
    }
    
    func postArticle() {
        let article = "계산기어렵다"
        // articleHandler에게 "요청"해서 책임을 전가.
        articleHandler.postArticle(element: article)
    }
}
```
## OCP (개발폐쇄의 원칙: Open Close Principle)
> 소프트웨어의 구성요소(컴포넌트, 클래스, 모듈, 함수)는 확장에는 열려있고, 변경에는 닫혀있어야 한다.
- 확장에 열려있는 것은 쉽게 클래스의 기능을 확장할 수 있다는 것이고, 변경에 닫혀있다는 것은 기존에 구현되어 있는 것들을 바꾸지 않고 클래스를 확장할 수 있어야 한다는 뜻이다. 이러한 특성들은 **추상화**를 통해 이루어질 수 있다.
- OCP 적용 전. 만약 토스기업을 추가하게 된다면 기존의 Company클래스의 printData함수를 수정해야 한다. 
```swift
class Company {
    func printData() {
        let kakaoCompanies = [
            Kakao(name: "카카오", brandColor: "yellow"),
            Kakao(name: "카카오뱅크", brandColor: "yellow")
        ]
        
        kakaoCompanies.forEach { print($0.printDetails()) }
    }
}

class Kakao {
    let name: String
    let brandColor: String
        
    init(name: String, brandColor: String) {
        self.name = name
        self.brandColor = brandColor
    }
    
    func printDetails() -> String {
        return "안녕하세요 \(name)입니다. 저희 회사의 브랜드컬러는 \(brandColor)입니다."
    }
}
```
- 토스기업을 추가했을 때 다음의 코드처럼 카카오회사 코드와 유사한 토스회사 코드들을 추가해야 한다. 수많은 기업들이 추가되면 printData의 크기는 비례적으로 계속 커져 알아보기 힘든 코드가 될 수 있다. 이 때문에 추상화작업이 필요하고 이를 구현하기 위한 수단으로 프로토콜을 사용할 것이다.
```swift
class Company {
    func printData() {
        let kakaoCompanies = [
            Kakao(name: "카카오", brandColor: "yellow"),
            Kakao(name: "카카오뱅크", brandColor: "yellow")
        ]
        
        kakaoCompanies.forEach { print($0.printDetails()) }
        
        let tossCompanies = [
            Toss(name: "토스", brandColor: "blue"),
            Toss(name: "토스뱅크", brandColor: "blue")
        ]
        
        tossCompanies.forEach { print($0.printDetails()) }
    }
}
```
- OCP 적용 후. 추가되는 기업들이 Printable프로토콜을 따르고 있기만 한다면 해당 프로토콜타입으로 추상화 된 타입으로 배열을 만들 수 있게 된다. 그리고 이 프로토콜을 채택함으로써 printDetails이 프로토콜 타입에 속하게 되고 이는 Company의 printData내 중복 구현을 회피하게 만든다.
```swift
protocol Printable {
    func printDetails() -> String
}

class Company {
    func printData() {
        let companies: [Printable] = [
            Kakao(name: "카카오", brandColor: "yellow"),
            Kakao(name: "카카오뱅크", brandColor: "yellow"),
            Toss(name: "토스", brandColor: "blue"),
            Toss(name: "토스뱅크", brandColor: "blue")
        ]
        
        companies.forEach { print($0.printDetails()) }
    }
}
```
- 적당한 추상화 레벨을 선택하는 게 중요하다. 그래디 부치에 의하면 '추상화란 다른 모든 종류의 객체로부터 식별될 수 있는 객체의 본질적인 특징'이라고 정의하고 있다. 본질적인 부분을 끄집어내 프로토콜로 만드는 것. 그리고 이 프로토콜을 채택함으로써 변경은 닫히고 확장은 열려있게 하는 것이 OCP의 목적이다.

## LSP(리스코프 치환 원칙: The Liskov Substitution Principle)
> 상위 타입과 하위 타입 사이에서, 프로그램의 속성(정확성, 수행하는 업무 등)의 변경없이 상위 타입의 객체를 하위 타입으로 교체할 수 있어야 한다는 원칙

- LSP적용 전. 직사각형은 정사각형이라 할 수 없고 정사각형은 직사각형이라고 할 수 있다. 그렇기 때문에 직사각형이 정사각형보다 넓은 범주에 있다. 아래 코드는 정사각형 클래스가 직사각형의 클래스를 상속받은 경우이다.
```swift
class Rectangle {
    var width: Float = 0
    var length: Float = 0

    var area: Float {
        return width * length
    }
}

class Square: Rectangle {
    override var width: Float {
        didSet {
            length = width
        }
    }
}

func printArea(of rectangle: Rectangle) {
    rectangle.length = 5
    rectangle.width = 2
    print(rectangle.area)
}

let rectangle = Rectangle()
printArea(of: rectangle) // 10

// -------------------------------

let square = Square()
printArea(of: square) // 4
```
- 결과를 보면 넓이가 다른 것을 볼 수 있다. 이는 하위 클래스로는 상위 클래스의 넓이를 구할 수 없기에 LSP를 위반한 것이라고 볼 수 있다. 자식 클래스가 상위 클래스를 완전히 대체할 수 있어야 하는데 대체할 수 없는 상황이기 때문이다. 이는 프로토콜을 이용해서 문제를 해결할 수 있다.


```swift
protocol Polygon {
    var area: Float { get }
}

class Rectangle: Polygon {

    private let width: Float
    private let length: Float

    init(width: Float, length: Float) {
        self.width = width
        self.length = length
    }

    var area: Float {
        return width * length
    }
}

class Square: Polygon {

    private let side: Float

    init(side: Float) {
        self.side = side
    }

    var area: Float {
        return pow(side, 2)
    }
}

// Client Method

func printArea(of polygon: Polygon) {
    print(polygon.area)
}

// Usage

let rectangle = Rectangle(width: 2, length: 5)
printArea(of: rectangle) // 10

let square = Square(side: 2)
printArea(of: square) // 4
```
- 상속 구조가 아닌 프로토콜을 받음으로써 혼동될 여지를 줄였다. 
- 상속 구조를 사용하려면 서브 클래스가 슈퍼 클래스의 사전 조건과 같거나 더 약한 수준에서 사전 조건을 대체할 수 있어야 한다. 만약 서브 클래스에서 제약조건이 더 강하다면 슈퍼 클래스에서 실행되지 않기 때문이다.(위의 예시처럼)

## ISP(인터페이스 분리 원칙: The Interface Segregation Principle)
> 클라이언트는 그들이 사용하지 않는 인터페이스에 의존해서는 안된다.

- 불필요한 인터페이스 요소들을 포함시키면 안된다는 뜻. 불필요한 요소들이 포함되면 무거워지고, 이에 따라 원하는 정보를 얻을 수 없는 지경까지 이르기도 한다. 이는 클래스나 프로토콜 모두에게 영향을 줄 수 있다.

- ISP적용 전
```swift
protocol GestureProtocol {
    func didTap()
    func didDoubleTap()
    func didLongPress()
}
```
- 위 프로토콜을 채택해서 사용하는 컴포넌트의 경우에는 문제가 되지않는다. 하지만 didTap에 해당하는 부분만 사용하면 되고 나머지는 필요없는 경우에는 불필요한 메서드 준수가 생기게 된다. 그래서 위 프로토콜은 구체적인 인터페이스를 구현하길 원하는 ISP의 지향점과는 거리가 있다. 
- ISP 적용 후 
```swift
protocol TapProtocol {
    func didTap()
}

protocol DoubleTapProtocol {
    func didDoubleTap()
}

protocol LongPressProtocol {
    func didLongPress()
}

class SuperButton: TapProtocol, DoubleTapProtocol, LongPressProtocol {
    func didTap() {
        // send tap action
    }

    func didDoubleTap() {
        // send double tap action
    }

    func didLongPress() {
        // send long press action
    }
}

class PoorButton: TapProtocol {
    func didTap() {
        // send tap action
    }
}
```
- 원하는 프로토콜만 채택함으로써 좀 더 명확한 클래스 구현이 가능해졌다.
- 클래스의 경우에도 ISP를 적용할 수 있다.
```swift
class Video {
    var title: String = "My Video"
    var description: String = "This is a beautiful video"
    var author: String = "Marco Santarossa"
    var url: String = "https://marcosantadev.com/my_video"
    var duration: Int = 60
    var created: Date = Date()
    var update: Date = Date()
}

func play(video: Video) {
    // load the player UI
    // load the content at video.url
    // add video.title to the player UI title
    // update the player scrubber with video.duration
}
```
- 비디오를 실행시키는데 title, url, duration만 필요하다면 나머지 프로퍼티들까지 전달할 필요는 없다. 이를 프로토콜을 사용해서 필요한 정보만 가지고 play할 수 있게 만들어 줄 수 있다.

```swift
protocol Playable {
    var title: String { get }
    var url: String { get }
    var duration: Int { get }
}

class Video: Playable {
    var title: String = "My Video"
    var description: String = "This is a beautiful video"
    var author: String = "Marco Santarossa"
    var url: String = "https://marcosantadev.com/my_video"
    var duration: Int = 60
    var created: Date = Date()
    var update: Date = Date()
}


func play(video: Playable) {
    // load the player UI
    // load the content at video.url
    // add video.title to the player UI title
    // update the player scrubber with video.duration
}
```

## DIP(의존관계 역전 원칙: Dependency Inversion Principle)
> 고차원의 모듈들은 하위 모듈에 의존하면 안된다. 두 개 모두 추상 개념에 의존해야 한다. 추상 개념은 세부 사항에 의존하면 안된다. 세부 사항이 추상 개념에 의존해야 한다.

- DIP는 확장에는 유연하고 변경에는 닫혀있는 OCP와 유사하다. 

- DIP 적용 전. Handler는 고수준, FilesystemManager는 저수준이다. 고수준이 저수준에 의존하고 있기 때문에 고수준을 재사용하기가 까다로워진다. 이 역시 만능 프로토콜을 사용해서 프로토콜에 의존시켜 문제를 해결할 수 있다.
```swift
class Handler { 
    let fm = FilesystemManager()
 
    func handle(string: String) {
        fm.save(string: string)
    }
}
 
class FilesystemManager { 
    func save(string: String) {
        // Open a file
        // Save the string in this file
        // Close the file
    }
}
```

- DIP 적용 후. 고수준에서 저수준에 의존하지 않고 프로토콜을 의존함으로써 재사용화가 가능해졌다. 이제 Handler에 어떤 종류의 storageManger를 넣어도 Storage프로토콜만 따르고 있다면 대응할 수 있게 되었다. 
```swift
protocol Storage {
   func save(string: String)
}

class Handler {
    let storage: Storage
 
    init(storage: Storage) {
        self.storage = storage
    }
 
    func handle(string: String) {
        storage.save(string: string)
    }
}
 
class FilesystemManager: Storage {
    func save(string: String) {
        // Open a file in read-mode
        // Save the string in this file
        // Close the file
    }
}
 
class DatabaseManager: Storage {
    func save(string: String) {
        // Connect to the database
        // Execute the query to save the string in a table
        // Close the connection
    }
}
```

### 결론
- 프로토콜을 이용해 추상화를 하고, 이를 통해 재사용화를 가능하게 하는 것이 핵심이라고 생각한다. 그리고 그 프로토콜은 명확한 책임을 가지고 있어야 객체 간 협력할 때 혼란을 주지않고 소통할 수 있게 된다. 
- 간단한 예시들로 Solid원칙을 학습해봤는데 이를 습득하는데에는 많은 시간이 걸릴 것 같다. 코드를 작성하면서 이 원칙들을 최대한 지키도록 계속해서 노력해야겠다.


### Reference
- [참신러닝 - OOP의 SOLID원칙](https://leechamin.tistory.com/518)
- [넥스트리소프트 - 객체지향 개발 5대 원리: SOLID](https://www.nextree.co.kr/p6960/)
- [동동이 - Swift SOLID 원칙](https://dongminyoon.tistory.com/49)

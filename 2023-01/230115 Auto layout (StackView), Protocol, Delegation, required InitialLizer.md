230115 Auto layout (StackView), Protocol, Delegation, required InitialLizer
===
TIL (Today I Learned)
===
학습내용
---
- Auto layout 스택 뷰 학습하기
- 프로토콜 학습하기
- required initializer
- Delegation

## 고민한 점 / 해결 방법

## StackView
> 스택뷰는 복잡한 제약사항 추가없이 강력한 오토레이아웃을 제공한다.
- axis: 뷰의 방향. 가로 or 세로
- distribution: axis에 따른 뷰 레이아웃
- alignment: axis의 수직방향에 따른 뷰 레이아웃
- spacing: 인접한 뷰들 사이의 간격

### Distribution
> axis에 따른 뷰들의 사이즈를 어떻게 지정할지 정하는 속성
- fill: contents huggings의 priority에 따라서 화면을 꽉 채운다. 레이블 텍스트는 contents hugging을 나타낸다.
<img src="https://i.imgur.com/zyIVkTc.png" width="300"/>
- fill Equally: priority에 관계없이 동등한 크기로 화면을 꽉 채운다. 첫번째 레이블에 CR을 제일 높게 줘도 화면에 잘린다.
<img src="https://i.imgur.com/tKZtaca.png" width="300"/>
- fill proportionally: 고유한 컨텐츠 크기에 따라 비례하여 크기가 조정된다. 
<img src="https://i.imgur.com/21vAOdj.png" width="300"/>
- equal spacing: fill인 상태에서 view사이에 동일한 spacing을 준다. 
<img src="https://i.imgur.com/9ImWUFO.png" width="300"/>
- equalCentring: 고유 컨텐츠 사이즈대로 배치한 뒤 각 뷰의 센터를 기준선으로 잡아 다른 뷰와의 사이를 일치시킨다.

### Alignment(가로 스택뷰일 때)
> axis의 반대 방향에 따른 뷰들의 위치를 어떻게 지정할지 정하는 속성
- fill: 가로방향의 스택뷰 상태에서 alignment를 fill로 준다면 세로로 다 채워진다.
<img src="https://i.imgur.com/ndUUdjE.png" width="300"/>
- top: 가로방향의 스택뷰 상태에서 alignment를 top을 준다면 top에 딱 맞게 정렬이 된다.(bottom은 반대)
<img src="https://i.imgur.com/DzQvYOD.png" width="300"/>
- center: 가로방향의 스택뷰 상태에서 alignment를 center를 준다면 가운데 정렬이 된다.
<img src="https://i.imgur.com/DzQvYOD.png" width="300"/>
- First BaseLine: 여러 객체들이 존재할 때 첫번째 줄(뷰)를 기준으로 정렬이 되는 것.
<img src="https://i.imgur.com/KwBJAdb.png" width="300"/>
- last BaseLine: 여러 객체들이 존재할 때 마지막 줄(뷰)를 기준으로 정렬이 되는 것.
<img src="https://i.imgur.com/kpve8bk.png" width="300"/>

### Alignment(세로 스택뷰일 때)
> axis의 반대 방향에 따른 뷰들의 위치를 어떻게 지정할지 정하는 속성
- leading: 맨 앞쪽으로 뷰들이 붙는다. (trailing은 반대)
<img src="https://i.imgur.com/TApPgYh.png" width="300" height="200"/>
- center: 가운데 정렬이 된다.
<img src="https://i.imgur.com/nXVvhHq.png" width="300" height="200"/>

### StackView 예제
<img src="https://i.imgur.com/C8X2UIz.png" width="200"/>

- imageView의 vertical content hugging을 낮춰서 이미지가 height를 채우게 만들고 alignment는 fill로 줘서 화면을 꽉 채운다.
<img src="https://i.imgur.com/IYN6vO4.png" width="200"/>

- 오토레이아웃은 내부에서 외부로 제약을 걸어주는 것이 쉬운 방법이다. 
- 레이블과 텍스트뷰를 가로 스택뷰로 놓고 first baseline으로 alignment를 준다. 이 스택뷰를 스택뷰1라고 하자.
- 스택뷰1처럼 아래 Middle과 Last도 가로스택뷰로 묶은 뒤 이들을 세로 스택뷰로 묶는다. 이 스택뷰를 스택뷰2라고 하자.
- 스택뷰2와 이미지를 가로 스택뷰로 묶는다. 이를 스택뷰 3라고 하자.
- 맨 아래 버튼끼리 가로 스택뷰를 묶는다. 이를 스택뷰 4라고하자.
- 스택뷰3, 텍스트뷰, 스택뷰4를 세로 스택뷰로 묶은 뒤 네 방향으로 10 constant씩 제약을 걸어준다. 그리고 alignment를 fill로 줌으로써 컨텐츠가 화면에 꽉차게 만든다.
- 스택뷰들 사이에 spacing을 8로 준다.

### 그런데 사진을 보면 텍스트필드의 width가 동일하지 않다.
세 개의 텍스트필드의 가로를 전부 맞추기 위해 첫 번째 텍스트필드와 나머지 텍스트 필드와의 width관계를 동일하게 지정해줬다.<br>
<img src="https://i.imgur.com/UkZBrFh.png" width="200"/><br>

> First TextField.width = 1.00845 * Second TextField.Width 
- ctrl을 통해 관계를 지으면 첫 번째 요소가 현재 더 크기 때문에 multiplier가 자동으로 붙은 걸 볼 수 있다. 두 번째 요소와 width가 동일해야 하기 때문에 width를 1로바꿔주면 해결이 된다. 세 번째 요소도 마찬가지다.<br>
<img src="https://i.imgur.com/JzXpNUL.png" width="200"/><br>

## 프로토콜
> 특정 기능 수행에 필수적인 요소(프로퍼티, 메서드)를 정의한 청사진

### 프로토콜 문법
- 클래스, 구조체, 열거형에 채택이 가능하다.
- 여러 개의 프로토콜을 상속할 수 있다.
- 슈퍼클래스와 프로토콜 동시에 가진다면 슈퍼클래스 뒤에 프로토콜을 채택한다.
```swift
class SomeClass: SomeSuperclass, FirstProtocol, AnotherProtocol {
    // class definition goes here
}
```

### 프로퍼티 요구사항
- 인스턴스 프로퍼티나 타입 프로퍼티를 요구할 수 있다.
- 프로퍼티가 저장속성인지 계산속성인지 정의하지 않는다. 오직 프로퍼티 이름, 타입, gettable한지 settable한지만 정의를 한다.
- 만약 프로토콜이 gettable, settable한 프로퍼티를 요구한다면 상수저장속성이나 읽기전용 계산속성은 이를 충족시킬 수 없다.
- 만약 프로토콜이 오직 gettable한 프로퍼티를 요구한다면 종류에 관계없이 어떤 프로퍼티든 만족시킬 수 있다. 이는 구현코드에서 settable하게도 사용할 수 있다.
- 프로퍼티 요구사항은 {get set}과 같이 계산속성처럼 사용되기 때문에 항상 var로 시작해야 한다.
- 타입 프로퍼티 요구사항은 비록 구현부에서 class나 static을 사용할지라도 요구사항에서는 static keyword를 붙여야 한다. 

#### 예시
```swift
protocol FullyNamed {
    var fullName: String { get }
}
class Starship: FullyNamed {
    var prefix: String?
    var name: String
    init(name: String, prefix: String? = nil) {
        self.name = name
        self.prefix = prefix
    }
    var fullName: String {
        return (prefix != nil ? prefix! + " ": "") + name
    }
}
var ncc1701 = Starship(name: "Enterprise", prefix: "USS")
// ncc1701.fullName is "USS Enterprise"
```
- 프로토콜 정의에서는 fullName이 gettable만 가능하도록 설정해두었고 채택해서 사용하는 Starship부분에서는 이를 계산속성으로 정의했다. 여기서 fullName은 계산속성이기 때문에 prefix속성에 참조할 수 있다.

### 메서드 요구사항
- 프로퍼티 메서드 요구사항은 기존의 메서드 정의에서 중괄호와 메서드 본문이 없는 형태이다.
```swift
protocol RandomNumberGenerator {
    func random() -> Double
}
```
- RandomNumberGenerator프로토콜을 채택해서 구현하는 클래스
```swift
class LinearCongruentialGenerator: RandomNumberGenerator {
    var lastRandom = 42.0
    let m = 139968.0
    let a = 3877.0
    let c = 29573.0
    func random() -> Double {
        lastRandom = ((lastRandom * a + c)
            .truncatingRemainder(dividingBy:m))
        return lastRandom / m
    }
}
let generator = LinearCongruentialGenerator()
print("Here's a random number: \(generator.random())")
```

### 변경 가능한 메서드 요구사항
- 값타입에서 메서드를 변경시키기 위해 mutating키워드를 붙여주는데 프로토콜에서도 명시해서 사용할 수 있다.
> 만약 프로토콜에서 mutating을 사용하고 클래스에서 이를 채택해서 사용할때는 mutating키워드를 사용할 필요가 없다.

### 초기자 요구사항
- 프로토콜을 채택한 타입에 의해 구현되어질 구체적인 초기자를 요구할 수 있다.
```swift
protocol SomeProtocol {
    init(someParameter: Int)
}

class SomeClass: SomeProtocol {
    required init(someParameter: Int) {
        // initializer implementation goes here
    }
}
```
- 프로토콜에서 특정 초기자가 필요하다고 명시했기 때문에 구현에서 해당 이니셜라이저에 required 키워드를 붙여줘야 한다. -> 서브클래싱때문에 필수라는 키워드가 존재해야 완전한 인스턴스를 보장받기 때문 
> 클래스에 final modifier가 붙어있는 경우에는 required를 붙이지 않아도 된다. final로 되면 서브클래싱되지 않기 때문이다. 
```swift
protocol SomeProtocol {
    init()
}

class SomeSuperClass {
    init() {
        // initializer implementation goes here
    }
}

class SomeSubClass: SomeSuperClass, SomeProtocol {
    // "required" from SomeProtocol conformance; "override" from SomeSuperClass
    required override init() {
        // initializer implementation goes here
    }
}
```
- SuperClasee를 상속받으면서 init을 재정의하기 때문에 override키워드를 붙이고, SomeProtocol을 채택함으로써 반드시 프로토콜을 구현해야하기 때문에 required키워드를 붙였다.

### 타입으로써의 프로토콜
> 프로토콜은 타입으로 사용되기 때문에 타입 사용이 허용되는 곳에 프로토콜을 사용할 수 있다
- 함수, 메소드, 이니셜라이저의 파라미터 타입 혹은 리턴타입
- 상수, 변수, 프로퍼티의 타입
- 배열, 딕셔너리 등의 타입

```swift
class Dice {
    let sides: Int
    let generator: RandomNumberGenerator
    init(sides: Int, generator: RandomNumberGenerator) {
        self.sides = sides
        self.generator = generator
    }
    func roll() -> Int {
        return Int(generator.random() * Double(sides)) + 1
    }
}
var d6 = Dice(sides: 6, generator: LinearCongruentialGenerator())
for _ in 1...5 {
    print("Random dice roll is \(d6.roll())")
}
// Random dice roll is 3
// Random dice roll is 5
// Random dice roll is 4
// Random dice roll is 5
// Random dice roll is 4
```
- generator는 프로토콜타입을 따른다.

### 델리게이션
> 클래스 혹은 구조체 인스턴스에 특정 행위에 대한 책임을 넘길 수 있게 해주는 디자인 패턴
```swift
protocol DiceGame {
    var dice: Dice { get }
    func play()
}
protocol DiceGameDelegate: AnyObject {
    func gameDidStart(_ game: DiceGame)
    func game(_ game: DiceGame, didStartNewTurnWithDiceRoll diceRoll: Int)
    func gameDidEnd(_ game: DiceGame)
}
```
- DiceGameDelegate프로토콜이 AnyObject프로토콜을 따름으로써 클래스만 이 프로토콜을 채택할 수 있도록 만든다.

```swift
class SnakesAndLadders: DiceGame {
    let finalSquare = 25
    let dice = Dice(sides: 6, generator: LinearCongruentialGenerator())
    var square = 0
    var board: [Int]
    init() {
        board = Array(repeating: 0, count: finalSquare + 1)
        board[03] = +08; board[06] = +11; board[09] = +09; board[10] = +02
        board[14] = -10; board[19] = -11; board[22] = -02; board[24] = -08
    }
    weak var delegate: DiceGameDelegate?
    func play() {
        square = 0
        delegate?.gameDidStart(self)
        gameLoop: while square != finalSquare {
            let diceRoll = dice.roll()
            delegate?.game(self, didStartNewTurnWithDiceRoll: diceRoll)
            switch square + diceRoll {
            case finalSquare:
                break gameLoop
            case let newSquare where newSquare > finalSquare:
                continue gameLoop
            default:
                square += diceRoll
                square += board[square]
            }
        }
        delegate?.gameDidEnd(self)
    }
}
```
- SnakesAndLadders는 DiceGame을 따르고 DiceGameDelegaate를 따르는 델리게이트를 갖는다. delegate가 게임을 실행시키는데 반드시 필요한 것은 아니기 때문에 옵셔널로 되어있다. 그리고 델리게이트는 다른 객체를 참조하고 있기 때문에 weak로 선언함으로써 강한 참조 순환을 예방한다.
- 아래 코드는 DiceGameDelegate를 상속받고있는 DiceGameTracker이다.
```swift
class DiceGameTracker: DiceGameDelegate {
    var numberOfTurns = 0
    func gameDidStart(_ game: DiceGame) {
        numberOfTurns = 0
        if game is SnakesAndLadders {
            print("Started a new game of Snakes and Ladders")
        }
        print("The game is using a \(game.dice.sides)-sided dice")
    }
    func game(_ game: DiceGame, didStartNewTurnWithDiceRoll diceRoll: Int) {
        numberOfTurns += 1
        print("Rolled a \(diceRoll)")
    }
    func gameDidEnd(_ game: DiceGame) {
        print("The game lasted for \(numberOfTurns) turns")
    }
}
```
- 실행부
```swift
let tracker = DiceGameTracker()
let game = SnakesAndLadders()
game.delegate = tracker
game.play()
```
- game에서 traking하는 부분은 tracker로 설정함으로써 tracker의 메서드를 game에서 이용할 수 있게 만들었다. 이는 game에서 delegate변수의 타입이 tracker와 일치하기 때문에 가능하다.

## required initializer
> 필수 생성자로, 슈퍼 클래스에서 정의할 경우 서브 클래스가 슈퍼 클래스의 생성자를 상속받지 않는 한 서브클래스에서 반드시 구현해주어야 한다.
- 서브 클래스에서 직접 지정 생성자를 정의할 때만 재정의해야 하고 만약 지정 생성자 정의가 필요없다면 자동으로 상속되기 때문에 정의할 필요가 없다.

## 델리게이션
> 한 객체가 다른 객체의 역할을 대신하거나 협력하는 강력한 패턴이다.
- 델리게이션 객체는 다른 객체에 대해 참조를 가지고 있고 적절한 시간에 메시지를 보내서 변화에 반응할 수 있도록 한다.
- 델리게이션의 주 가치는 하나의 중앙 객체 안에서 여러개의 객체에 접근해서 customizing할 수 있다는 것이다.

### Data source
- Data source는 델리게이트와 거의 동일하다. 차이점은 유저 인터페이스의 컨트롤 대신에 data자체이다. 일반적으로 델리게이션 객체는 테이블뷰와 같이 뷰 객체가 data Source에 참조를 갖고 보여질 데이터를 요청한다. 
- 델리게이트 뷰에 제공할 모델 객체의 메모리를 관리할 책임이 있다.

## 객체 행동을 커스터이즈 하기 위해 델리게이트 사용
> 코코아 객체들(앱 이벤틀를 알려주는)과 상호작용하는 델리게이트를 사용할 수 있다.

### 델리게이트 프로토콜 채택
- 코코아 API들은 델리게이트 메서드 프로토콜을 포함한다. 위임자는 이벤트를 감지하고 사용자 코드에 있는 델리게이트 메서드를 호출한다.
```swift
class MyDelegate: NSObject, NSWindowDelegate {
    func window(_ window: NSWindow, willUseFullScreenContentSize proposedSize: NSSize) -> NSSize {
        return proposedSize
    }
}
```

### 델리게이트 존재를 체크해라
- 코코아 델리게이션 패턴은 위임자를 초기화하는 것이 필수가 아니다. 델리게이트 메서드를 호출하기전에 델리게이트가 nil이 아닌지 확인해야 한다. 이를 옵셔널 체이닝으로 확인한다.
```swift
let myWindow = NSWindow(
    contentRect: NSRect(x: 0, y: 0, width: 5120, height: 2880),
    styleMask: .fullScreen,
    backing: .buffered,
    defer: false
)

myWindow.delegate = MyDelegate() // myWindoe의 위임자를 MyDelegate()로 설정. 즉 myWindow에 resize이벤트가 감지되면 MyDelegate메서드가 실행
if let fullScreenSize = myWindow.delegate?.window(myWindow, willUseFullScreenContentSize: mySize) {
    print(NSStringFromSize(fullScreenSize))
}
```




###### tags: `Auto layout`, `stackView`, `protocol`, `delegation` `required initializer`


### Reference
- [야곰닷넷 - Auto Layout](https://yagom.net/courses/autolayout/)
- [Apple Archive - Auto Layout](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/AutolayoutPG/ViewswithIntrinsicContentSize.html#//apple_ref/doc/uid/TP40010853-CH13-SW1)
- [Swift Language Guide - Protocol](https://docs.swift.org/swift-book/LanguageGuide/Protocols.html)
- [Apple archive - Delegation](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/Delegation.html)
- [Apple Developer Documentation - Delegate](https://developer.apple.com/documentation/swift/using-delegates-to-customize-object-behavior)

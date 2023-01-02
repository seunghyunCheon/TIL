230102 MVC 디자인패턴, initializer, Nested types
===
TIL (Today I Learned)
===
학습내용
---
- MVC 디자인패턴 학습하기
- initializer 학습하기
- Nested Types 학습하기

## 고민한 점 / 해결 방법

### [아키텍처, 디자인패턴]
> 아키텍처는 프로그램의 뼈대(품질이나 성능)같은 높은 수준의 레벨에서의 정형화된 설계이고, 디자인패턴은 코드 수준과 같은 낮은 수준의 레벨에 대한 설계이다.

### [MVC 디자인패턴]
> 역할에 따라 객체를 Model, View, Controller 중 하나에 assign하는 것. 그리고 이들 간의 상호작용하는 방법을 정의한 것
> 
- Model: 데이터를 캡슐화하고 비즈니스 로직을 다루는 레이어
- View: 사용자에게 정보를 보여주는 레이어
- Controller: Model과 View사이의 통로역할을 하는 레이어

### [initializer]
#### 기본 이니셜 라이저
- 항상 같은 초기 값을 지닌다면 기본값 프로퍼티를 이용하는 것이 좋다.
- 만약 모든 프로퍼티의 초기값이 설정돼있꼬 하나의 초기자도 정의하지 않았다면 모든 프로퍼티를 기본 값으로 초기화하는 기본 초기자를 제공한다.
    ```swift
    class ShoppingListItem {
        var name: String?
        var quantity = 1
        var purchased = false
    }
    var item = ShoppingListItem()
    ```
#### 값 타입에서의 이니셜라이저 위임
- 값 타입과 참조 타입의 이니셜라이저 위임은 다르다. 값 타입은 상속을 지원하지 않아 이니셜라이저를 자기 자신의 다른 이니셜라이저를 통해서만 사용할 수 있다. 반면에 클래스의 경우 superclass의 이니셜라이저를 subclass에서 호출이 가능하다.

#### 클래스에서의 이니셜라이저 위임
- 지정 생성자 반드시 직계 superclass의 지정 초기자를 호출해야한다.
- 편의 생성자는 반드시 같은 클래스의 다른 초기자를 호출해야 한다.
- 편의 생성자는 궁극적으로 지정 생성자를 호출해야 한다.<br>
<img src="https://i.imgur.com/Kt899Ku.png" width="500"/>

#### 2단계 초기화
- 클래스 초기화는 2단계로 진행된다.
- 첫번째 단계는 각 저장된 프로퍼티가 초기값으로 초기화되고, 두번째 단계는 새로운 인스턴스의 사용이 준비되었다고 알려주기 전에 커스터마이징을 할 수 있는 단계이다.
- Swift는 4단계 safe-check를 통해 안전하게 끝나는 것을 보장한다.
> 1. 지정 생성자는 superClass의 생성자에게 위임하기 전에 소유하고 있는 프로퍼티 모두를 초기화 해야 한다.
> 2. 지정 생성자는 상속된 값을 할당하기 전에 반드시 superClass의 초기화로 위임을 넘겨야 한다. 그렇지 않으면 상속된 값이 superClass 초기화에 의해 덮어 쓰여진다.
> 3. 편의 생성자에서는 값을 할당하기 전에 다른 생성자로 위임을 넘겨야 한다. 만약 그렇지 않으면 편의 생성자에서 할당된 값이 지정 생성자에서 덮어쓰여지게 된다.
> 4. 이니셜라이저는 초기화의 1단계가 끝나기 전에는 self의 값을 참조하거나 어떠한 인스턴스, 메서드를 읽을 수 없다.


#### 4가지 safe-check를 기반한 two-phase initialization
#### Phase1
- 지정, 편의 생성자를 통해 호출된다
- 클래스의 인스턴스 메모리가 할당된다. 아직 초기화가 된 건 아님
- 클래스의 모든 저장속성이 값을 가지는지 확인한다. 이때 저장속성들이 초기화가 된다.
- superclass로 위임해서 같은작업을 한다.
- Top of the chain에 도달하면 인스턴스 메모리, 저장 속성들이 완전히 초기화 된 상태.

#### Phase2
- Top of the chain에서 내려오면서 커스터마이징옵션을 가진다. 이 단계에서는 self, 속성, 메서드에 접근이 가능하다.
- 마지막으로 편의 생성자가 커스터마이징옵션을 가진다.

#### 실패 가능한 초기화(init?)
- 초기화에 실패하는 부분에서 nil을 반환하는 코드를 작성해 초기화가 실패했다는 것을 나타낼 수 있다.
> 엄밀이 말하면 초기화 init은 값을 반환하지 않는다. 그래서 비록 nil을 반환하는 return nil코드에는 사용하지만 init이 성공하는 경우에는 값을 리턴하지 않는다.
```swift
struct Animal {
    let species: String
    init?(species: String) {
        if species.isEmpty { return nil }
        self.species = species
    }
}

let anonymousCreature = Animal(species: "")
// anonymousCreature is of type Animal?, not Animal

if anonymousCreature == nil {
    print("The anonymous creature could not be initialized")
}
// Prints "The anonymous creature could not be initialized"
```

#### 필수 생성자
> 모든 subclass에서 필수적으로 구현해야 할 initializer가 존재할 때 정의
- required initializer는 override 키워드를 사용할 필요가 없다.

### [Nested Types]
> 특정 문맥에서 조금 더 복잡한 타입을 위해 지원되는 타입
> 보통 열거형이 특정 구조체나 클래스 안에서 사용된다.
```swift
class User {
    let rank: Rank
    let job: Job
    
    enum Job {
        case swordsman
        case wizard
        case archer
        
        var jobName: String {
            switch self {
            case .swordsman:
                return "검사"
            case .wizard:
                return "마법사"
            case .archer:
                return "궁수"
            }
        }
    }
    
    enum Rank {
        case challenger
        case grandMaster
        case diamond
        case platinum
        case gold
        
        var userRank: String {
            switch self {
            case .challenger:
                return "첼린저"
            case .grandMaster:
                return "그랜드마스터"
            case .diamond:
                return "다이아몬드"
            case .platinum:
                return "플래티넘"
            case .gold:
                return "골드"
            }
        }
    }
    
    init(rank: Rank, job: Job) {
        self.rank = rank
        self.job = job
    }
    
    var description: String {
        return "직업은\(job) 랭크는 \(rank)"
    }
}

let user1 = User(rank: .diamond, job: .archer)
print(user1.description)
```


### Reference
- [Swift Lanugate Guide - initializer](https://docs.swift.org/swift-book/LanguageGuide/Initialization.html)
- [Swift Lanugate Guide - nested types](https://meetup.nhncloud.com/posts/315)
- [Apple Document Archive - MVC](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/MVC.html)
- [Apple Document Archive - MVC](https://developer.apple.com/library/archive/documentation/General/Conceptual/CocoaEncyclopedia/Model-View-Controller/Model-View-Controller.html#//apple_ref/doc/uid/TP40010810-CH14)
###### tags: `MVC`, `디자인패턴`, `아키텍처`, `initializer`, `delegate`, `nested Types`, `required initializer` 

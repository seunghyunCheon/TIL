# 230505 Replace Enum with Polymorphism, 스위프트로 다시보는 객체지향 프로그래밍: 피해야할 코딩 습관(SOL)

학습내용
- Replace Enum with Polymorphism
- 스위프트로 다시보는 객체지향 프로그래밍: 피해야할 코딩 습관

## Replace Enum with Polymorphism

스위프트에서 Enum은 강력하고 좋은 기능이다. 하지만 항상 좋은기능으로 사용될지에 대해서는 고민을 해봐야 한다.
예를 들어 다음의 코드를 살펴보자.

```swift
enum Animal {
    enum Size {
        case small
        case medium
        case large
    }
    
    case dog
    case cat
    case cow
    
    var noise: String {
        switch self {
            case .dog
                return "woof"
            case .cat:
                return "meow"
            case .cow:
                return "moo"
        }
    }
    
    var size: Size {
        switch self
        case .dog:
            return .meduim
        case .cat:
            return .small
        case .cow:
            return .large
    }
}
```

평소에 `enum`을 위 코드처럼 작성했을 것이다. 
만약 여우를 추가하면 어떻게 될까? case에 여우를 추가하면 컴파일러는 case가 완전하지않다고 알려줄 것이기에 enum을 사용하고 있는 곳을 전부 수정해야 한다.

그리고 중복코드도 상당히 많아보이며 추후 여러 타입과 혼합해서 사용한다면 그 많은 코드들을 수정해야하니 유지보수에 별로 좋지않아보인다.

### 개선

프로토콜을 사용해서 이를 개선시킬 수 있다.

```swift
enum AnimalSize {
    case small
    case medium
    case large
}

protocol Animal {
    var noise: String { get }
    var size: AnimalSize { get }
}


struct Dog: Animal {
    var noise = "woof"
    var size = AnimalSize.meduim
}

struct Cat: Animal {
    var noise = "meow"
    var size = AnimalSize.small
}

struct Cow: Animal {
    var noise = "moo"
    var size = AnimalSize.large
}
```

먼저 동물의 특징을 추려낸 뒤 프로토콜로 만들고 구체타입에 프로토콜을 채택한다. 
여기서 중요한 점은 `enum`의 case를 추가하더라도 기존의 `Animal`코드를 변경시키지 않아도 된다는 점이다. 단순히 타입만 추가해서 프로토콜을 채택하기만 하면 된다. 이게 `OCP`에 해당하는 확장에는 열려있고 변경에는 닫혀있는 코드인 것이다. 
그리고 중복코드도 많이 줄였다!

```swift
struct Fox: Animal {
    var noise = "wa-pa-pa-pa-pa-pa-pow"
    var size = AnimalSize.medium
}
```

### 언제 Enum을 사용해야 할까?
`enum`은 적절히 사용한다면 분명히 좋은 기능이다. 이는 case가 추가되지 않는 유연하지않은 상황에 사용하는 것이 좋다. 

위 예시로는 `AnimalSize`가 고정되어있기에 `enum`을 사용하기 좋은 위치인 것이다.


## 스위프트로 다시보는 객체지향 프로그래밍: 피해야할 코딩 습관

SOLID를 몇 가지 예시만으로 완전하게 이해하는 것은 불가능하다. 다만 이런 예시들을 참고해서 내 프로젝트에 어떻게 적용하면 좋을지 지속적으로 사고하려는 자세를 가져야 한다. 평소에 이를 조금씩이라도 인지하고 개선해나가려고 한다면 OOP식으로 코딩하는 것에 한발자국 더 다가갈 것라고 믿는다. 

이 블로그의 저자는 **잘못된 코드가 어떤건지는 알고 피해가자**라는 컨셉으로 OOP를 설명하고 있다. 그리고 ID가 빠진 SOL만 다루고 있다. 블로그 저자의 내용에 더해 개인적인 생각을 담을 것이니 잘못된 내용이 있을 수도 있다.

### Single Responsibility Principle

단일책임 원칙은 모든 클래스는 하나의 책임만 가져야 한다는 원칙이다. Swift에서 가장 대표적인 위반 사례는 `cocoa-MVC`디자인 패턴 내에서 발생할 수 있는 `Massvie View Controller`이다. `ViewController`는 `View`레이어에 해당하는 객체들도 갖고있기 때문에 라이프사이클 뿐만 아니라 `Model`에서 전달받은 데이터를 토대로 `View`를 업데이트하는 역할을 갖고 있다.
그리고 `Model`의 데이터가 변경되었을 때 이 데이터를 가공해서 `View`에 전달해야 한다면 가공하는 역할을 하는 공간도 `ViewController`가 갖기 떄문에 코드양이 더 커질 것이다. 

프로젝트 규모가 작다면 `cocoa-MVC`가 좋은 패턴일 것이다. 그리고 `ViewController`내부에서 데이터를 가공하는 로직을 갖지않고 어떤 객체에게 그 역할을 위임한다면 `cocoa-MVC`는 큰 프로젝트에서도 좋을 것이라고 생각한다. 하지만 그렇지않은 경우에는 `ViewController`의 역할이 너무 많아져서 유지보수에 좋지 않은 코드가 될 것이다. 

이를 개선해서 나온 것이 `MVVM`이다. 화면에 보여질 뷰를 위한 모델(View Model)을 만들고 이곳에서 모델의 데이터를 가공하고 뷰에 연결시켜준다. 이 방법이 `cocoa-MVC`에서 SRP를 적용시킨 디자인패턴이라고 생각한다.

저자는 SRP를 지키기 위한 노력으로 **10-200룰**을 지키려고 노력하기를 권장한다. 함수는 10줄 이내, 클래스는 200줄 이내로 만드는 룰이다. 

'함수는 10줄 이내로' 규칙을 지키려면 `함수는 하나의 작업만 해야 한다`라는 대원칙을 지킬 수 밖에 없고 `추상화 레벨`에 대해 고민하게 된다. 
추상화 레벨이란 얼만큼의 정보를 해당 함수에서 노출할 지를 정하는 것이다. 

티스토리 글쓰기를 예시로 들면 `회원가입한다`, `글쓰기버튼을누른다`, `글을작성한다`, `포스팅 버튼을 누른다`의 과정으로 글을 쓰게 된다. 
하지만 이는 고차원의 추상화레벨이다. `회원가입한다`는 `이름을입력한다`, `아이디를 입력한다`, `비밀번호를 입력한다` 등의 구체적인 동작이 존재하기 때문이다. 

함수를 10줄 이내로 작성하려고 노력하다보면 SRP를 지키는 함수가 만들어지고 이는 추상화레벨을 적절하게 고려한 코드를 작성할 가능성이 높아진다. 

`클래스는 200줄` 룰을 지키려면 프로퍼티나 함수, 메서드를 그냥 손이 가는 곳 아무데나 만들어서 쓸 수 없다. 꼭 얘가 있어야 하는 이유를 찾아야 한다. 해당 클래스에 있을 이유가 없으면 SRP 위반이다. 가장 쉽게 판단할 수 있는 방법 하나는 함수나 메서드 내부에서 self의 프로퍼티나 메서드를 얼마나 쓰고 있는지 보는 것이다. `self`를 별로 사용하고 있지않는다면 클래스와 연관성이 떨어지는 것이니 내쫓을 후보가 된다. 

이 부분을 읽고 한 가지 생각이 들었다.
> 클래스에서는 이와 연관된 프로퍼티와 메서드만 구성해야 하는데 메서드가 순수함수로 만든다면 입력값과 출력값이 똑같기 때문에 `self`의 참조가 없어지게끔 만들어지니 내쫓을 후보가 되는 것인가?

그래서 생각한 결론은 클래스 내부의 함수는 클래스 내부의 상태 혹은 메서드와 연관이 되어있는 것이 좋으니 꼭 순수함수로 만들 필요는 없다는 것이다. 

만약 우유를 먹었을 때 키가 1cm증가하는 함수가 있다고 가정해보자. 이때 함수는 순수함수로 구성이 된다.

```swift
struct Human {
    let name: String
    var height: Int
    
    init(name: String, height: Int) {
        self.name =  name
        self.height = height
    }
    
    func drinkMilk() -> Int {
        return self.height + 1
    }
}

let minsu = Human(name: "Minsu", height: 180)
let increasedHeight = minsu.drinkMilk()
minsu.height = increasedHeight
```

증가된 키를 반환받고 민수의 키에 다시 할당해주는 과정이 어색해보인다. 

```swift
func drinkMilk() { 
    self.height += 1
}
```

해당 함수가 더 직관적이고 불필요한 코드를 줄인다. 순수함수는 여러종류의 입력값이 있을때 동일한 결과값을 출력값으로 주길 원할 때 사용하는 것이 좋아보인다. 

예를 들어 은행에 등록된 고객이 있고 은행원이 등록된 고객의 나이에 따라 적용되는 대출이자 퍼센트를 조회하고 싶다면 순수함수를 사용하는 것이 좋을 수 있다. 대출이자퍼센트를 조회할 때 상태를 직접 변경하지 않기 때문이다. 

```swift
struct Customer {
    let name: String
    var age: Int
    
    init(name: String, age: Int) {
        self.name =  name
        self.age = age
    }
    
    func loanPercentage() -> Int {
        switch self.age {
        case 0..<20:
            return 6
        case 20..<40:
            return 4
        case 40...:
            return 2
        }
    }
}
```

결론은 클래스에서 함수는 꼭 순수함수일 필요는 없다는 것이다. 순수함수와 사이드이펙트를 주는 함수를 적절히 사용하는 것이 SRP를 준수하는 방법이라고 생각한다. 

### Open-Closed Principle
OCP는 위 `Replace Enum with Polymorphism`섹션에서 다루고 있다. 
핵심은 확장에는 열려있고 변경에는 닫혀있어야 한다는 것이고 이를 가능하게 만드는 것이 프로토콜이다. `enum`으로 구성된 코드를 프로토콜을 채택해서 사용하는 것으로 변경한다면 case가 추가되는 것에 유연하게 대처할 수 있게 된다.

### Liskov Subtitution Principle
리스코프 치환 원칙은 상위 타입(수퍼클래스)을 하위 타입(서브클래스)의 인스턴스로 바꿔도 프로그램의 동작을 해치지 않아야 한다는 원칙이다. 

**자식이 부모의 동작을 제한하지 않는다**가 핵심이다. 만약 정사각형이 직사각형 클래스를 상속받는다면 정사각형은 반드시 정사각형이 되기위해 width나 height를 동일하게 만들어야 하기 때문에 너비와 높이를 자유롭게 바꿀 수 있는 직사각형 부모 클래스의 동작을 제한해야만 한다.
반대로 직사각형이 정사각형을 상속받는다면 정사각형에는 width나 height를 변경하는 동작이 없기 때문에 부모의 동작을 제한하지 않는다.

LSP를 절대 어기지 않고 프로그래밍 하는 것을 어렵지만 너무 많은 곳에서 어긴다면 문제가 생긴다. 
상위 클래스를 기준으로 코딩할 수 없어지고, 그렇게되면 상속 자체가 의미가 없어지고 OCP조차 지킬 수 없게된다. 예를 들어 UITableView는 UITableViewCell을 기준으로 만들어져 있기 때문에 우리가 만든 커스텀 셀이 LSP를 지키지 않는다면 테이블뷰도 제대로 동작하지 않을 것이다. 

그래서 상속을 만들때는 LSP위반인지 아닌지를 숙지하고 사용하는 것이 좋다.

### 참고문서
- [Refactoring: Replace Enum with Polymorphism](https://medium.com/swift-fox/refactoring-replace-enum-with-polymorphism-c4803baeba07)
- [
스위프트로 다시보는 객체지향 프로그래밍: 피해야할 코딩 습관](https://soojin.ro/blog/solid-principles-in-swift)

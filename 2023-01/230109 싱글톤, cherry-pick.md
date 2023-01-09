230109 싱글톤, cherry-pick
===
TIL (Today I Learned)
===
학습내용
---
- 싱글톤 학습하기
- Git 특정 커밋만 적용시키기(cherry-pick)

## 고민한 점 / 해결 방법
## 싱글톤
> 하나의 전역 인스턴스만을 필요로할 때 사용

#### 사용방법
```swift
class BookStore {
    var books = ["1984", "인간실격", "참을 수 없는 존재의 가벼움"]
    
    static let shared = BookStore()
    private init() {}
    ...
    print(BookStore.shared)
```

#### 싱글톤은 구조체로? 클래스로?
```swift
import Foundation

struct StructSingleton {
    static let shared = StructSingleton()
    var value = 1
    private init() {
    }
}

var structSingleton1 = StructSingleton.shared
structSingleton1.value = 2
var structSingleton2 = StructSingleton.shared

print(structSingleton2.value) // output: 1
class ClassSingleton {
    static let shared = ClassSingleton()
    var value = 1
    private init() {
    }
}


var classSingleton1 = ClassSingleton.shared
classSingleton1.value = 2
var classSingleton2 = ClassSingleton.shared

print(classSingleton2.value) // output: 2
```
- 구조체로 싱글톤을 구현한 경우 같은 인스턴스를 참조하지 못해 `structSingleton1.value`값이 1이 되는 것을 볼 수 있다. 그 이유는 구조체가 값타입이기 떄문에 별개의 인스턴스가 되기 떄문이다. 반면에 클래스는 힙 영역의 객체를 가리키고 있기 때문에 값이 변하고 이는 싱글톤 객체가 변하지 않음을 보장할 수 있다.


#### 장점과 단점
| 장점 | 단점 | 
| -------- | -------- | 
| ▪️ 하나의 인스턴스만을 사용해 메모리 낭비를 줄일 수 있다. <br/> ▪️ 싱글톤 인스턴스는 전역 인스턴스이기 때문에 다른 클래스와의 자원 공유가 쉽고 접근이 쉽다. <br/> ▪️  Swift에서는 static let을 사용해서 인스턴스를 생성시키면 사용시점에 초기화될 수 있기 때문에 하나의 인스턴스만을 생성하는 것을 보장한다. 즉 thread-safe하다.| ▪️ 싱글톤 인스턴스가 너무 많은 일을 하거나 너무 많은 데이터를 공유할 경우 결합도가 높아진다. <br/> ▪️ 많은 테스트 프레임워크들이 모의 객체들을 생성할 때 상속에 의존하기 때문에 유닛 테스트하기가 어렵다 <br/> ▪️ 클래스의 작업을 처리하는 역할 뿐 아니라 자신의 인스턴스에 대한 접근을 관리하는 역할에 대해서도 책임을 져야해서 단일 책임원칙을 위반한다.| 

#### 주소값을 통한 비교
- 주소값을 통해서 **구조체 일때는 같은 인스턴스를 가리키는 것이 아니다**라는 결과를 도출시키고 싶었으나 실패했다. 클래스 일 때도 같은 인스턴스의 주소값을 가리키지 않는 것이었다. 그래서 고민끝에 다음과 같은 결론을 내렸다.(정확하진 않습니다..😂)
> UnsafeRawPointer타입을 이용한 주소값 판별은 참조하고 있는 주소를 나타내는 것이 아니라 메모리의 주소를 나타낸다
```swift
class A {
    var a = 1
}
var test = A()
var test2 = test
address(of: &test) 
address(of: &test2)
```
- 위 두 개의 address는 다르다. 클래스인데도 주소가 다르게 나오는 이유는 test와 test2는 각각 새로운 메모리 주소를 받기 때문이다. 하지만 두 변수가 가리키는 객체는 동일하기 때문에 test의 프로퍼티를 수정하더라도 test2에서 확인할 수 있다. 이 때문에 구조체와 클래스에서의 싱글톤을 비교할 때 주소가 아닌 내부 프로퍼티의 값을 통해서 비교할 수 밖에 없었다.


## Git 특정 커밋만 적용시키기(cherry-pick)
- 프로젝트를 진행하는데 요구사항을 순서대로 구현해야 하는데 역순으로 구현을 해서 커밋이 꼬였었다. (STEP1 -> STEP2 -> STEP3로 구현한 상황)
- STEP1 -> STEP2로 변경하고 PR을 보내야하하기 때문에 이번 기회에 cherry-pick이라는 깃의 기능을 사용해봐야겠다고 생각했다.
### Cherry-pick
> git을 사용하다보면 커밋을 다른 브랜치에 잘못 하거나, 요구사항이 바뀌어 필요 없는 커밋이 생기거나, 코드 의존성 때문에 다른 사람의 커밋 중 일부를 가져와야 할 때 사용한다.

### Cherry-pick 사용하기
- 현재까지 작업한 내용을 토대로 새로운 브랜치 STEP2라는 이름으로 만들었다.
- STEP1을 마무리한 시점으로 `git reset --hard`명령을 줬다.
- 기존에 작업했던 브랜치로 돌아가서 `git log --oneline`을 통해 커밋이력을 확인했다.
- 다시 STEP2로 체크아웃한 후 띄워진 로그의 커밋해쉬를 보면서 필요한 부분을 cherry-pick해서 STEP2브랜치에 적용했다.

### 잘 풀리지 않았던 부분
- cherry-pick해야 하는 커밋이 상당수 존재했기에 ...을 사용해서 한 번에 적용하려고 했다. 
```bash
git cherry-pick 시작해쉬^..마무리해쉬
```
- 하지만 충돌이 있는 경우 특정 커밋에서 멈춰지고 다음과 같은 문구가 나온다.
<img src="https://i.imgur.com/RVTrte5.png" width="500"/>
- both modifed부분이 충돌파일이며 파일에서 conflict을 해결하고 git add 파일이름을 하면 해결이 된다.

이후 잘 해결이 되나...?싶었으나
main 스토리보드가 열리지 않는 현상이 발생했다..😂
추측으로는 기존의 STEP1 -> STEP3 -> STEP2로 구현을 했었기 때문에 STEP2를 cherry-pick하는 과정에서 STEP3의 코드가 main 스토리보드에 섞여있었고 이게 충돌이 일어난 것 같다. (메인 스토리보드의 Storyboard ID같은 것들을 염두해두고 적용시켜야 하는 것 같다.) 결국 잘 해결은 했지만 무언가 찝찝함이 남아있다.

### 결론
- 다수의 커밋에 대해 cherry-pick을 사용할 때는 `git cherry-pick --continue`를 통해 커밋 각각에 대해 충돌을 해결하자. 
- 메인 스토리보드 ID와 같은 스토리보드의 수정사항을 꼼꼼히 반영하고 해결하자.
- ...을 통해 다수의 커밋을 적용할 때 만약 시작해쉬를 포함한다면 ^(웃음 표시)를 시작해쉬에 적어주자.
- cherry-pick은 본래 권장하는 명령은 아니다. cherry-pick을 하는 경우 같은 내용을 갖고있는 커밋이 여러개 생긱 ㅣ때문에 나중에는 누가 누굴 cherry-picking했는지 모르는 상황이 생길 수도 있다. 하지만 부득이하게 사용해야 할 때가 온다면 그 때 사용하면 된다.


###### tags: `싱글톤`, `cherry-pick`


### Reference
- [Apple Archive - Singleton](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/Singleton.html)
- [git - cherry-pick](https://cselabnotes.com/kr/2021/03/31/56/)

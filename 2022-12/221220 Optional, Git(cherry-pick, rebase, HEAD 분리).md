
221220 Optional, Git(cherry-pick, rebase, HEAD 분리)
===
TIL (Today I Learned)
===
학습내용
---
- Optional 이해하기
- git 명령어 이해하기

고민한 점 / 해결 방법
---
#### 학습한 내용
- Optional
    - 옵셔널은 값이 없을 수 있는 상황에 사용하는 타입이다. 이는 enum 열거형으로 되어있으며 값이 존재할 때는 some(Type), 값이 존재하지 않는다면 nil로 되어있는 swift의 안정성을 높여주는 타입이다.
    - 옵셔널값을 안전하게 추출하는 방법은 if nil체크, if let 바인딩, 닐 코어레싱 3가지가 존재한다. 만약 안전하게 추출하지 않는다면 런타임에러를 야기시킬 수 있다. 
    - 옵셔널 바인딩은 Boolean 조건식과 같이 사용할 수 있다.<br><br>
    ```swift
        if let firstNumber = Int("4"), let secondNumber = Int("42"),
            firstNumber < secondNumber && secondNumber < 100 {
            print("\(firstNumber) < \(secondNumber) < 100")
        }
    ```
    <br>
    
    - if var 변수의 형태로도 옵셔널값을 안전하게 추출할 수 있다. 이 경우에는 if var 스코프 안에서 unwrapping된 변수가 변할 때 사용하게 되는데, 바인딩되고 추출된 값은 if scope안에서만 동작하기 때문에 if let을 관용적으로 사용하는 것으로 보인다.
    - Implicitly Unwrapped Optional(암묵적 강제언래핑 옵셔널)은 프로그램 구조상 초기에 값이 설정된 후 옵셔널이 항상 값을 가지고 있을때 사용한다. 이는 옵셔널 변수에 접근할때마다 언래핑해야하는 단점을 없앨 수 있다. 그럼 String! 대신 String을 쓰면 문제가 해결되는 게 아닐까? 그렇지 않다. 아래 class initialization 시점의 예시를 보면 암묵적 강제언래핑 옵셔널을 사용하는 이유를 알 수 있다.<br><br>
    ### Implicitly Unwrapped Optional이 필요한 이유
    - Country와 City가 존재하고 이 둘은 서로를 소유하는 프로퍼티를 가지고 있다고 가정해보자. 아래 코드를 빌드해보면 주석 1번줄에서 self를 넘길 수 없기 때문에 컴파일 에러가 나온다. Country 인스턴스가 fully initialized되기 전까지는 self를 넘길 수 없기 때문이다.<br><br>
    

    ```swift
    class Country {
        let name: String 
        var capitalCity: City
        init(name: String, capitalName: String) {
            self.name = name
            self.capitalCity = City(name: capitalName, country: self) // ----- 1
        }
    }

    class City {
        let name: String
        unowned let country: Country
        init(name: String, country: Country) {
            self.name = name
            self.country = country
        }
    }
    ```
    그래서 이를 옵셔널로 바꿔주면 해결이 된다.
    ```swift
    class Country {
        let name: String 
        var capitalCity: City? // <------ 옵셔널로 변화
        init(name: String, capitalName: String) {
            self.name = name
            self.capitalCity = City(name: capitalName, country: self) // ----- 1
        }
    }
    ```
    하지만 Country의 capitalCity는 인스턴스 생성때부터 계속해서 존재하는 값인데도 capitalCity에 접근하기 위해선 안전하게 추출하는 방법을 거쳐야한다. 이 때문에 초기화되기전에는 옵셔널이지만 초기화된 이후부터는 반드시 값이 존재하는 Implicitly Unwrapped Optional를 사용하는 것이다.
    ```swift
    class Country {
        let name: String 
        var capitalCity: City!  // <------- 암묵적 강제언래핑으로 변화
        init(name: String, capitalName: String) {
            self.name = name
            self.capitalCity = City(name: capitalName, country: self) // ----- 1
        }
    }
    ```
    
    
### 결론
코어데이터를 다루는 Realm라이브러리를 불러오거나 UILabel!에 !를 붙인 이유를 잘 몰랐는데 이번 정리로 잘 알게 되었다. <span style="color: #e83e8c">인스턴스가 생성되려면 모든 프로퍼티가 초기화가 되어야하고 생성되는 시점 이후로는 값이 보장되어야 하기 때문이다.</span>
- git
    - git cherry-pick [커밋해쉬]: 다른 브랜치의 커밋해쉬를 참조해 그 커밋을 현재 checkout된 브랜치에 추가한다.  
    - git rebase [브랜치명]: 현재 checkout된 브랜치에서 인수로 넣은 브랜치명으로 합치는 방법.
### [merge vs rebase vs cherry-pick]
브랜치를 통합하는 방법에는 크게 merge, rebase와 좀 더 세분화해서 cherry-pick이 존재한다. 이들은 각자 띄고있는 성격이 다르기 때문에 상황에 맞게 쓰는 게 중요하다.
#### merge
두 개의 브랜치로 나누어진 커밋 히스토리를 가장 쉽게 합치는 방법은 merge를 사용하는 것이다. 공통 조상(C2)와 마지막 커밋 두개(C3, C4)를 이용하는 3-way Merge로 새로운 커밋을 만들어내는 특징이 있다.<br>
<img src="https://i.imgur.com/EcCzJII.png)" width="500"/>

#### rebase
merge와 비슷한 결과를 만드는 다른 방식으로, C4에서 변경된 사항을 Patch로 만들고 이를 master 브랜치에 적용하는 방법이 있다.
```bash
$ git checkout experiment
$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: added staged command
```
<img src="https://i.imgur.com/qLV7WEd.png" width="500"/><br>

실제로 일어나는 일을 조금 더 구체적으로 설명하자면 C4에 있던 experiment브랜치에서 C3와의 공통조상인 C2를 기준으로 차이점(diff)를 만들어 어딘가에 임시로 저장해 놓는다. 그리고 rebase될 브랜치인 experiment브랜치가 master에 합쳐지면서 변경사항을 차례대로 적용시킨다.
마지막으로 master 브랜치를 Fast-forward merge를 시키면 깔끔하게 정리가 된다. 
그럼 rebase는 왜 사용하는걸까?

아무래도 선형으로 히스토리가 기록되기 때문인 이유가 가장 크다. 보기에 깔끔하고 일을 병렬로 동시에 진행했다하더라도 모든 작업이 차례로 진행된 것처럼 보인다.
이렇게 좋은 rebase에도 잊지 말아야 할 치명적인 단점이 있다. 

> 이미 공개 저장소에 Push한 내용을 rebase 하지마라

rebase는 기존의 커밋의 내용을 이용하지만 그 커밋 자체를 이동시키는 것이 아니다. 그렇기 때문에 협업하면서 이미 push된 코드를 누군가 pull을 했는데, 그 이후에 rebase를 하고 다시 push를 한다면 처음 pull을 했던 팀원이 혼란을 겪게 된다.(같은 이름의 커밋이 2개가 되는 기이한 현상이...)


#### cherry-pick
cherry-pick은 다른 브랜치의 특정 커밋을 내 브랜치에 가져오고 싶을 때 사용한다. 
같은 내용의 커밋을 여러개 생성하기 때문에 꼭 사용해야하는 상황이 오지않는다면 사용하지 않는게 좋아보인다. 
#### 사용방법
```
git cherry-pick 34b4cac
git cherry-pick 12as23f

// 두개도 가능하다
git cherry-pick 34b4cac 591abc
git cherry-pick 12as23f 291fxc
```
체리픽 충돌이 일어난다면 다음의 순서로 해결한다.
1. 충돌난 코드를 수정
2. git add {충돌난 파일 경로}로 충돌 해결한 코드 추가
3. git cherry-pick --continue로 cherry-pick을 다시 진행


체리픽을 중단시키고 싶다면
`git cherry-pick --abort`로 중단한다.


Reference
---
- [Apple Developer Documentation Optional](https://developer.apple.com/documentation/swift/optional)
- [Swift Docs-Optional](https://docs.swift.org/swift-book/LanguageGuide/TheBasics.html)
- [Implicitly Unwrapped Optional개념과 예제](https://eunjin3786.tistory.com/492)
- [What is the different between `if var` and `if let` in swift?](https://stackoverflow.com/questions/30526190/what-is-the-different-between-if-var-and-if-let-in-swift)
- [Git브랜치 Rebase하기](https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-Rebase-%ED%95%98%EA%B8%B0)
- [stepup-CherryPick](https://backlog.com/git-tutorial/kr/stepup/stepup7_4.html)

###### tags: `git`, `rebase`, `cherry-pick`, `Optional`

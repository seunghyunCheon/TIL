
221219 FlowChart, Git, 짝 프로그래밍
===
TIL (Today I Learned)
===
학습내용
---
- FlowChart에 대해 알아보기
- Git의 기본명령어들을 사용해보고 실습하기
- 짝 프로그래밍에 대해 알아보기

고민한 점 / 해결 방법
---
#### Git 정리
- Git 개념, 초기 세팅
    - git은 VCS(Version Control Sytstem)으로 파일의 변경이력을 용이하게 해주는 시스템이다.
    - git config --global user.name</span>, git config --global user.email을 초기에 설정함으로써 터미널에서 커밋 로그에 찍힐 이름과 이메일을 정할 수 있다.
    - git status에는 Tracked상태와 Untracked상태 두 개로 나뉜다.<br>
    <img src="https://i.imgur.com/mzq1Qp5.png" width="500"><br><br>
    - Tracked상태와(추적되고 있는 파일)에는 Unmodified, Modified, Staged 3가지 상태가 존재하며 `git add` 명령어로 인해 추적되고 있는 파일들이다.
    - Unmodified상태는 처음 레포지토리를 clone했을때의 비어있는 상태로이다.
    - branch는 독립적으로 어떤 작업을 진행하기 위한 개념이다. 각각의 브랜치는 서로 독립적이기 떄문에 영향을 주지않고 작업할 수 있다는 특징이 있다.
    - revert는 새로운 커밋을 추가하면서 커밋을 되돌린다.
    - reset은 이전 커밋으로 돌아간다는 점에서는 revert와 동일하지만 커밋을 추가하면서 되돌리는 것이 아닌 아예 그 커밋으로 돌아간다는 점에서 차이가 있다.
- Git 명령어
     - `git init`: 깃 저장소를 생성하는 명령어. git system을 이용하려는 디렉토리로 이동해 이 명렁어를 실행하면 사용할 수 있다.
    - `git add`: 파일을 추적할 수 있게 staging area에 올려놓는 명렁어. 
    - `git commit -m "message"`: git repository에 올려지는 명렁어로 변경이력을 저장하는 공간이다. 이 부분은 log가 남는 곳이다.
    - `git branch test1`: test1이름의 브랜치를 만든다. 
    - `git branch -m test1 test2`: test1이름의 브랜치를 test2로 바꾼다.
    - `git checkout test1`: test1 브랜치로 이동한다.
    - `git checkout -b test3`: test3브랜치를 만들고 test3로 이동한다.
    - `git merge test1`: 현재 브랜치에서 test1브랜치를 merge한다.
    - `git revert (커밋해쉬)`: 커밋해쉬에 해당하는 부분을 되돌린다.

#### [revert vs reset]
revert와 reset은 이전의 커밋으로 돌아가고 싶을때 사용한다는 점을 제외하면 많이 다르기 때문에 용도에 맞게, 신중하게 사용해야한다.   
#### reset
```bash
git commit -m "1"
git commit -m "2"
git commit -m "3"
git reset --hard [1번 commit hash]
```
위의 명렁어는 1번으로 이동을 하면서 2번과 3번은 완전히 지우는 코드이다. --hard옵션은 변경이력과 변경내용 둘 다 완전히 지우기 때문에 --hard사용은 가급적 사용하지 않는 것이 좋다. --mixed는 변경이력은 날아가지만 변경내용은 있는 상태, --soft는 mixed상태에서 staged된 상태이다.

#### revert
```bash
git commit -m "1"
git commit -m "2"
git commit -m "3"
git revert [1번 commit hash]
```
위의 명렁어를 사용하면 어떻게 될까? git log를 찍으면 다음과 같이 보인다.
```bash
Revert "1번 커밋"
"3번 커밋"
"2번 커밋"
"1번 커밋"
```
reset과 다르게 새로운 커밋이 생긴 걸 볼 수 있다. 그리고 실제로 이력에는 1번 커밋의 내용이 없어지고 2번과 3번커밋의 내용만 존재하는 것을 볼 수 있다. 즉 특정 커밋에 해당하는 내용만 이전으로 돌릴 수 있다.

### 결론
팀원들과 협업할 때는 어떤 부분을 되돌리려 했는지, 왜 되돌리려했는지를 의도를 알 수 있는 커밋이 찍혀있기 때문에 revert를 사용하는 게 좋아보인다.

#### FlowChart, 짝 프로그래밍
- FlowChart
    - 플로우차트(Flowchart)란 프로세스 수행에 있어 필요한 처리와 과정들을 시각적으로 표현한 차트이다.
    - 플로우차트는 분석이 명료해지고, 다른사람들이 알아보기 쉬우며 논리적인 오류를 찾기 쉽다는 장점이 있다. 
    - 플로우차트 도형의 기본적인 기능은 다음과 같다.<br>
    <img src="https://i.imgur.com/RAgTJPx.png" width="500"><br><br>
- 짝 프로그래밍
> 두 사람이 한 짝이되어 같이 프로그래밍하는 것을 말한다.
> 
짝 프로그래밍은 3가지 장점이 존재한다.
1. 결함수가 적어진다. 
    - SW를 개발할 때 예상치 못하게 개발속도를 잡아 먹는 것이 '결함'인데, 결합의 삽입과 해결은 완전히 비대칭 구조로 되어있기 때문에 초기에 예방하는 것이 중요하다.
2. 통합시간이 줄어든다.
  - 팀원 간 코드스타일과 방식들이 전부 제각각이기 때문에 통합비용은 상당히 많은 비용을 차지한다. 하지만 짝 프로그래밍을 함으로써 처음부터 통합을 고려하면서 코드를 작성하기 때문에 시간이 적게 걸린다. <span style="color: #e83e8c">시작하는 과정에서부터 계속해서 방향을 조정하기 때문에 적게 걸리는 것이다.</span>
3. 팀워크를 향상시킨다.
    - 서로 긴밀하게 소통을 하면서 작업을 하기 때문에 협력의 기술이 늘어난다. 그리고 이 과정에서 계속해서 피드백을 하기 때문에 프로젝트는 계속 깨지기 쉬운 상태가 된다. 에자일은 모든 프로세스를 깨지기 쉽게 만드는 것이라고 할 수 있다. 즉 문제를 빠르게 인식하고 빠르게 해결하는 것이 핵심이기 때문에 장기적인 관점에서 더 빠르고 안정적인 개발이 가능해진다.<br>

<span style="color: #e83e8c">전체적인 그림확인 -> 구체적으로 진행 -> 다시 전체적인 그림을 확인</span>
짝 프로그래밍은 역할을 빠르게 교체하는 것이 중요하다. 에자일은 추상과 구상 사이를 왔다갔다하면서 **발견**이 있기 때문이다.



Reference
---
- [브랜치란?](https://backlog.com/git-tutorial/kr/stepup/stepup1_1.html)
- [git reset, revert](https://kyounghwan01.github.io/blog/etc/git/git-reset-revert/#revert)
- [Git의 기초 - 수정하고 저장소에 저장하기](https://git-scm.com/book/ko/v2/Git%EC%9D%98-%EA%B8%B0%EC%B4%88-%EC%88%98%EC%A0%95%ED%95%98%EA%B3%A0-%EC%A0%80%EC%9E%A5%EC%86%8C%EC%97%90-%EC%A0%80%EC%9E%A5%ED%95%98%EA%B8%B0)
- [FlowChart 작성법](https://gongbu-ing.tistory.com/92)
- [smartDraw](https://www.smartdraw.com/flowchart/flowchart-symbols.htm)
- [짝 프로그래밍](https://gmlwjd9405.github.io/2018/07/02/agile-pair-programming.html)
###### tags: `git`, `revert`, `flowChart`, `Pair Programming`

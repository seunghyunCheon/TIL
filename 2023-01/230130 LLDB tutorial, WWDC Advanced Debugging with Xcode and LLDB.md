230130 LLDB tutorial, WWDC Advanced Debugging with Xcode and LLDB 
===
TIL (Today I Learned)
===
학습내용
---
- LLDB tutorial, 사용
- WWDC Advanced Debugging with Xcode and LLDB 시청
## LLDB tutorial

### LLDB breakpoint set
파일 안에 line breakpoint를 걸고 싶다면
```bash
(lldb) breakpoint set --file ViewController --line 9
(lldb) br s -f ViewController.swift --l 9
```

함수에 breakpoint를 걸고 싶다면
```bash
(lldb) breakpoint set --name executeTwo
(lldb) br s -n executeTwo
```
여러개의 함수에는
```bash
(lldb) breakpoint set --name excuteOne --name executeTwo
(lldb) br s -n executeOne -n executeTwo
```
더 구체적으로 메서드 옵션을 사용할 수 있다.
```bash
(lldb) breakpoint set --method executeOne
(lldb) br s -M executeOne
```

Objective-C selector에는 
```bash
(lldb) breakpoint set --selector alignLeftEdges:
(lldb) breakpoint set -S alignLeftEdges:
```

shlib option을 이용해 공유 라이브러리의 범위로 breakpoint를 더 제한할 수도 있다. 
```bash
(lldb) breakpoint set --shlib foo.dylib --name foo # foo.dylib라이브러리의 foo라는 이름의 메서드에 breakpoint
(lldb) breakpoint set -s foo.dylib -n foo
```
alias를 이용해 별칭을 달아줄 수 있다.
```bash
(lldb) command alias bfl breakpoint set -f %1 -l %2
(lldb) bfl ViewController.swift 17
```
unalias를 이용해 지울 수도 있다.
```bash
(lldb) command unalias b
```

list를 이용해 설정한 breakpoint들을 볼 수 있다
```bash
(lldb) breakpoint list
```

### Breakpoints list

list를 이용해 설정한 breakpoint들을 볼 수 있는데 만약 selector에 대한 breakpoint를 설정한다면 프로그램 내 selector를 구현한 메서드들에 breakpoint가 걸리게 된다. 

이 breakpoint들은 정수 id를 갖게되는데 부모의 breakpoint안에 id를 갖게 된다. 

<img src="https://i.imgur.com/WgFFey2.png" width="500"/> 

만약 다른 공유 라이브러리에서 특정 selector를 구현해서 breakpoint가 설정되었지만 로드되지 않는다면 더이상 resolve할 수 없게 된다.
이에 대해 delete, disable, set condition and ignore할 수 있다.

### Execution Commands
- continue: 정지된 프로그램 실행 재개
```bash
(lldb) continue
(lldb) c
```
- Step over: 현재 선택된 Frame에서 소스 수준의 한 단계를 진행
```bash
(lldb) thread step-over
(lldb) next
(lldb) n
```
- Step Out: 현재 선택된 Frame에서 벗어남
```bash
(lldb) thread step-out
(lldb) finish
```

### Examing Variables - 변수 검사
- Frame Variable - 메모리에서 변수를 읽어 lldb 형태의 description을 출력
```bash
# 현재 frame에 argument와 local 변수 출력
(lldb) frame variable
(lldb) fr v

# local 변수 `bar` 내용 출력
(lldb) frame variable bar
(lldb) fr v bar

# Object의 Description 출력
(lldb) frame variable -O self
```
- Target Variable - global/static 변수를 출력
```bash
# global 변수 `baz` 내용 출력
(lldb) target variable baz
(lldb) ta v baz

# 현재 소스 파일에서 정의된 global/static 변수 출력
(lldb) target variable
(lldb) ta v
```

### 특정 메모리 주소의 값을 출력
```bash
(lldb) expr -l swift -- import UIkit
(lldb) expr -l swift -- let $vc = unsafeBitCast(0x주소), ViewController.self)
(lldb) po $vc
```

### Stack, Frame, Thread
> 정확하진 않습니다. 확실한 자료가 보이지않아 뇌피셜로 작성
- Stack은 함수가 호출될 때마다 함수 Frame이 생기는데 이곳에는 지역변수와 돌아갈 주소 등을 담고 있다. 이와 비슷한 맥락으로 Thread가 생길 때 Stack이 생기고 그 Stack 내부에는 여러개의 debugging Frame이 생겨, stepping-in, stepping-out을 통해 스레드의 실행흐름을 제어하는 것이다. 


## WWDC Advanced Debugging with Xcode and LLDB

#### 디버깅 목적으로 코드를 수정하는 건 좋지않다. 빌드시간이 오래걸리는 프로젝트라면 상당히 비효율적이다.
- `expression didReachSelectedHeight = false`처럼 디버깅 중에 표현식을 사용해 코드의 흐름을 테스트한다.
- 반복적인 표현식을 사용하기 위해서 breakPoint에 걸릴 때마다 Action을 걸어줄 수 있다. 
- 현재 살아있는 애플리케이션을 사용하여 테스트할 수 있도록 breakpoint를 이용해 변경 사항을 주입하는 것이 핵심

#### UIKit에 대한 소스 코드가 없어도 함수에 전달된 인자를 검사할 수 있다.
- 만약 [UILabel setText:]에 breakpoint를 걸게 된다면 다음처럼 확인할 수 있다.
```bash
po $arg1
<UILabel: 0x주소; frame .....>
po (SEL)$arg2  #selector를 출력
"setText:"
po $arg3 # 메소드에 전달된 첫 번째 매개변수
0ft
```
- 어셈블리 프레임워크에 있는 경우 인수를 검사하는데 매우 편리한 방법
- br s one-shot은 한 번만 쓰이고 breakpoint list에서 삭제된다.

#### 명령 포인터를 옮겨 스레드를 조작할 수 있다.
<img src="https://i.imgur.com/FPwgrPV.png" width="500"/>

오른쪽에 Thread 1: breakpoint 8.1을 명령포인터라고 부른다. 마우스로 이를 드래그해서 스레드의 흐름을 조작할 수 있다. 
하지만 이는 꽤나 위험한 기능이다. 명령 지점을 원하는 곳으로 이동은 하지만 응용 프로그램 상태가 원하는 대로 동작한다고 보장할 수 없기 때문이다. 

#### 사용자 지정 LLDB 명령을 추가할 수 있다.
- LLDB의 Python 스크립팅을 사용하여 명령을 만들고 사용할 수 있다. (python의 nudge를 통해 offset과 주소값을 받아 쉽게 객체의 레이아웃을 디버깅 할 수 있음.)

#### Etc+
`expression CATrasaction.flush()`를 통해 애니메이션을 업데이트할 수 있다.


- [LLDB - tutorial](https://lldb.llvm.org/use/tutorial.html)
- [Xcode debugging - 민소네](https://minsone.github.io/ios/mac/xcode-lldb-debugging-with-xcode-and-lldb)
- [WWDC Advanced Debugging with Xcode and LLDB](https://developer.apple.com/videos/play/wwdc2018/412)

221230 Xcode Debug, 재귀함수와 반복문, 꼬리재귀함수
===
TIL (Today I Learned)
===
학습내용
---
- Xcode Debug 학습하기
- 반복문과 재귀함수의 장단점

## 고민한 점 / 해결 방법
#### [Xcode에서 디버그하는 방법]
> 반복문과 재귀함수가 어떻게 동작하는지 확인하기 위해 Xcode에서 디버그하는 방법을 찾아보게되었다.
>
> 코드환경에서 디버그하는 방법은 console을 이용하는 방법과 breakPoint를 이용하는 방법이 있다. 

- 어느 지점에서 데이터가 존재하지 않는지 혹은 메모리를 사용하는지 확인하려면 코드 왼쪽 column를 클릭해 breakPoint를 찍는다.<br>
<img src="https://i.imgur.com/O6AoTPD.png" width="400"/><br>

- 파란색 부분을 더블 클릭하면 아래와 같은 화면이 나오게 된다.<br>
<img src="https://i.imgur.com/UGLDwGk.png" width="400"/>
    - Name: 해당 breakPoint에 이름을 설정해주는 부분
    - Condition: 특정 조건을 넣어 이 조건에 해당하면 멈춤
    - ignore: 특정 횟수까지는 무시하고 진행
    - Action: breakPoint에 걸렸을 때 특정 로그를 찍거나 lib의 명령어를 실행 
> Automatically continue after evaluating actions는 위의 Actions에 해당하는 명령을 실행하고 멈추지않고 바로 넘어가기 위한 옵션이다.

- 만약 재귀함수의 인자에 1000을 넣고 breakPoint를 걸어 실행하게 된다면 아래 화면이 보이게 된다.

    <img src="https://i.imgur.com/cAUUJfp.png" width="400"/>
- 예상치 못한 부분에서 프로퍼티에 값이 잘 들어왔는지 확인할 때 유용할 것 같다. 
- 추가적으로 watch라는 기능이 있는데 이는 watch로 설정한 변수가 수정되는 모든 부분에서 멈추게하는 기능이 있었다. <br>
    <img src="https://i.imgur.com/AZrdJ4D.png" width="300"/> 
- 먼저 변수가 초기화된 이후에 breakPoint를 찍어놓고 실행을 하면<br>
    <img src="https://i.imgur.com/E2MfIy5.png" width="300"/>
- 이전 breakPoint와 같이 나타나고 감시하고 싶은 변수에 마우스 오른쪽을 클릭해 Watch "count"를 누르면 해당 프로퍼티가 수정되는 곳마다 breakPoint가 자동으로 찍히게 된다.
- ### 하지만
- watch는 제한된 자원이기 때문에 사용할 수 있는 타입이 한정돼있다 . Int, float, Double, String 각각의 타입들이 가지는 메모리는 다음과 같다.<br>
<img src="https://i.imgur.com/L0DfTPU.png" width="300"/>
- String의 경우 16바이트를 가지고있기 때문에 watch하는 순간 해당 사이즈를 watch할 수 없다고 나온다.
- Int의 경우는 의도대로 수정되는 부분에서 잘 멈추지만 float과 double은 이상하게 특정 부분을 건너 뛰게 된다.
<img src="https://i.imgur.com/XQgBeY3.png" width="300"/>
- 짐작으로는 float과 double에서 메모리를 할당하는 방법이 Int와 달리 지수부분과 가수부분이 있기 때문인 것 같은데 정확하지 않다. (watch에 대한 부분을 나중에 더 파봐야겠다.)
- 결론적으로 현재 상태에서 watch를 사용할 때는 Int형만 사용해야겠다고 생각했다.

[재귀함수, 반복문 그리고 꼬리재귀함수]

> 프로젝트의 내용으로 재귀함수와 반복문은 어떤 차이가 있고 또 꼬리 재귀함수라는 것은 무엇인지 궁금해졌다.

- 반복문, 재귀함수, 꼬리 재귀함수 각각의 특징들을 알기위해 Xcode 디버그를 이용했다. 
- 실행 속도를 비교하기 위해서 `Foundation`을 임포트하고 TimeInterval과 CFAbsoluteTimeGetCurrent를 사용해서 비교했다. 사용법은 progressTime함수 아규먼트 안에 실행시간을 확인해볼 함수를 넣으면 된다.
```swift
    func progressTime(_ closure: () -> ()) -> TimeInterval {
        let start = CFAbsoluteTimeGetCurrent()
        closure()
        let diff = CFAbsoluteTimeGetCurrent() - start
        return (diff)
    }

    print(progressTime {
        print()
    })
```

- 먼저 반복문과 재귀함수를 정의하고 그 안에 breakPoint를 찍어 serial-queue안에 어떻게 호출되는지와 실행시간을 알아봤다.
```swift
func 재귀함수(_ n: Int) -> Int {
    guard n > 0 else { return 0 }
    return n + 재귀함수(n-1)
}

func 반복문(_ n: Int) -> Int {
    var num = 0
    for i in stride(from: n, through: 0, by: -1) {
        num += i
    }
    return num
}

func 꼬리재귀함수(_ n: Int, _ acc: Int) -> Int {
    guard n > 0 else { return acc }
    return 꼬리재귀함수(n - 1, acc + n)
}
```

- 재귀함수는 함수를 계속해서 호출해야 하는 구조기 때문에 계속 쌓이는 것을 볼 수 있고 반복문은 CPU가 명령을 계속해서 호출하기 때문에 쌓이지 않는 것을 볼 수 있다.

||재귀함수|반복문|
|:------:|:---:|:---:|
|Serial-Queue|<img src="https://i.imgur.com/viwOviS.png" width = "300"/>|<img src="https://i.imgur.com/cAWnN2i.png" width = "300"/>|

### 재귀함수 반복문 실행시간 (10,000입력 기준)
<img src="https://i.imgur.com/6NZ9f5Y.png" width="300"/><br>
> 위: 재귀함수 
> 아래: 반복문

### 꼬리재귀함수 재귀함수 실행시간 (10,000,000입력 기준)
<img src="https://i.imgur.com/PBELAC8.png" width="300"/><br>

> 위: 꼬리재귀함수
> 아래: 재귀함수

### 비교

||반복문|재귀함수|꼬리 재귀함수|
|:------:|:---:|:---:|:------------:|
|속도|빠름|느림|매우 빠름|
|실행|명령을 반복적으로 실행| 함수를 호출 | 함수를 호출 |
|스택메모리|사용하지 않음|함수 호출 될때마다 사용|스택프레임 하나만 사용|
|가독성|코드 길이가 많아져 떨어짐|코드 길이가 줄어 높아짐|코드 길이가 줄어 높아짐
|무한반복| CPU 사이클을 반복적으로 사용 | 스택 오버플로우 발생 | 한번만 실행되기 떄문에 스택오버플로우 미발생| 


### 결론
쉬운 코드에서는 재귀함수의 시간복잡도를 예상할 수 있지만 복잡하게 얽혀있는 코드에서는 종료분기점이 명확하지 않기 때문에 계산하기가 어렵다. 그리고 알고리즘에서 간결하게 표현되는 식을 사용할 때 많이 사용하는 것 같다. 하지만 재귀함수는 스택오버플로우가 발생할 수 있기 때문에 꼬리재귀함수로 바꿀 수 있다면 바꾸는 게 좋다.
그리고 반복문은 시간복잡도가 예상가능하고 쉽게 코드르 읽을 수 있기 때문에 순차적으로 작성해야 하는 상황이 올 때 사용하면 좋을 것 같다!

### Reference
- [Apple Developer Documentation - Debugging](https://developer.apple.com/documentation/xcode/diagnosing-and-resolving-bugs-in-your-running-app)
- [디버깅을 활용한 Xcode 활용 방법](https://meetup.nhncloud.com/posts/315)
- [컴파일 최적화 꼬리재귀](https://velog.io/@cherrish_red/iOS-%EC%BB%B4%ED%8C%8C%EC%9D%BC-%EC%B5%9C%EC%A0%81%ED%99%94-feat-%EA%BC%AC%EB%A6%AC%EC%9E%AC%EA%B7%80)
- [Apple Developer Documentation - MemoryLayout](https://developer.apple.com/documentation/swift/memorylayout)
###### tags: `Debug`, `꼬리재귀`, `MemoryLayout`

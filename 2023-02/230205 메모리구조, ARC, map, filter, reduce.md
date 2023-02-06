230205 메모리구조, ARC, map, filter, reduce
===
TIL (Today I Learned)
===
학습내용
---
- 메모리구조 학습하기
- RC, MRC, ARC
- map, filter, reduce 만들어보기

## ✏️ 메모리 구조
프로그램이 실행되기 위해서는 프로그램이 메모리에 로드되어야 한다. 또 프로그램에서 사용되는 변수들을 저장할 메모리도 필요하다.

따라서 컴퓨터 운영체제는 프로그램 실행을 위해 4가지의 메모리 공간을 제공한다.
<img src="https://i.imgur.com/mvTDkxj.png" width="200"/>

### 코드 영역
실행할 프로그램의 코드가 저장되는 영역. CPU는 코드 영역에 저장된 명령어를 하나씩 가져가서 처리한다.

### 데이터 영역
전역변수와 정적(static)변수가 저장되는 영역. 프로그램 시작과 함께 할당되며 프로그램이 종료되면 소멸한다.

### 스택 영역
함수의 호출과 관계되는 지역 변수와 매개변수가 저장되는 영역
함수의 호출과 함께 할당되며, 함수 호출이 완료되면 소멸한다.

### 힙 영역
사용자가 직접 관리할 수 있는 메모리 영역.
힙 영역은 사용자에 의해 메모리 공간이 동적으로 할당되고 해제된다. (Swift는 ARC가 한다)

## ✏️ ARC

### RC

`Reference Counting`의 약어로 애플에서 메모리를 관리하는 방법이다.

메모리를 할당하거나, 메모리 포인터를 참조할 때 레퍼런스 카운트를 증가시키고, 사용을 완료하면 레퍼런스 카운트를 감소 시킨다. 

Objective-C에는 개발자가 직접 참조 관리를 했다고 한다.
이를 `MRC`라고 한다.

### MRC
`Manual Reference Counting`의 약어로 이름만봐도 수동으로 참조 카운트를 관리하는 것을 짐작할 수 있다. 
레퍼런스 증가는 `alloc`, `new`, `mutableCopy`, `retain` 을 사용했고 감소는 `release`를 사용했다.

### ARC, GC
ARC는 컴파일 타임에 기존 MRC때 개발자가 직접 코드를 작성해야하던 부분을 자동으로 구문 분석해서 관리를 한다. 그래서 실행 중에는 별도의 메모리 관리가 이루어지지 않는다.

GC는 프로그램 실행 중(Runtime)에 동적으로 감시하고 있다가, 더 이상 사용할 필요가 없다고 여겨지는 걸 소멸한다.

### ARC를 이해해야 하는 이유
기본적으로 Swift에서는 ARC를 이용해 메모리관리를 자동으로 하지만 특정상황에서는 메모리누수가 발생할 수 있기 때문에 코드 간의 관계에 대한 정보를 나타내야 한다. 
인스턴스들이 서로를 참조하거나, 클로저와 인스턴스가 참조할 때 발생하는 것이 메모리 누수의 예시이다. 이를 약한참조나 미소유참조를 사용해 메모리 누수를 해결할 수 있다.


## ✏️ map, filter, reduce 만들어보기

### map
```swift
extension Array {
    func myMap<T>(transform: (Element) -> T) -> [T] {
        var result: [T] = []
        for element in self {
            result.append(transform(element))
        }
        return result
    }
}
```
스위프트의 배열에는 Element라는 제네릭 타입이 존재한다. 이는 배열을 이루고있는 요소의 타입이다. 
`myMap`은 T라는 제네릭을 이용하기 떄문에 어떤타입으로든 변환해서 밖으로 내보낼 수 있다. 
만약 `[1, 2, 3]`이라는 배열에 대해 myMap을 만들면 다음과 같이 placeholder가 제시된다.

```swift
let arr = [1, 2, 3]
arr.myMap(transform: (Int) -> T)
```
여기서 a의 타입이 [Int]이기 때문에 transform은 Element로 설정되었기에 입력값이 Int로 변환된 것을 볼 수 있다.
클로저를 transform에 넣게되면 클로저의 반환타입이 T이기 때문에 `myMap`의 내부에서 `result.append(transform(element))`할 때 `transform(element)`가 외부에서 넣어줬던 반환타입으로 설정이 된다.

아래 코드는 클로저의 트레일링 문법을 사용한 코드이다.
```swift
let arr = [1, 2, 3]
let doubledArr = a.myMap { element in
    element * 2
}
```

### filter 

```swift
 func myFilter(transform: (Element) -> Bool) -> [Element] {
    var result: [Element] = []
    for element in self {
        if transform(element) {
            result.append(element)
        }
    }
    return result
}
```
`filter`는 `map`과 다르게 특정 타입으로 변환해서 반환해주는 메서드가 아니다. 정해진 타입에서 특정 조건에 맞는 요소만 추출해서 반환하는 메서드이기 때문에 제네릭을 사용하지 않았다.

```swift
let a = [1, 2, 3]
let evenNums = a.myFilter { element in
    element % 2 == 0
}
```

`myFilter`의 `in`뒤에 Bool타입이 나오는 조건이 있는 것을 볼 수 있다. 


### reduce
```swift
func myReduce<T>(initialResult: T, nextPartialResult: (T, Element) -> T) -> T {
    var result = initialResult
    for element in self {
        result = nextPartialResult(result, element)
    }
    return result
}
```

`reduce`는 초기값과 클로저를 받아 값을 누적시킨 뒤 그 값을 반환하는 함수이다. `reduce`도 `map`처럼 제네릭을 사용하기 때문에 초기값과 본래 `array`가 가지고있는 요소의 값이 다를 수 있다. 

```swift
let b = a.myReduce(initialResult: "") { initialResult, nextPartialResult in
    return (initialResult) + "\(nextPartialResult)" + " "
}

let c = a.myReduce(initialResult: "") { $0 + String($1) + " " }  // -----1
``` 

1번 코드에서 $0와 $1은 정의부에서 타입이 정의되어있기 때문에 생략할 수 있고 `$0 + String($1) + " "`는 return타입이다. 

### 참고자료
- [토마의 개발노트 - reduce만들어보기](https://jusung.github.io/Reduce-%ED%95%A8%EC%88%98-%EA%B5%AC%ED%98%84%ED%95%B4-%EB%B3%B4%EA%B8%B0/)
- [TCP Scholl- 메모리구조](http://www.tcpschool.com/c/c_memory_structure)
- [najin - ARC 뿌시기](https://sujinnaljin.medium.com/ios-arc-%EB%BF%8C%EC%8B%9C%EA%B8%B0-9b3e5dc23814)
- [swift org - ARC](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html)



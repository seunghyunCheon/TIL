230124 Queue Data Structure(LinkedList, Array, head, Double Stack)
===
TIL (Today I Learned)
===
학습내용
---
- Queue 학습하기
## 고민한 점 / 해결 방법
## Queue
> 뒤에서 새로운 item을 삽입하고 앞에서 삭제하는 list. (First In First Out)
> queue를 사용하는 것이 항상 최고의 선택은 아니다. 만약 리스트의 요소들이 순서에 맞게 삽입되거나 제거되어야 하는 것이 아니라면 queue대신 stack을 사용하자. 스택이 더 간단하고 빠르다.
### queue 예시
- 큐에 10이라는 숫자를 enqueue한다면 queue가 [10]이 된다.
```swift
queue.enqueue(10) // queue: [10]
```
- 3이라는 숫자를 또 enqueue한다면 queue는 [10, 3]이 된다.
```swift
queue.enqueue(3) // queue: [10, 3]
```
- queue를 dequeue한다면 처음 넣었던 숫자가 빠지게 되어 [3]이 된다.
```swift
queue.dequeue()
```
- 만약 큐가 전부 비어진 상태로 dequeue를 한다면 nil을 리턴하게 될 것이다.

### Queue 구현(With LinkedList)
> Queue 구현은 기본적으로 LinkedList를 사용하거나 List를 사용해 구현할 수 있다. 현재는 LinkedList가 구현이 되어있다고 가정한다.

#### Queue라는 구조체를 만든다.
```swift
public struct Queue {
    
}
```
#### list변수와 Enqueue함수를 만든다.
```swift
// 1
fileprivate var list = LinkedList<Int>()

// 2
public mutating func enqueue(_ element: Int) {
    list.append(element)
}
```
- 1. list는 아이템들을 저장할 큐로써 사용이 된다.
- 2. enqueue를 통해 list변수의 값들을 변화시키기 떄문에 mutating키워드를 붙여준다.

#### Dequeue함수를 만든다.
```swift
// 1
public mutating func dequeue() -> Int? {
    // 2
    guard !list.isEmpty, let element = list.first else { return nil }
    list.remove(element)
    return element.value
}
```
- 1. queue에서 첫번째 아이템을 반환하는 dequeue함수를 추가한다. 반환타입은 옵셔널로함으로써 list가 비어있을때 nil을 받을 수 있다. 이 메서드도 enqueue메서드처럼 list변수의 값을 조작하기 때문에 mutating을 붙여야 한다.
- 2. guard문을 사용해 list가 비어있을 때 바로 nil을 반환하도록 만든다.

#### Peek함수를 만든다.
```swift
public func peek() -> Int? {
    return list.first?.value
}
```
- dequeue에서 guard문을 사용한 이유는 뒤에 바인딩한 값을 사용하기 위한 이유도 존재한다. peek함수에서는 바인딩해서 값을 제거하는 책임이 존재하지 않기 떄문에 옵셔널체이닝으로 간단하게 구현했다.

#### IsEmpty 함수를 만든다.
```swift
public var isEmpty: Bool {
    return list.isEmpty   
}
```
- LinkedList기반의 list변수를 통해 큐의 값 유무를 체크한다.

#### Queue 출력
```swift
var queue = Queue()
queue.enqueue(10)
queue.enqueue(3)
queue.enqueue(57)

print(queue) // output: Queue
```
- queue를 만들고 print를 하면 값을 확인할 수 없는 Queue라는 모호한 값이 출력이 된다. 이에 대해 조금 더 명확한 output문자열을 보여주기 위해 Queue에 CustomStringConvertible 프로토콜을 채택해서 사용할 수 있다.
```swift
// 1
extension Queue: CustomStringConvertible {
    // 2
    public var description: String {
        return list.description
    }
}
```
- 1. CustomStringConvertible을 채택한다. 이 프로토콜은 직접 description이라는 String타입의 계산 속성을 정의하도록 요구한다.
- 2. 읽기전용 프로퍼티인 description을 선언한다.
- 3. LinkedList에 기반한 list변수의 description을 리턴한다.

이제 queue를 출력하면 다음처럼 보여지게 된다.
```swift
print(queue) // output: "[10, 3, 57]"
```
### 제네릭 큐 구현
> 제네릭을 이용해서 여러가지 타입에 대응해서 queue를 구현할 수 있다. 
```swift
// 1
public struct Queue<T> {

  // 2
  fileprivate var list = LinkedList<T>()

  public var isEmpty: Bool {
    return list.isEmpty
  }
  
  // 3
  public mutating func enqueue(_ element: T) {
    list.append(element)
  }

  // 4
  public mutating func dequeue() -> T? {
    guard !list.isEmpty, let element = list.first else { return nil }

    list.remove(element)

    return element.value
  }

  // 5
  public func peek() -> T? {
    return list.first?.value
  }
}
```
- 1. 제네릭 타입T를 이용해 Queue를 선언한다.
- 2. 목적은 다양한 타입을 포용하는 Queue를 선언하는 것이기 때문에 내부의 list프로퍼티도 제네릭하게 만든다. 
- 3, 4, 5: 각 메서드들은 다양한 타입을 포용하는 방식으로 동작한다.

### Queue구현(with Array)

#### 배열을 이용해 만든 큐
```swift
public struct Queue<T> {
  fileprivate var array = [T]()

  public var isEmpty: Bool {
    return array.isEmpty
  }
  
  public var count: Int {
    return array.count
  }

  public mutating func enqueue(_ element: T) {
    array.append(element)
  }
  
  public mutating func dequeue() -> T? {
    if isEmpty {
      return nil
    } else {
      return array.removeFirst()
    }
  }
  
  public var front: T? {
    return array.first
  }
}
```
- 위 큐는 잘 동작하지만 최적화되지는 않았다. enqueuing은 배열에 크기와 상관없이 배열끝에 추가만 하면 되기 때문에 O(1)의 시간복잡도를 가진다.

#### 그럼 어째서 요소추가가 O(1)의 시간복잡도만 가지는 걸까?
- 결론부터 말하자면 항상 O(1)은 아니다. Swift에서 Array의 append는 다음의 과정으로 요소가 추가가 된다.

만약 아래의 코드처럼 배열을 만들게 된다면
```swift
["brody", "goatt", "christy"]
```
실제로는 배열에 들어가있는 요소만큼 xxx가 추가가 되어있다. 이는 예약은 되어있지만 아직 채워지지는 않은 상태이다.
```swift
["brody", "goatt", "christy", "xxx", "xxx", "xxx"]
```
이 상태에서 `rowan`이라는 요소를 추가하게 된다면 다음의 형태를 갖게 된다.
```swift
["brody", "goatt", "christy", "rowan", "xxx", "xxx"]
```
- 이는 어떤 장소에서 다른 장소로 메모리를 복사하는 것이기 때문에 constant-time 작업시간이 걸린다.
- 만약 모든 요소들이 채워진 상태라면 앞의 메모리들 개수만큼 다시 재할당 해야 하기 떄문에 O(N)시간복잡도를 갖게 된다.

dequeue메서드의 경우는 다르다. 첫번째 요소를 제거한다면 남아있는 요소들을 앞으로 당겨야하기 때문에 O(N)시간복잡도가 들게 된다.
```
before   [ "Brody", "goatt", "christy", "rowan", xxx, xxx ]
                     /       /          /
                    /       /          /
                   /       /          /
                  /       /          /
 after   [ "goatt", "christy", "rowan", xxx, xxx, xxx ]
```
- 이러한 이유로 enqueue는 효율적이지만 dequeue는 효율적이라고 말하기가 애매해졌다.

### 효율적인 큐
효율적인 dequeuing을 하기위해 enqueue처럼 충분한 space를 예약해야겠다라는 아이디어를 생각할 수 있지만 built-in Swift array에서는 이를 지원하고 있지 않기 때문에 사용할 수 없다.

생각할 수 있는 main idea는 우리가 item을 dequeuing할 때마다 array내용들을 전부 이동하지 않고 아이템의 위치에 표시를 해두는 것이다. 위의 예시로 만약 Brody요소가 사라졌다면 다음의 그림으로 표시가 되는 것이다.
```swift
[ xxx, "goatt", "christy", "rowan", xxx, xxx]
```
goatt요소도 지운다면 
```swift
[ xxx, xxx, "christy", "rowan", xxx, xxx ]
```
앞의 비어진 위치들을 이제 재사용되지 않을 것이기 때문에 나머지 요소를 앞으로 이동해 array를 정기적으로 지울 수 있다. 이 trimming절차는 O(n)시간복잡도를 가지는데 한 번만 발생하기 때문에 dequeuing은 평균적으로 O(1)이 된다.

먼저 전체적으로 효율적인 큐의 코드를 살펴보자
```swift
public struct Queue<T> {
  fileprivate var array = [T?]()
  fileprivate var head = 0
  
  public var isEmpty: Bool {
    return count == 0
  }

  public var count: Int {
    return array.count - head
  }
  
  public mutating func enqueue(_ element: T) {
    array.append(element)
  }
  
  public mutating func dequeue() -> T? {
    guard head < array.count, let element = array[head] else { return nil }

    array[head] = nil
    head += 1

    let percentage = Double(head)/Double(array.count)
    if array.count > 50 && percentage > 0.25 {
      array.removeFirst(head)
      head = 0
    }
    
    return element
  }
  
  public var front: T? {
    if isEmpty {
      return nil
    } else {
      return array[head]
    }
  }
}
```

array는 이제 비어있는 요소에 대해 표시할 필요가 있기 때문에 T타입이 아닌 T?타입으로 선언이 된다. head변수는 맨 앞 객체의 인덱스이다.

효율적인 큐는 dequeuing을 개선하는데 사용되었다는 것에 집중하자. 만약 한 요소를 지운다면 먼저 `array[head]`를 nil로 바꿔주어 그 객체를 array에서 지운다. 이후 다음 객체가 첫번째 요소임을 가리키기 위해 head를 증가시킨다.

```swift
[ "brody", "goatt", "christy", "rowan", xxx, xxx ]
   head
```
`dequeue brody`
```swift
[ xxx, "goatt", "christy", "rowan", xxx, xxx ]
         head
```

이건 마치 슈퍼마켓에서 계산대에 있는 사람들이 계산대를 향해 앞으로 가지 않고 계산대가 줄 위로 이동하는 것과 같다.

만약 앞쪽의 빈 부분을 제거하지 않으면 요소를 enqueue하거나 dequeue할 때 array가 계속 증가한다. array를 정기적으로 trimming하기 위해 다음을 적용한다.
```swift
public mutating func dequeue() -> T? {
    guard head < array.count, let element = array[head] else { return nil }

    array[head] = nil
    head += 1

    let percentage = Double(head)/Double(array.count)
    if array.count > 50 && percentage > 0.25 {
      array.removeFirst(head)
      head = 0
    }
    
    return element
}
```
- array의 개수가 50보다 많으면서 percentage가 1/4보다 많아졌을때 앞쪽의 nil을 제거해준다. 이 경우에만 removeFirst로 앞에있는 nil값을 제거함으로써 평균적인 dequeuing은 O(1)을 기대할 수 있게 된다.
- 아래 코드는 `removeFirst(k: Int)`의 코드이다.
```swift
  public mutating func removeFirst(_ k: Int) {
    if k == 0 { return }
    _precondition(k >= 0, "Number of elements to remove should be non-negative")
    guard let end = index(startIndex, offsetBy: k, limitedBy: endIndex) else {
      _preconditionFailure(
        "Can't remove more items from a collection than it has")
    }
    removeSubrange(startIndex..<end)
  }
```
- `let end = index(startIndex, offsetBy: k, limitedBy: endIndex)`를 통해 첫번째 인덱스에서 k번째 떨어진 부분부터 endIndex까지의 범위를 end라는 변수에 담고, `removeSubrange(startIndex..<end)`를 통해 지정한 범위까지 제거하는 함수를 호출한다. 


### Queue 구현(With Double Stack)
> enqueue스택과 dequeue스택 두 개를 만들어 삽입, 삭제 시간복잡도를 O(1)로 만들어주는 방법

```swift
struct QueueStack<T>: Queue {
  private var dequeueStack: [T] = []
  private var enqueueStack: [T] = []
  
  var isEmpty: Bool {
    return dequeueStack.isEmpty && enqueueStack.isEmpty
  }
  var peek: T? {
    return !dequeueStack.isEmpty ? dequeueStack.last : enqueueStack.first
  }
  
  mutating func enqueue(_ element: T) {
    enqueueStack.append(element)
  }
  
  @discardableResult
  mutating func dequeue() -> T? {
    if dequeueStack.isEmpty {
      dequeueStack = enqueueStack.reversed()
      enqueueStack.removeAll()
    }
    return dequeueStack.popLast()
  }
}
```
- dequeue할 때 dequeue가 비어있다면 기존의 enqueue를 reversed한 배열을 dequeueStack에 담고 popLast를 통해 O(1)의 시간복잡도로 마지막 요소를 제거한다. (reverse의 시간복잡도는 O(n)이지만 reversed는 O(1)이다.)
- dequeue할 때 dequeue 스택이 비어있지않다면 시간복잡도가 O(1)인 popLast()를 바로 호출해 마지막 요소를 반환한다.

#### 두 개의 스택사이에서 reversed를 이용해 시간복잡도를 줄인 케이스다.

### Reference
- [kodecocodes git - queue](https://github.com/kodecocodes/swift-algorithm-club/tree/master/Queue)
- [kodeco - queue](https://www.kodeco.com/848-swift-algorithm-club-swift-queue-data-structure)
- [Apple git - RangeRelaceableCollection](https://github.com/apple/swift/blob/main/stdlib/public/core/RangeReplaceableCollection.swift)
- [개발자 아라찌 - Swift로 구현한 Queue와 더블스택](https://apple-apeach.tistory.com/8)

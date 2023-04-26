# 2304023 Animate, animateKeyframes

학습내용
- Animate
- animateKeyframes

## Animate
애니메이션은 명시된 기간, 딜레이, 옵션, completion 핸들러를 이용해서 여러개의 뷰를 변화시킨다.

```swift
class func animate(
    withDuration duration: TimeInterval,
    delay: TimeInterval,
    options: UIView.AnimationOptions = [],
    animations: @excaping () -> Void,
    completion: ((Bool) -> Void)? = nil
)
```

### Paramaters

- duration: 초단위로 측정되는 애니메이션 시간. 만약 0이하로 명시하면 애니메이션이 적용되지않는다.
- delay: 애니메이션을 시작하기전에 기다리는 시간. 즉시 시작하려면 0을 명시
- options: 어떻게 애니메이션을 수행할지 명시하는 옵션.
- animations: 뷰에 커밋할 변경사항을 포함하는 블럭객체. 뷰의 변경가능한 속성을 코드로 변경시키는 공간이다. 이 블록은 파라미터도 리턴값도 없다.
- completion: 애니메이션 순서가 끝났을때 실행되는 블럭객체. 반환값이 없고 단일 bool 인자를 가진다. 이는 completion 핸들러가 호출되기전에 실제로 애니메이션을 끝났는지 아닌지를 알려주는 값이다. 만약 애니메이션 기간이 0이라면 이 블록은 다음 런루프 사이클 시작에 수행된다. 


### Discussion
이 메서드는 뷰에서 수행되는 애니메이션 집합을 초기화한다.`animations` 파라미터안에 블록객체는 하나이상의 뷰들의 프로퍼티가 애니메이팅하는 코드를 포함한다.

애니메이션 동안에 사용자 인터렉션은 일시적으로 비활성화된다. 만약 사용자가 뷰와 소통할 수 있도록 하려면 `options`파라미터 안에 `allowUserInteraction`상수를 포함시킨다.

## animateKeyframes

keyframe기반의 애니메이션을 설정하는데 사용되는 애니메이션 블록객체를 만든다.

```swift
class func animateKeyframes(
    withDuration duration: TimeInterval,
    delay: TimeInterval,
    options: UIView.KeyframeAnimationOptions = [],
    animations: @excaping () -> Void,
    completion: ((Bool) -> Void)? = nil
)
```

### Discussion

keyFrame은 주로 동영상 작업에서 사용하는 용어로, 시작 프레임과 끝 프레임 중에 전체 정보를 가지고있는 중심 프레임을 키프레임이라고 부른다. 

### Concurrency Note
completion handler를 이용해서 동기 코드에서 호출할 수 있다. 비동적으로도 하고싶다면 다음 자료를 참고한다.[Calling Objective-C APIs Asynchronously](https://developer.apple.com/documentation/swift/calling-objective-c-apis-asynchronously)

이 메서드는 keyframe기반의 애니메이션을 설정할 수 있는 메서드이다. keyframe 자체로는 초기 애니메이션 블럭이아니다. `addKeyframe(withRelativeStartTime:relativeDuration:animations)`메서드를 호출함으로써 `animations` 블록 안에서 keyframe 시간과 애니메이션 데이터를 추가해야 한다. 

keyframe을 추가하는 것은 뷰의 현재 값에서 첫번째 keyframe값으로 애니메이션하게 만들고 이후 다음 keyframe으로 애니메이션하게 만든다. 

만약 `animations` 블럭안에 keyframe을 추가하지않는다면 그 애니메이션은 표준 애니메이션 블럭처럼 처음부터 끝까지 진행된다.


### 활용법

`keyframe`을 이용하면 completion Handler를 연달아서 사용하지 않아도 되기 때문에 콜백지옥을 막을 수 있다.

```swift
UIView.animateKeyframes(withDuration: 3, delay: 0, animations: {

        UIView.addKeyframe(withRelativeStartTime: 0, relativeDuration: 1.0/3.0, animations: {

         // 1번째 애니메이션

        })

        UIView.addKeyframe(withRelativeStartTime: 1/3, relativeDuration: 1.0/3.0, animations: {

         // 2번째 애니메이션

        })

        UIView.addKeyframe(withRelativeStartTime: 2/3, relativeDuration: 1.0/3.0, animations: {

          //3번째 애니메이션 

        })

    })
})
```

### 참고문서
- [Apple Docs - animate](https://developer.apple.com/documentation/uikit/uiview/1622451-animate)
- [Apple Docs - animateKeyframes](https://developer.apple.com/documentation/uikit/uiview/1622552-animatekeyframes)
- [animateKeyframes 사용해서 애니메이션 중첩해보기](https://i-colours-u.tistory.com/4)

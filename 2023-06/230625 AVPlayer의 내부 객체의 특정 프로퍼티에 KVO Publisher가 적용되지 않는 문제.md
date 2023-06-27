# 230625 AVPlayer의 내부 객체의 특정 프로퍼티에 KVO Publisher가 적용되지 않는 문제
### 학습내용
- AVPlayer의 내부 객체의 특정 프로퍼티에 KVO Publisher가 적용되지 않는 문제

## AVPlayer의 내부 객체의 특정 프로퍼티에 KVO Publisher가 적용되지 않는 문제
AVPlayer의 currentItem의 Duration속성의 seconds속성에 대해 Publisher를 만들기 위해 다음의 코드를 작성했다.

```swift
guard let videoItem = player.currentItem else {
        error = VideoPlayerViewModelError.failedToDurationError
        return
    }

videoItem.publisher(for: \.duration.seconds)
.sink {
    //
}
```

하지만 위 코드를 작성하니 다음의 에러가 나오면서 앱이 크래시났다.
 > Could not extract a String from KeyPath Swift.KeyPath

이런 에러가 발생한 이유는 KVO는 Objective C의 기능이기 때문에 `@objc dynamic`을 붙여줘야 하는 것이다. `AVPlayerItem`의 `duration`에는 해당 prefix가 존재하겠지만 seconds에는 존재하지 않기 떄문이었다. [Apple Docs - KVO Observing](https://developer.apple.com/documentation/swift/using-key-value-observing-in-swift#Annotate-a-Property-for-Key-Value-Observing)

AVPlayerItem은 애플에서 제공하는 객체이기 때문에 직접 프로퍼티에 KVO prefix를 붙일 수 없었기 때문에 다음과 같이 변경했다. 

```swift
guard let videoItem = player.currentItem else {
    error = VideoPlayerViewModelError.failedToDurationError
    return
}

videoItem.publisher(for: \.duration)
    .sink { [weak self] duration in
        guard duration.seconds > 0.0 else { return }
        self?.duration = self?.getTimeString(from: videoItem.duration)
    }
    .store(in: &cancellables)
```
### 참고문서
- [Key-Value Observing in the age of Swift](https://medium.com/nerd-for-tech/key-value-observing-in-the-age-of-swift-f509c54fa5e8)
- [Apple Docs - Performing Key-Value Observing with Combine](https://developer.apple.com/documentation/combine/performing-key-value-observing-with-combine)

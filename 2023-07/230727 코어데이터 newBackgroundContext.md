## 230727 코어데이터 newBackgroundContext

## newBackgroundContext()
> private queue에서 실행하는 새로운 관리객체 컨텍스트를 반환

CoreData를 사용하면서 context는 항상 main Context인 viewContext만 사용했었다. 하지만 메인 컨텍스트는 메인 큐를 이용하기 때문에 로드하는 시간이 큰 작업이라면 뚝뚝 끊길 수 있다. 이 경우에 main 큐가 아닌 새로운 큐를 만들어 이곳에서 데이터 처리를 할 수 있다.

즉 사용자의 반응성을 위해 백그라운드 큐와 컨텍스트를 생성해야 하는데 그 메서드가 `newBackgroundContext`인 것이다.
이렇게 생성한 컨텍스트는 `perform`을 같이 사용해줘야 비동기적으로 코드 Block을 실행할 수 있다.

그리고 해당 컨텍스트가 최신상태를 유지하기 위해 `automaticallyMergesChangesFromParent`를 `true`로 변경하면 된다. 


### 사용예시
```swift
let context = persistentContainer.newBackgroundContext()
context.automaticallyMergesChangesFromParent = true

// Perform operations on the background context
// asynchronously
context.perform {
    do {
        // Modify the background context
        // ...
        
        // Save the background context
        try context.save()
    }
    catch let error {
    }
}
```
### 참고문서
- [Apple Docs - automaticallyMergesChangesFromParent](https://developer.apple.com/documentation/coredata/nsmanagedobjectcontext/1845237-automaticallymergeschangesfrompa)
- [Apple Docs - Using Core Data in the background](https://developer.apple.com/documentation/coredata/using_core_data_in_the_background)
- [Apple Docs - newBackgroundContext](https://developer.apple.com/documentation/coredata/nspersistentcontainer/1640581-newbackgroundcontext)
- [Core Data Background Fetch, Save, & Create](https://www.advancedswift.com/core-data-background-fetch-save-create/)
- [newBackgroundContext](https://eunjin3786.tistory.com/154)

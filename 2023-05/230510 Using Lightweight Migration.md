# 230510 Using Lightweight Migration

### 학습내용
- Using Lightweight Migration

## Using Lightweight Migration
앱 내의 모델 변경사항과 매치하기 위해 경량의 마이그레이션을 요청한다.

### Overview
코어 데이터는 일반적으로 경량의 마이그레이션을 가리켜서 자동 데이터 마이그레이션을 수행한다. 경량 마이그레이션은 source와 destination 관리객체 모델 사이의 차이점으로부터 마이그레이션을 추론한다.

### 추론된 매핑 모델을 생성

자동경량 마이그레이션을 수행하기위해 코어데이터는 런타임에 source와 destination 관리객체 모델을 찾을 수 있어야 한다. 그것은 `Bundle`클래스의 `allBundles`와 `allFrameworks`메서드에 의해 번들 안에 모델을 찾는다.
그 후 코어데이터는 persistent 엔티티와 프로퍼티에 대한 스키마 변경사항을 분석한다. 그리고 추론된 매핑모델을 생성한다.

추론된 매핑 모델을 생성하는 것은 명확한 마이그레이션 패턴에 맞게 변경해야 한다.
- 속성 추가
- 속성 제거
- non옵셔널이 옵셔널로
- 옵셔널이 non옵셔널로, 기본 값을 가진다
- entity, property의 rename

### 엔티티와 프로퍼티 변화를 관리
엔터티 또는 속성의 이름을 바꾸는 경우 대상 모델의 이름 변경 식별자를 소스 모델의 해당 속성 또는 엔터티의 이름으로 설정할 수 있다. Xcode Data Modeling tool's inspector를 사용하여 managed object model안의 renaming identifier를 설정한다. 예를들어 다음과 같이 한다.
- `Car` 엔티티를 `Automobile`로 이름을 변경
- `Car`의 `color` 속성을 `paintColor`로 이름을 변경

renaming identifier는 정식 이름을 만든다. 그래서 renaming identifier는 source model안의 프로퍼티 이름으로 설정된다. (이미 존재하는 것이 아닌 이상)
이는 version 2에서 프로퍼티를 rename한 후에 version 3에서 다시 rename할 수 있다는 것을 의미한다. 이름 바꾸기는 버전 2에서 버전 3으로 또는 버전 1에서 버전 3으로 올바르게 작동한다.

### 관계 변화를 관리
경량 마이그레이션은 관계변화를 관리할 수 있다. 새로운 관계를 추가하거나 기존의 관계에서 삭제할 수도있다. 또한 renaming identifier를 사용하여 관계를 리네임할수도 있다.

추가적으로 1:1에서 1:N으로 변경할 수도 있다.

### 계층변화를 관리
계층안에서 엔티티를 추가ㅏ, 삭제, 리네임할 수 있다. 또한 새로운 부모나 자식 엔티티를 만들고 프로퍼티를 계층에서 이동시킬 수도 있다. 엔티티를 계층 밖으로 내보내는 것도 가능하다. 하지만 엔티티 계층을 병합시킬 수는 없다. 만약 두 개의 존재하는 엔티티가 소스 내에서 공동의 부모를 가지고 있지않는다면 그것들은 목적지안에서 공동의 부모를 공유할 수 없다. 

### 코어데이터가 모델을 추론할 수 있는지를 확인
Core Data가 실제로 마이그레이션 작업을 수행하지 않고 원본 모델과 대상 모델 간의 매핑 모델을 유추할 수 있는지 여부를 미리 확인하려는 경우 `NSmappingModel`의 `inferredMappingModel(forSourceModel:destinationModel:)`메서드를 사용한다. 
이 메서드는 만약 코어데이터가 그것을 만들 수 있다면 추론된 모델을 반환하고 아니면 `nil`을 반환한다.

만약 데이터변경이 자동 마이그레이션의 수용치를 초과했다면 heavyweight migration을 해야 한다.(manual migration이라고도 불림)

### 경량 마이그레이션 요청
딕셔너리 옵션을 사용하여 경량 마이그레이션을 요청한다. 이는 `addPersistentStore(ofType:configurationName:at:options:)`에 전달한다. `NSMigratePersistentSotresAutomaticallyOption`과 `NSInferMappingModelAutomaticallyOption`키를 `true`로 만들어서 값을 설정한다.

```swift
let psc = NSPersistentStoreCoordinator(managedObjectModel: mom)
let options = [NSMigratePersistentStoresAutomaticallyOption: true, NSInferMappingModelAutomaticallyOption: true]
do {
    try psc.addPersistentStore(ofType: NSSQLiteStoreType, configurationName: nil, at: storeURL, options: options)
} catch {
    fatalError("Failed to add persistent store: \(error)")
}
```

이러한 설정을 사용하면 Core Data는 영구 저장소가 더 이상 현재 모델과 일치하지 않음을 감지하면 경량 마이그레이션을 시도한다.

### 생각한 점
Editor에서 새로운 모델버전을 만들고, 이곳에서 변경하고 싶은 프로퍼티, 엔티티의 이름을 변경하고 모델을 변경한 뒤에 바로 위의 딕셔너리 옵션값을 사용해주면 자동으로 마이그레이션 하는 건가 싶었다. 자세한 건 실제로 마이그레이션하면서 알아봐야겠다.

### 참고문서
- [Using Lightweight Migration](https://developer.apple.com/documentation/coredata/using_lightweight_migration)

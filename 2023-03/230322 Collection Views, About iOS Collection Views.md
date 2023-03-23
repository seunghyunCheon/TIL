230322 Collection Views, About iOS Collection Views
===
학습내용
--- 작업을 처리하는 이유
- Collection views


## Collection views
수정가능하고 커스텀가능한 레이아웃을 사용해 중첩된 뷰를 보여준다.

### 개요
컬렉션 뷰는 포토앱의 grid 사진들처럼 순서가 있는 내용을 관리한다.

컬렉션 뷰는 많은 객체들의 콜라보다.
- Cells. 하나의 셀은 하나의 내용을 표현하는 시각적인 표현이다.
- Layout. 레이아웃은 컬렉션 뷰 내에 내용을 배치를 정의한다.
- Data source object. 이 객체는 `UICollectionViewDataSource` 프로토콜을 채택하고 있고 컬렉션뷰에 데이터를 제공한다.
- delegate object. 이 객체는 `UICollectionViewDelegate` 프로토콜을 따르고 있고 선택과 강조와 같은 유저 상호작용을 관리한다.
- Collection View Controller. `UICollectionViewController`를 사용하면 컬렉션과 관련된 작업들이 구현되어있다.

## About iOS Collection Views

컬렉션뷰는 유연하고 변경가능한 레이아웃을 사용하여 데이터들을 보여주는 방법이다. 컬렉션 뷰의 가장 흔한 사용은 grid처럼 보여지는 배치이다. 하지만 iOS에서 컬렉션 뷰는 row와 column을 더 많이 가질 수 있다. 시각적인 요소들의 정밀한 레이아웃은 subclass해서 정의가 가능하고 동적으로 변경될 수 있다. 그래서 grid, stack, circular layout을 만들 수 있고 동적으로 변화도 가능하다.

컬렉션뷰는 보여지는 데이터와 데이터를 보여주는 요소 사이를 엄격하게 분리한다. 대부분의 경우에 앱은 혼자서 데이터를 관리한다. 앱에서 데이터를 표현하는 뷰 객체를 제공한 후에 컬렉션 뷰는 화면에 위치를 배치시키는 일을 하는 것이다. 이 일은 layout 객체와 같이 동작한다. 이는 배치와 시각적인 속성을 명시하고 앱의 니즈에 맞게 subclassing할 수도 있다. 그러므로 앱에서 데이터를 제공하고, 레이아웃 객체는 배치에대한 정보를 주고 컬렉션뷰는 이 두개를 합치는 것이다.

### 컬렉션뷰는 데이터 기반 뷰의 시각적인 표현을 관리한다.
컬렉션뷰는 앱에서 제공되는 데이터기반 뷰의 표현을 용이하게 한다. 컬렉션뷰의 유일한 관심사는 뷰를 가지고 특별한 방법으로 그것들을 그리는 것이다. 컬렉션뷰는 뷰의 배치와 표현에 대한 것이고 내용에 대한 것이 아니다. 컬렉션 뷰, data source, 레이아웃 객체, 커스텀 객체 사이에서 상호작용하는 것은 효율적인 방법으로 컬렉션 뷰를 사용하는데 있어 매우 중요하다.

### Flow Layout은 Grid와 다른 선지향의 표현을 지원한다.
flow layout 객체는 UIKit에서 제공하는 구체적인 레이아웃이다. 일반적으로 grid를 구현하는데 사용한다.(행과 열로 아이템들이 있는) 그런데 flow layout은 여러 종류의 선형 flow들을 지원한다. 이는 grid가 아니기 때문에 flow layout을 사용해 더 유연한 배치를 가지는 레이아웃을 만들 수 있다. 이 작업은 어떠한 서브클래싱 없이 가능하다. flow layout은 다른 사이즈의 아이템들을 지원하고, 다양한 간격, 커스텀 헤더와 푸터 등을 지원한다. 그리고 서브클래싱을 사용하면 flow layout 클래스의 동작을 더욱 미세하게 조정할 수 있게 된다.

### Gesture Recognizer는 셀과 레이아웃 조작에 사용된다.
모든 뷰에 gesture recognizer를 부착해 view의 내용을 조작할 수 있다. 

### Custom Layout은 grid를 뛰어넘게 만든다.
기본 레이아웃 객체를 subclass해서 커스텀 레이아웃을 구현할 수 있다. 비록 커스텀 레이아웃을 설계하는 것이 일반적으로 많은 양의 코드를 요구하지는 않아도 레이아웃 동작 방식을 많이 알수록 효율적으로 디자인할 수 있다.

- [Apple Docs - UICollectionView](https://developer.apple.com/documentation/uikit/uicollectionview)
- [Apple git - URLSession](https://github.com/apple/swift-corelibs-foundation/blob/main/Sources/FoundationNetworking/URLSession/URLSession.swift)

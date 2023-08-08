# 230808 FlowLayout layoutAttributesForElements(in rect: CGRect), layoutAttributesForItem(at indexPath: IndexPath)

달력을 구현하다가 커스텀한 레이아웃을 만들기 위해 `UICollectionViewFlowLayout`의 서브클래스를 정의하면서 해당 메서드를 찾아봤다. 

## FlowLayout layoutAttributesForElements(in rect: CGRect)
> 지정된 사각형의 모든 셀과 뷰에 대한 레이아웃 속성을 반환한다.

```swift
func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]?
```

셀과 뷰에 대한 레이아웃 정보를 표현하는 `UICollectionViewLayoutAttributes`객체 배열. 기본값은 nil이다.

서브클래스는 이 메서드를 재정의해야하고 레이아웃 정보를 반환해야 한다. 레이아웃 정보에는 셀과 supplementary 뷰, 데코레이션 뷰를 비롯하여 시각적인 정보를 의미한다.

## 프로젝트 적용
```swift
override open func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
        
    return super.layoutAttributesForElements(in: rect)?.map { attrs in
        let attrscp = attrs.copy() as! UICollectionViewLayoutAttributes
        self.applyLayoutAttributes(attrscp)
        return attrscp
    }
}

func applyLayoutAttributes(_ attributes : UICollectionViewLayoutAttributes) {
    guard attributes.representedElementKind == nil else { return }

    guard let collectionView = self.collectionView else { return }

    var xCellOffset = CGFloat(attributes.indexPath.item % 7) * self.itemSize.width
    var yCellOffset = CGFloat(attributes.indexPath.item / 7) * self.itemSize.height

    let offset = CGFloat(attributes.indexPath.section)

    switch self.scrollDirection {
    case .horizontal:   xCellOffset += offset * collectionView.frame.size.width
    case .vertical:     yCellOffset += offset * collectionView.frame.size.height
    @unknown default:
        fatalError()
    }

    // set frame
    attributes.frame = CGRect(
        x: xCellOffset,
        y: yCellOffset,
        width: self.itemSize.width,
        height: self.itemSize.height
    )
}
```

위 코드에서 attribute는 immutable 객체이기 때문에 값을 직접 변경할 수 없기 떄문에 `copy`메서드를 사용한다. 

## layoutAttributesForItem(at indexPath: IndexPath)
> 지정된 사각형의 모든 셀과 뷰에 대한 레이아웃 속성을 반환한다.

이 메서드는 특정 indexPath에 특별한 애니메이션이나 offSet을 주고싶을 때 사용하는 것 같다. supplementaryView나 decoration view에는 사용할 수 없다.



### 참고문서
-[layoutAttributesForElements](https://developer.apple.com/documentation/uikit/uicollectionviewlayout/1617769-layoutattributesforelements)
-[layoutAttributesForItem(at:)](https://developer.apple.com/documentation/uikit/uicollectionviewlayout/1617797-layoutattributesforitem)

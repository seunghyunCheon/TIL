230110 Auto layout(priority, content Hugging, Compression Resistance Priority)
===
TIL (Today I Learned)
===
학습내용
---
- Auto layout 학습하기

## 고민한 점 / 해결 방법
## Auto layout
> 오토 레이아웃은 뷰에 지정된 제약 조건에 따라 뷰 계층에 있는 모든 뷰의 크기와 위치를 동적으로 계산한다
- 버튼과 이미지가 존재하는 경우 버튼의 상단을 이미지 하단보다 8포인트 밑으로 제약사항을 걸 때 이미지 크기에 따라 버튼이 자동으로 조정되게 할 수 있다.

#### Auto Layout의 효과
- 외부환경과 내부환경에 대해 동적으로 반응한다.
- 외부환경은 superview자체의 모양(디바이스)이나 크기를 말하며 내부환경은 유저 인터페이스 내부의 content를 말한다.
- 외부환경변화의 예시로는 디바이스의 종류, 디바이스 회전, active call, 다른 size의 클래스가 있다.
- 내부환경변화의 예시로는 앱에 의해 표시되는 컨텐츠의 변화, 국제화 지원, Dynamic Type지원이 있다.

#### Creating Nonambiguous, Satisfiable Layout
- 오토레이아웃을 사용할 때 목표는 일련의 equation을 이용해  오직 한 가지 솔루션을 갖는 것이다. 모호한 제약사항은 한가지보다 더 많은 솔루션을 갖게 된다.

<img src="https://i.imgur.com/Y8dxhKp.png" width="500"/><br>

- 위 세개의 경우 전부 다른 제약사항을 줬지만 같은 모습을 띈다.
- 하지만 세 가지 모두 외부 컨텐츠나 내부 컨텐츠의 요구사항이 바뀌었을 때 제약사항이 전부 다르다. 즉 구현하고자하는 특정 레이아웃에 가장 적합한 방법을 적용 시켜야한다는 것이다.
- 예를 들어 첫번째 방식의 경우 superview의 width가 변경된다면 하위view의 width는 그대로이기 때문에 superview의 trailing과 하위view의 trailing사이의 거리가 굉장히 클 것이다. 
- 두 번째 방식과 세 번째 방식은 좀 더 동적으로 설정할 수 있지만 요소들을 가운데 배치하려는 경우에는 세 번째 방식이 유용하다.

#### Constant Priorities
- 기본적으로 모든 제약사항은 필수이다. 오토레이아웃은 모든 제약사항을 만족하는 하나의 솔루션을 계산하고 그렇지 않다면 에러를 발생시킨다. 그리고 만족하지 못하는 제약사항들에 대해 정보를 알려주고 break되었던 제약사항 중 하나를 선택해 재계산한다.
- 모든 제약사항들은 1에서 1000사이의 priority를 가진다. 1000은 반드시 적용되고 나머지는 optional Contraint가 된다.
- 우선순위가 높은 순서대로 적용이 되며 optional contraint가 만족되지 않는다면 skip을 해서 적용되지만, 만약 레이아웃을 그리는데 모호하다면 이를 가져와 가장 가까운 솔루션을 선택한다.
> priority가 1000이면 제일 높고, 750(high), 500(medium), 250(low) 순으로 낮다. 

#### Priority 적용
- 만약 저장버튼아이콘의 trailing속성을 50과 100 두 개를 준다면 다음과 같은 에러를 맞게 된다.<br>
<img src="https://i.imgur.com/VW9q2VK.png" width="300"/>
- 이는 두 개의 제약사항이 있기 때문에 하나만 선택을 해줘야 정상적으로 작동한다. trailing으로 50을 주고싶다면 100을 준 제약사항의 우선순위를 낮추면 된다. 그리고 반대로 하고 싶다면 50을 준 제약사항의 우선순위를 낮추면 된다. (기존의 1000이었던 걸 999로 낮춤. 실제로는 250, 500, 750, 1000을 사용하는 게 좋다고 한다)<br>
<img src="https://i.imgur.com/CVOHNfr.png" width="300"/>


#### Conetent Hugging Vs Compression Resistance Priority
> 두 오브젝트 중에 하나가 작아져야하는 상황이거나 커져야 하는 상황일 때 어떤 오브젝트가 intrinsic Size를 유지해야 하는지 모르기 때문에 사용하는 속성. intrinsic Size를 유지하고 싶은 객체에게 높은 우선순위값을 주면 된다.

- 01 두 개의 레이블을 라이브러리에서 불러와 왼쪽 레이블은 leading만, 오른쪽 레이블은 trailing만 준다.<br>
<img src="https://i.imgur.com/BFCkrYe.png" width="500"/>
- 02 왼쪽 레이블은 trailing을 지정해주고 오른쪽 레이블은 leading을 지정해줘야 한다고 warning이 나온다.
- 03 오른쪽레이블의 leading을 왼쪽 레이블의 trailing + 8 만큼 설정한다.
- 04 그럼 다음의 에러를 만나게 된다. 이는 레이블 끼리 서로 leading / trailing을 설정한 경우 둘 중 하나는 Intrinsic Width을 유지하지 못하기 때문이다.<br>
<img src="https://i.imgur.com/GVOzYVu.png" width="400"/>
- 05 따라서 두 개의 객체 중에 하나가 커져야하는 상황일 때 intrinsic Size를 유지하고 싶은 객체에 더 높은 Hugging priority를 부여하면 된다.
- 06 01단계에서의 priority체크에 Hugging Priority를 낮춘다면 다음의 사진처럼 보이게 된다.<br>
<img src="https://i.imgur.com/ofHOnpj.png" width="500"/>
- 07 priority체크레이블의 텍스트가 엄청 길어져서 priority체크2 레이블을 침범하게 된다면 다음과 사진처럼 보이게 된다.<br>
<img src="https://i.imgur.com/HBmxLRb.png" width="500"/>
- 08 둘 중 하나가 커져야하는 경우는 hugging을 통해 우선순위를 결정해줬지만 둘 중 하나가 작아져야하는 경우는 설정해주지 않았다. 이때 사용하는 priority가 Compression Resistance Priority다. priority체크2는 영역에 침범되지 않아야하므로 compression에 대해 우선순위를 높게 준다면 영역이 가려지지않고 잘 보일 것이다.<br>
<img src="https://i.imgur.com/nVxBV0n.png" width="500"/>


#### 결론
- 오토레이아웃의 개념을 알고 적용하는 것은 꽤 간단해보일지 몰라도 특정 레이아웃에 맞게 올바른 제약사항을 주는 것을 어려운 일인 것 같다. 디바이스에 따라 요소들의 크기를 동적으로 조절해야 하기 때문에 다양한 상황에 대비할 수 있는 최적의 제약사항을 고려해야 한다. 단순히 눈에 보이는 것만이 아닌 특정경우의 레이아웃과 같은 것들을 고려하면서 말이다. 

###### tags: `Auto layout`


### Reference
- [Apple Archive - Auto Layout](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/AutolayoutPG/AnatomyofaConstraint.html#//apple_ref/doc/uid/TP40010853-CH9-SW1)
- [Zeddios - AutoLayout](https://zeddios.tistory.com/380)
- [개발자 소들이 - AutoLayout](https://babbab2.tistory.com/135)

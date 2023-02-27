230227 nib vs xib, archive, awakeFromNib, Self-Sizing Table View Cells
===
학습내용
---
- nib vs xib, archive, awakeFromNib 
- Self-Sizing Table View Cell 오토레이아웃 잡기

## nib vs xib, archive, awakeFromNib

### ✏️ nib vs xib
nib은 (Next Interface Builder)의 약자, xib는 (Xml Interface Builder)의 약자이며 xib는 xml기반이고 nib은 바이너리이다. 

사실 원래 nib만 있었는데 xib가 추가된 것이라고 한다. xib가 "플랫 파일"에 저장된다는 점을 제외하고는 nib과 기능적으로 동일하다고 한다.
(flat file: 플랫파일은 아무런 구조적 상호관계가 없는 레코드들이 들어있는 파일)

xib는 Bundle이 아닌 플랫파일이기 때문에 SCM(소스 제어 관리)시스템을 보다 쉽게 처리할 수 있다고 한다.

그리고 빌드를 하면 xib가 앱에 포함될 nib파일로 컴파일이 된다고 한다.

### ✏️ archive, unarchiving, NSCoding

### 아카이빙
- iOS에서 모델 객체를 저장하는 방법 중 하나이며 가장 흔한 방법
- 객체 아카이브: 객체 그래프를 따라 데이터 내용을 저장하는 방식
    - 객체 그래프: 객체들의 참조 관계를 나타낸 모습
- 오브젝트나 어떤 구조의 데이터가 디스크에 저장되는 것

### 언아카이빙
- 아카이브한 데이터로부터 객체를 생성
- storyboard같은 interface 파일에 객체를 추가 -> 객체들이 아카이빙 -> 실행 중 객체들이 interface에서 언아키이빙되어 메모리에 로드

### 아카이빙 구현
```swift
class Person: NSCoding {
    var name: String = ""
    var age: Int

    func encode(with coder: NSCoder) {
        coder.encode(name, forKey: "name")
        coder.encode(age, forKey: "age")
    }

    required init?(coder: NSCoder) {
        name = coder.decodeObject(forKey: "name") as! String
        age = coder.decodeObject(forKey: "age") as! Int
    }
}
```



### ✏️ init(frame:), init(coder:)

UIView를 상속받은 커스텀뷰를 생성할 때 UIView의 필수 생성자 두개를 작성해야 한다.
- init(frame: CGRect) -> 코드로 뷰를 만들 때 호출됨
- init(coder: NSCoder) -> 스토리보드/nib로 뷰를 만들 때 호출됨

```swift
class MyCustomView: UIView {
    @IBOutlet weak var titleLabel: UILabel!
    @IBOutlet weak var descriptionLabel: UILabel!
    
    override init(frame: CGRect) {
        super.init(frame: frame)
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        custominit()
    }
    
    override func awakeFromNib() {
        super.awakeFromNib()
        titleLabel.text = "수정함"
        titleLabel.textColor = .blue
        descriptionLabel.text = "엥"
    }
    
    
    func custominit() {
        if let view = Bundle.main.loadNibNamed("MyCustomView", owner: self, options: nil)?.first as? UIView {
            view.frame = self.bounds
            addSubview(view) // 가져온 nib파일을 customView에 담는다. 담겨진 view에는 titleLabel과 descriptionLabel 객체가 존재한다.
        }
    }
}
```

위 코드는 커스텀 뷰를 nib파일로 생성한 경우 사용하는 코드이다. 

frame메서드는 코드로 커스텀 뷰를 생성시킬 때 호출이 되고 init?(coder)와 awakeFromNib()은 호출하지 않는다.


### ✏️ awakeFromNib()

awakeFromNub()은 XIB로 만든 커스텀 뷰나 인터페이스 빌더에서 연결된 객체를 설정할때 사용한다.

### 호출시점

Nib/Xib 파일이 언아카이브되고 난 후 init(coder: NSCoder)로 모든 객체가 초기화가 완료되었을 때 호출된다.
-> Nib파일이 인터페이스 빌더에서 빠져나와 메모리에 로드되고, init(coder:)에서 IBOulet, IBAction등 연결시켜 초기화를 한다. 이 이후에 awakeFromNib이 호출된다.

즉 오토레이아웃을 통해 제약을 잡아주거나 연결된 객체(IBOutlet)에 데이터를 설정하는 부분.

nib archive로 부터 재생성된 각 객체들에게awakeFromNib 메세지를 보내기 떄문에 하나의 커스텀뷰 안에 초기화된 객체들에게 전부 awakeFromNib메서드안에서 레이아웃 및 데이터 설정을 할 수 있다.

### 그럼 awakeFromNib은 커스텀 클래스인 경우에 꼭 필요한 메서드인가?
>서치를 기반으로 추측한 의견입니다. 이견이 있다면 댓글, DM보내주세요😀

모든 상황에 꼭 필요한 메서드는 아니다.

커스텀 뷰 내부에서 메서드를 정의해 외부에서 사용해 초기화면을 구성할 수 있기에 시점에 따라 필요할 수도, 필요없을 수도 있는 것 같다.

커스텀셀을 만들어 `configure`라는 메서드를 정의해 외부에서 사용한다면 굳이 `awakeFromNib`은 정의하지 않아도 될 것이다.

위의 예시를 대입해서 호출순서를 알아보자면 `MyCustomView` 클래스는 뷰 컨트롤러 내부에 있는 커스텀 view인데, 뷰컨의 loadView가 호출될 때 `MyCustomView`가 언아카이빙 되어 메모리에 올라가고, `MyCustomView`의 init(coder: NSCoder)가 호출이 된다. 
이 init에는 customInit이 호출되어 xib파일에 접근해서 가져온 뒤 frame을 설정하고 `awakeFromNib()` 메서드에서 데이터를 설정해준다.


### 추가적으로

뷰컨의 awakeFromNib() 시점에는 IBOutlet 접근이 불가하다. 왜냐하면 뷰컨과 뷰컨이 가진 서브뷰의 Nib 파일이 분리되어있기 때문이다. 

뷰컨의 nib은 top-level object nib으로서 이것만 로딩하고 서브뷰의 nib은 로딩하지않은채로 awakeFromNib()을 전달한다. 

서브뷰의 nib파일은 loadView() 메서드에서 비로소 로드된다고 한다.







## Self-Sizing Table View Cell 오토레이아웃 잡기

기존에 테이블 뷰 셀의 높이가 content가 커짐에 따라 커지게하려면 다음의 코드가 필요했다.
```swift
tableView.estimatedRowHeight = 85.0
tableView.rowheight = UITableViewAutomaticDimension
```

하지만 현재는 위 코드가 default값이기에 사용하지 않아도 동작한다.

커스텀 셀이 아니고 시스템에서 제공하는 셀의 경우 오토레이아웃을 직접 수정할 수 없다. 그래서 세부적인 제약을 줄 수없기에 커스텀셀을 보통 사용한다.

커스텀 셀은 UITableViewCell을 상속받아서 정의한다.

```swift 
class CustomClassCell: UITableViewCell {
    override func awakeFromNib() {
    }
}
```

UITableViewCell을 상속받기만 해도 내부적으로 `contentView` 프로퍼티를 갖게 된다.
<img src="https://i.imgur.com/UAXrRYg.png" width="600"/>

이 contentView안에 이미지나 타이틀, 서브타이틀 등을 addsubView해서 커스텀해서 사용하면 된다. 

accessoryView와 독립적이라는 사실을 주의해야 한다.

오른쪽 공간은 accessoryView이기 때문에 이를 침범하지 않도록 contentView 오토 레이아웃을 정확히 잡아주는 것이 중요하다.

이미지 뷰를 오토레이아웃 잡을때 width와 height 제약을 비율로준다면 모호하기에 오토레이아웃이 잘 잡히지 않을 수 있다.

예를 들어 아래 코드는 이미지뷰는 height가 width와 1:1비율이지만 이미지뷰가 셀의 bottom에 대해 어느정도 떨어져있는지에 대한 부분이 명시되어있지않아서 크기를 잡지 못한다.
```swift
// 셀 내부 contentView에 myImageView를 넣은 상황 
NSLayoutConstraint.activate([
    myImageView.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 20),
    myImageView.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 20),
    myImageView.widthAnchor.constraint(equalTo: contentView.widthAnchor, multiplier: 0.3),
    myImageView.heightAnchor.constraint(equalTo: myImageView.widthAnchor, multiplier: 1),
```

좀 더 명확하게 잡아준다면 셀의 이미지 크기가 비율대로 들어가게 된다.
```swift
NSLayoutConstraint.activate([
    myImageView.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 20),
    myImageView.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 20),
    myImageView.widthAnchor.constraint(equalTo: contentView.widthAnchor, multiplier: 0.3),
    myImageView.heightAnchor.constraint(equalTo: myImageView.widthAnchor, multiplier: 1),
    // 추가. contentView의 bottom을 넘어가지 않는다라는 제약만 걸어줘도 이미지뷰가 높이를 잡을 수 있게 된다.
    myImageView.bottomAnchor.constraint(lessThanOrEqualTo: contentView.bottomAnchor),
```
### 참고자료
- [zedd님 블로그- nib과 xib의 차이](https://zeddios.tistory.com/298)
- [김종권님 블로그 - Archive(아카이브), 아카이빙, 언아카이빙, NSCoding](https://ios-development.tistory.com/520)
- [NSCoding 이란?](https://woozzang.tistory.com/126)
- [스토리보드 요소가 Nib으로 다뤄지는과정](https://velog.io/@yohanblessyou/%EC%8A%A4%ED%86%A0%EB%A6%AC%EB%B3%B4%EB%93%9C%EC%9D%98-%EC%9A%94%EC%86%8C%EA%B0%80-Nib%EC%9D%B4-%EB%90%98%EB%8A%94-%EA%B3%BC%EC%A0%95-feat.-awakeFromNib)
- [야곰닷넷 - 오토레이아웃](https://yagom.net/courses/autolayout/lessons/understanding-auto-layout/)

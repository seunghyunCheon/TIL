230317 Responder Chain, Gesture Recognizer
===
학습내용
---
- Responder Chain, Gesture Recognizer

## Responder Chain, Gesture Recognizer

화면에 어떤 지점을 Touch했을 때 그 지점에 있는 가장 하위의 View에서부터 Responder Chain이 시작된다. 

가장 깊은 View는 Root View를 의미하며 이 뷰에 `addSubview`로 두 개의 뷰를 추가했다면 추가한 두 개의 뷰가 다음으로 깊숙한 뷰가 된다. 이는 `hitTest`메서드를 통해 체이닝형식으로 First Responder를 찾을 수 있다.

### OrangeView 클릭 시
`BlueView(nil) -> GreenView -> PurpleView(nil) -> OrangeView`
먼저 rootView에 직접적인 SubView들인 BlueView와 GreenView가 실행되고, BlueView는 nil을 리턴하여 없어지고 GreenView에서 PurpeView도 nil을 리턴하여 없어지고 그 다음인 OrangeView에서 point지점이 있으니 이 뷰를 리턴한다. 
그래서 이 OrangeView가 First Responder가 되어 이벤트를 처리한다.

### GreenView 클릭 시
`BlueView(nil) -> GreenView -> PurpleView(nil) -> OrangeView(nil) -> GreenView `
GreenView내부에서 터치한 지점의 좌표에 해당하는 서브뷰들이 존재하지 않기 때문에 GreenView가 처리한다.

### PurpleView 클릭 시
겹쳐진 부분과 겹쳐지지않은 부분의 결과가 다르다.

`self.point`는 receiver가 특정 point를 포함하는지 아닌지에 해당하는 Bool타입을 반환한다. 

GreenView에서 `self.point`를 할 때 GreenView가 특정 point(겹치지않은 부분을 누름)를 포함하지 않기 때문에 nil을 리턴하기 때문에 GreenView의 슈퍼뷰인 root View가 이벤트를 처리하게 된다.

겹쳐진 부분은 `self.point` 시 특정 point를 포함하고 있기 때문에 PurpleView까지 이벤트를 전달할 수 있는 것이다.


### View에 Gesture Recognizer 추가

```swift
class ViewController: UIViewController {
    @IBOutlet weak var greenView: GreenView!
    override func viewDidLoad() {
        super.viewDidLoad()
        greenView.addGestureRecognizer(UITapGestureRecognizer(target: self, action: #selector(tappedconcern1(_:))))
    }
    
    @objc func tappedconcern1(_ gesture: UITapGestureRecognizer) {
        if greenView.backgroundColor == .black {
            greenView.backgroundColor = .green
        } else {
            greenView.backgroundColor = .black
        }
    }
}
```

`hitTest`를 이용해서 이벤트를 GreenView에서 처리하게 만드려면 OrangeView에서 `hitTest`를 override해 nil을 반환하는 방법이 있을 것 같다.


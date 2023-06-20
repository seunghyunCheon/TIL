# 230620 Coordinator Pattern, Coordinator pattern with Tab Bar Controller
### 학습내용
- Coordinator Pattern
- Coordinator pattern with Tab Bar Controller

## Coordinator Pattern
Coordinator는 움직임을 조정하는 사람으로 이 패턴도 동일한 의미로 사용이 된다.

Coordinator Pattern은 2015년 Soroush Khanlou가 소개하면서 알려졌다.
> Khanlou: Massive ViewController의 가장 큰 문제는 flow logic(흐름 로직)과 비즈니스 로직이 얽혀있다는 것이다.

비즈니스 로직은 클린아키텍처의 유즈케이스를 통해서 해결할 수 있다. 이번에 살펴볼 부분은 흐름 로직에 대한 부분이다. 

Khanlou의 예시 코드는 다음과 같다.

```swift
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    let object = self.dataSource[indexPath]
    let detailViewController = SKDetailViewController(with: object)
    self.navigationController?.present(detailViewController, animated: true, completion: nil)
}
```

1. 객체를 가져온다.
2. 뷰컨을 만든다.
3. 뷰컨을 보여준다.

기존에는 이와 같은 방식으로 진행했을 것이다. 하지만 뷰컨트롤러가 커진다면? 컨텐츠를 보여주고 라이프사이클을 관리하는 역할 뿐만 아니라 이러한 흐름로직을 담게 되면서 점차 복잡해질 것이다. 
그리고 `self.navigationController`를 통해 부모에게 자신이 만든 뷰컨으로 이동하라고 메세지까지 보내고 있다. 이러한 흐름로직이 여러 군데 분산되어있다면 코드를 찾기가 굉장히 불편할 것이다. 

그러다 Khanlou는 ViewController가 UI의 본질을 갖고있기 때문에 흐름을 처리하는 다른 무언가가 필요하다고 느껴 Coordinator라는 객체를 만들었다.

그리고 이 패턴을 제대로 실행하려면 전체 앱을 direct하는 high-level coordinator가 필요하다고 말했다. 

즉 **AppDelegate(SceneDelegate)는 AppCoordinator를 유지하며, 모든 Coordinator에는 일련의 하위 Coordinator**가 존재하는 것이다. 

## Coordinator pattern with Tab Bar Controller

### Coordinator 프로토콜은 어떤 모양일까?

```swift
protocol Coordinator: AnyObject {
    var finishDelegate: CoordinatorFinishDelegate? { get set }
    // 각 coodinator는 하나의 navigationController를 갖고있다.
    var navigationController: UINavigationController { get set }
    // 모든 child coordinator를 추적한다. 대부분의 경우 이 어레이에는 하위 코디네이터가 하나만 포함된다.
    var childCoordinaotrs: [Coordinator] { get set }
    // flow type을 정의
    var type: CoordinatorType { get }
    // 흐름을 시작하는 로직을 담는다.
    func start()
    // 흐름을 종료하는 로직을 담는다. 모든 자식 coordinator를 제거하기위해 부모에게 deallocated될 준비가 되었다는 것을 알린다.
    func finish()
    
    init(_ navigationController: UINavigationController) 
}

extension Coordinator {
    func finish() {
        childCoordinators.removeAll()
        finishDelegate?.coordinatorDidFinish(childCoordinator: self)
    }
}

// MARK: - CoordinatorOutput

// 자식이 끝낼 준비가 되었을 때 부모 Coordinator가 알도록 도와주는 프로토콜 델리게이트
protocol CoordinatorFinishDelegate: AnyObject {
    func coordinatorDidFinish(childCoordinator: Coordinator)
}

// MARK: - 앱에서 사용할 흐름타입을 정의
enum CoordinatorType {
    case app, login, tab
}
```

이 구현은 클래식한 예시와 차이점이 존재한다. 각 프로퍼티와 메서드가 더욱 복잡하고 관리가능한 앱을 만들도록 하기 때문이다. 

### 어떻게 사용할까?
모든 앱은 흐름의 시작점인 하나의 main Coordinator를 가져야 한다. - AppCoordinator. 이곳에서 시작되는 흐름을 정의하기 위해 위에서 정의한 Coordinator를 파생해서 추가적인 메서드 요구사항을 정의했다.

```swift
// 앱 coordinator로부터 시작될 수 있는 흐름을 정의
protocol AppCoordinatorProtocol: Coordinator {
    func showLoginFlow()
    func showMainFlow()
}

// 앱 coordinator는 앱 라이프사이클 동안 존재할 유일한 하나의 coordinator
class AppCoordinator: AppCoordinatorProtocol {
    weak var finishDelegate: CoordinatorFinishDelegate? = nil
    
    var navigationController: UINavigationController
    
    var childCoordinators = [Coordinator]()
    
    var type: CoordinatorType { .app }
    
    required init(_ navigationController: UINavigationController) {
        self.navigationController = navigationController
        navigationController.setNavigationBarHidden(true, animated: true)
    }
    
    func start() {
        showLoginFlow()
    }
    
    func showLoginFlow() {
        // 로그인 flow 구현
    }
    
    func showMianFlow() {
        // 메인(탭바) flow 구현
    }
}

extension AppCoordinator: CoordinatorFinishDelegate {
    func coordinatorDidFinish(childCoordinator: Coordinator) {
        childCoordinators = childCoordinators.filter({ $0.type != childCoordinator.type})
    }
}
```

앱 coordinator를 초기화시키는 가장 좋은 곳은 AppDelegate이다.
```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    // Override point for customization after application launch.
            
    window = UIWindow.init(frame: UIScreen.main.bounds)
    
    let navigationController: UINavigationController = .init()

    window?.rootViewController = navigationController
    window?.makeKeyAndVisible()
    
    appCoordinator = AppCoordinator.init(navigationController)
    appCoordinator?.start()
            
    return true
}
```

### 첫번째 플로우 설정
보통 앱은 사용자가 로그인하거나 회원가입하는 화면에서 시작된다. 

먼저 `LoginViewController`부터 만들어보자.

```swift
class LoginViewController: UIViewController {
    var didSendEventClosure: ((LoginViewController.Event) -> Void)?
    
    private let loginButton: UIButton = {
        let button = UIButton()
        button.setTitle("Login", for: .normal)
        button.backgroundColor = .systemBlue
        button.setTitleColor(.white, for: .normal)
        button.layer.cornerRadius = 8.0
        
        return button
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()

        view.backgroundColor = .white
        
        view.addSubview(loginButton)

        loginButton.translatesAutoresizingMaskIntoConstraints = false

        NSLayoutConstraint.activate([
            loginButton.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            loginButton.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            loginButton.widthAnchor.constraint(equalToConstant: 200),
            loginButton.heightAnchor.constraint(equalToConstant: 50)
        ])
        
        loginButton.addTarget(self, action: #selector(didTapLoginButton(_:)), for: .touchUpInside)
    }
    
    deinit {
        print("LoginViewController deinit")
    }
    
    @objc
    private func didTapLoginButton(_ sender: Any) {
        didSendEventClosure?(.login)
    }
}

extension LoginViewController {
    enum Event {
        case loign
    }
}
```

다음으로 login flow를 담당하는 LoginCoordinator를 작성해보자.
```swift
protocol LoginCoordinatorProtocol: Coordinator {
    func showLoginViewController()
}

class LoginCoordinator: LoginCoordinatorProtocol {
    weak var finishDelegate: CoordinatorFinishDelegate?
    
    var navigationController: UINavigationController
    
    var childCoordinators: [Coordinator] = []
    
    var type: CoordinatorType { .login }
    
    required init(_ navigationController: UINavigationController) {
        self.navigationController = navigationController
    }
    
    func start() {
        showLoginViewController()
    }
    
    deinit {
        print("LoginCoordinator deinit")
    }
    
    func showLoginViewController() {
        let loginVC: LoginViewController = .init()
        loginVC.didSendEventClosure = { [weak self] event in
            self?.finish()
        }
        
        navigationController.pushViewController(loginVC, animated: true)
    }
}
```

사용자가 로그인 버튼을 탭하면 coordinator가 이 이벤트를 받고 finish 함수를 호출한다. 기본적으로 finishDelegate에 이를 알려서 다음에 일어날 일을 설정한다.

### 메인 플로우를 정의
어떻게 TabBarController와 함께 main flow를 정의해야 할까? main Coordinator를 `TabCoordinator`로 네이밍 지어보자. 먼저 어떤 탭바가 선택되는지 보기위한 enum을 정의해보자.

```swift
enum TabBarPage {
    case ready
    case steady
    case go
    
    init?(index: Int) {
        switch index {
            case 0:
                self = .ready
            case 1:
                self = .steady
            case 2:
                self = .go
            default:
                return nil
        }
    }
    
    func pageTitleValue() -> String {
        switch self {
            case .ready:
                return "Ready"
            case .steady:
                return "Steady"
            case .go:
                return "Go"
        }
    }
    
    func pageOrderNumber() -> Int {
        switch self {
            case .ready:
                return 0
            case .steady:
                return 1
            case .go:
                return 2
        }
    }
}
```

이런식으로 enum을 구성할 수도 있겠지만 사용되는 메서드가 많아진다면 TabBarItem프로토콜을 정의한 뒤 이를 채택하는 구체타입 여러개를 정의하는 것도 좋은 방법일 것 같다.

이제 본격적으로 TabBarCoordinator를 작성해보자.

```swift
protocol TabCoordinatorProtocol: Coordinator {
    var tabBarController: UITabBarController { get set }
    func selectPage(_ page: TabBarPage)
    func setSelectedIndex(_ index: Int)
    func currentPage() -> TabBarPage?
}

class TabCoordinator: NSObject, Coordinator {
    weak var finishDelegate: CoordinatorFinishDelegate?
    
    var childCoordinators: [Coordinator] = []
    
    var navigationController: UINavigationController
    
    var tabBarController: UITabBarController
    
    var type: CoordinatorType { .tab }
    
    required init(_ navigationController: UINaigationController) {
        self.navigationController = navigationController
        self.tabBarController = .init()
    }
    
    func start() {
        // 탭바에 추가할 페이지를 정의
        let pages: [TabBarPage] = [.go, .steady, .ready]
            .sorted(by: { $0.pageOrderNumber() < $1.pageOrderNumber() })
        
        let controllers: [UINavigationController] = pages.map({ getTabController($0) })
        
        prepareTabBarController(withTabControllers: controllers)
    }
    
    deinit {
        print("TabCoordinator deinit")
    }
    
    private func prepareTabBarController(withTabControllers tabControllers: [UIViewController]) {
        // 탭바 델리게이트 설정
        tabBarController.delegate = self
        tabBarController.setViewControllers(tabControllers, animated: true)
        tabBarController.selectedIndex = TabBarPage.ready.pageOrderNumber()
        tabBarController.tabBar.isTranslucent = false
        
        navigationController.viewControllers = [tabBarController]
    }
    
    private func getTabController(_ page: TabBarPage) -> UINavigationController {
        let navController = UINavigationController()
        navController.setNavigationBarHidden(false, animated: false)
        navController.tabBarItem = UITabBarItem.init(title: page.pageTitleValue(),
                                                     image: nil,
                                                     tag: page.pageOrderNumber())
        switch page {
        case .ready:
            // 필요하다면 자기 자신의 coordinator를 가질 수 있다.
            let readyVC = ReadyViewController()
            readyVC.didSendEventClosure = { [weak self] event in
                switch event {
                case .ready:
                    self?.selectPage(.steady)
                }
            }
                        
            navController.pushViewController(readyVC, animated: true)
        case .steady:
            let steadyVC = SteadyViewController()
            steadyVC.didSendEventClosure = { [weak self] event in
                switch event {
                case .steady:
                    self?.selectPage(.go)
                }
            }
            
            navController.pushViewController(steadyVC, animated: true)
        case .go:
            let goVC = GoViewController()
            goVC.didSendEventClosure = { [weak self] event in
                switch event {
                case .go:
                    self?.finish()
                }
            }
            
            navController.pushViewController(goVC, animated: true)
        }
        
        return navController
    }
    
    func currentPage() -> TabBarPage? { TabBarPage.init(index: tabBarController.selectedIndex) }
    
    func selectPage(_ page: TabBarPage) {
        tabBarController.selectedIndex = page.pageOrderNumber()
    }
    
    func setSelectedIndex(_ index: Int) {
        guard let page = TabBarPage.init(index: index) else { return }
        
        tabBarController.selectedIndex = page.pageOrderNumber()
    }
}

// MARK: - UITabBarControllerDelegate
extension TabCoordinator: UITabBarControllerDelegate {
    func tabBarController(_ tabBarController: UITabBarController,
                          didSelect viewController: UIViewController) {
        // Some implementation
    }
}
```

세 개의 뷰컨은 기본적으로 동일하다. start메서드를 통해 탭바의 흐름을 조정하도록 준비한다.

이제 이 coordinator들을 사용하는 코드를 작성해보자. AppCoordinator에서 작성하는 것이 좋아보이기 때문에 다음과 같이 작성한다.

```swift
func showLoginFlow() {
    let loginCoordinator = LoginCoordinator.init(navigationController)
    loginCoordinator.finishDelegate = self
    loginCoordinator.start()
    childCoordinators.append(loginCoordinator)
}

func showMainFlow() {
    let tabCoordinator = TabCoordinator.init(navigationController)
    tabCoordinator.finishDelegate = self
    tabCoordinator.start()
    childCoordinators.append(tabCoordinator)
}
```
그리고 해당 플로우가 끝났을 때 새로운 흐름을 전달하도록 다음의 코드를 추가해준다.

```swift

extension AppCoordinator: CoordinatorFinishDelegate {
    func coordinatorDidFinish(childCoordinator: Coordinator) {
        childCoordinators = childCoordinators.filter({ $0.type != childCoordinator.type })

        switch childCoordinator.type {
        case .tab:
            navigationController.viewControllers.removeAll()

            showLoginFlow()
        case .login:
            navigationController.viewControllers.removeAll()

            showMainFlow()
        default:
            break
        }
    }
}
```

### Coordinator의 이점
1. ViewController의 고립
뷰컨은 데이터를 표시하는 방법 외에는 아무것도 모르며 어떤 일이 발생할 때마다 대리자에게 알리지만 대리자가 누군지는 알 수 없게 된다.

2. 의존성 주입
여러 객체나 클린아키텍처를 사용하는 경우 여러 UseCase, Repository를 주입하는 역할을 coordinator가 지게되면서 뷰컨의 역할을 줄인다.


### 참고문서
- [Zedd - Coodinator](https://zeddios.medium.com/coordinator-pattern-bf4a1bc46930)
- [Coordinator pattern with Tab Bar Controller](https://somevitalyz123.medium.com/coordinator-pattern-with-tab-bar-controller-33e08d39d7d)

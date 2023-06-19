# 230612 Clean Architecture and MVVM on iOS
### 학습내용
- Clean Architecture and MVVM on iOS

## Clean Architecture and MVVM on iOS
소프트웨어를 개발할 때 디자인패턴 뿐만 아니라 아키텍처 패턴도 중요하다.
소프트웨어 엔지니어링에서는 많은 다양한 아키텍처 패턴이 존재한다. 모바일 분야에서는 가장 널리 사용되는 것은 MVVM, Clean Architecture 및 Redux 패턴이다.

<img src="https://hackmd.io/_uploads/SyFbY_Bv2.png" width="500"/><br/>
위 사진처럼 다양한 레이어가 존재한다. 중요한 규칙은 내부 레이어가 외부 레이어를 알지 못하는 것이다. 바깥에서 안쪽으로의 화살표가 의존성 규칙이다. 외부에서 내부로만 종속성이 있을 수 있는 것이다.

<img src="https://hackmd.io/_uploads/S1Yk9OBv2.png" width="500"/><br/>

### Domain Layer
도메인 계층은 가장 안쪽 레이어다. 이곳에서는 `Entities`, `UseCases`, `Interfaces`를 포함하고 있다. 이 레이어는 다른 프로젝트에서도 재사용될 수 있다. 이러한 분리는 종속성이 필요하지 않기 때문에 테스트 대상 내에서 호스트 앱을 사용하지 않도록 허용한다. 이렇게 하면 도메인을 테스트를 하는데 몇초밖에 걸리지 않는다. 
도메인 레이어는 다른 레이어의 어떤것도 포함하면 되지 않는 것이 중요하다.(UIKit, SwiftUI, Codable을 mapping하는 Data Layer)

좋은 아키텍처가 유즈케이스를 중심으로 하는 이유는 설계자가 프레임워크, 도구 및 환경에 전념하지 않고 해당 유즈케이스를 지원하는 구조를 안전하게 설명할 수 있기 때문이다.

### Presentation Layer
UI(UIViewController, SwiftUI)를 포함한다. 뷰들은 뷰모델(Presenter)에 의해 조정이 되어진다. 이 뷰모델은 하나이상의 유즈케이스를 실행시킨다. Presentation Layer는 오로지 Domain Layer만 의존한다.

### Data Layer
Repository 구현과 하나이상의 Data Source를 포함한다. 레포지토리들은 서로 다른 데이터 소스의 데이터 조정을 담당한다. 데이터 소스는 Remote나 Local이 될 수 있다. 
데이터 레이어는 오직 Domain Layer에 의존한다. 이 계층에서 네트워크 Json Data의 매핑을 추가할 수도 있다.

<img src="https://hackmd.io/_uploads/Hk8aAOSw3.png" width="500"/><br/>
1. View는 ViewModel의 메서드를 호출한다.
2. ViewModel은 UseCase를 실행한다.
3. UseCase는 사용자 및 리포지토리의 데이터를 결합한다.
4. 각각의 Repository는 원격 데이터(네트워크), 영구 DB 스토리지 소스 또는 메모리 내 데이터(원격 또는 캐시됨)에서 데이터를 반환한다.
5. 위 정보가 View로 흐른다.

### Dependency Direction
Presentation Layer -> Domain Layer <- Data Repositories Layer
Presentation Layer (MVVM) = ViewModels(Presenters) + Views(UI)

Domain Layer = Entities + Use Cases + Repositories Interfaces

Data Repositories Layer = Repositories Implementations + API(Network) + Persistence DB


### 예시 앱
<img src="https://hackmd.io/_uploads/Bk3VoqIPh.png" width="500"/><br/>

도메인 레이어에는 엔티티와 유즈케이스가 존재한다. 이 유즈케이스는 영화를 검색하고 최신영화검색기록을 저장한다.

```swift
protocol SearchMoviesUseCase {
    func execute(requestValue: SearchMoviesUseCaseRequestValue,
                 completion: @escaping (Result<MoviesPage, Error>) -> Void) -> Cancellable?
}

final class DefaultSearchMoviesUseCase: SearchMoviesUseCase {

    private let moviesRepository: MoviesRepository
    private let moviesQueriesRepository: MoviesQueriesRepository
    
    init(moviesRepository: MoviesRepository, moviesQueriesRepository: MoviesQueriesRepository) {
        self.moviesRepository = moviesRepository
        self.moviesQueriesRepository = moviesQueriesRepository
    }
    
    func execute(requestValue: SearchMoviesUseCaseRequestValue,
                 completion: @escaping (Result<MoviesPage, Error>) -> Void) -> Cancellable? {
        return moviesRepository.fetchMoviesList(query: requestValue.query, page: requestValue.page) { result in
            
            if case .success = result {
                self.moviesQueriesRepository.saveRecentQuery(query: requestValue.query) { _ in }
            }

            completion(result)
        }
    }
}

// Repository Interfaces
protocol MoviesRepository {
    func fetchMoviesList(query: MovieQuery, page: Int, completion: @escaping (Result<MoviesPage, Error>) -> Void) -> Cancellable?
}

protocol MoviesQueriesRepository {
    func fetchRecentsQueries(maxCount: Int, completion: @escaping (Result<[MovieQuery], Error>) -> Void)
    func saveRecentQuery(query: MovieQuery, completion: @escaping (Result<MovieQuery, Error>) -> Void)
}
```

`execute`함수 부분을 컴바인으로 바꾼다면 다음과 같을 것이다. 이 상황에서는 fetchRequest에서 실제 비동기 작업이 일어나고 Repository에서 네트워크와 통신하는 코드가 존재한다고 가정해보자.

```swift
// MoviesRepository.swift
func fetchMovieList(query: String, page: Int) -> AnyPublisher<MoviesPage, Error> {
    return Future<MoviesPage, Error> { promise in
        // 비동기로 데이터 가져오기.
        // 성공한 경우 promise(.success(moviesPage))
        // 실패한 경우 promise(.failure(error))
    }
    .eraseToAnyPublisher()
}

// MoviesQueriesRepository.swift
func saveRecentQuery() -> AnyPublisher<Void, Error> {
    return Future<Void, Error> { promise in 
        // 비동기로 저장하기
        // 성공한 경우 promise(.success(()))
        // 실패한 경우 promise(.failure(error))
    }
}

// DefaultSearchMoviesUseCase.swift
func execute(requestValue: SearchMoviesUseCaseRequestValue) -> AnyPublisher<MoviesPage, Error> {
    return moviesRepository.fetchRequestMoviesList(query: requestValue.query, page: requestValue.page)
    .flatMap { [weak self] moviePages -> AnyPublisher<MovivesPage, Error> in
        guard let self else { return Fail(error: SearchMoviesUseCaseeError.failedToSaveQuery.eraseToAnyPublisher() }
        return self.moviesQueriesRepository.saveRecentQuery(query: requestValue.query)
              .map { _ in moviePages }
              .eraseToAnyPublisher()
    }
    .eraseToAnyPublisher()
}
```
UseCase를 생성하는 또다른 방법은 `start()`함수와 함께 UseCase 프로토콜을 사용하며 모든 useCase들이 이 프로토콜을 따르도록 하는 것이다. 
UseCase들을 `Interactors`라고 부른다.

### Presentation Layer
프레젠테이션 계층에는 MoviesListView에서 관찰되는 항목이 있는 MoviesListViewModel이 포함된다. MoviesListViewModel은 UIKit을 import하지 않는다. UIKit, SwiftUI 또는 WatchKit과 같은 모든 UI 프레임워크에서 ViewModel을 깨끗하게 유지하면 쉽게 재사용하고 리팩터링할 수 있기 때문이다.

```swift
protocol MoviesListViewModelInput {
    func didSearch(query: String)
    func didSelect(at indexPath: IndexPath)
}

protocol MoviesListViewModelOutput {
    var items: Observable<[MoviesListItemViewModel]> { get }
    var error: Observable<String> { get }
}

protocol MoviesListViewModel: MoviesListViewModelInput & MoviesListViewModelOutput { }

struct MoviesListViewModelActions {
    let showMovieDetails: (Movie) -> Void
}

final class DefaultMoviesListViewModel: MoviesListViewModel {
    private let searchMovieUseCase: searchMoviesUseCase
    private let actions: MoviesListViewModelActions?
    
    private var movies: [Movie] = []
    
    // MARK: - OUTPUT
    let itmes: Observable<[MoviesListItemViewModel]> = Obserable([])
    let error: Observable<String> = Observable("")

    init(searchMoviesUseCase: SearchMoviesUseCase, actions: MoviesListViewModelActions) {
        self.searchMoviesUseCase = searchMoviesUseCase
        self.actions = actions
    }
    
    private func load(movieQuery: MovieQuery) {
        searchMoviesUseCase.execute(movieQuery: movieQuery) { result in 
           switch result {
               case .success(let moviesPage):
                    // 도메인과 뷰를 분리하기 위해 도메인 엔티티를 뷰모델로 매핑하는 코드를 적어야 한다.
                    self.items.value += moviesPage.movies.map(MoviesListItemViewModel.init)
                    self.movies += moviesPage.movies
               case .failure:
                   self.error.value = NSLocalizedString("Failed loading movies")
           }                                                     
        }
    }
}

// MARK: - INPUT. View event methods
extension MoviesListViewModel {
    func didSearch(query: String) {
        load(movieQuery: MovieQuery(query: query))
    }
    
    func didSelect(at indexPath: IndexPath) {
        actions?.showMovieDetails(movies[indexPath.row])
    }
}
struct MoviesListItemViewModel: Equatable {
    let title: String
}

extension MoviesListItemViewModel {
    init(movie: Movie) {
        self.title = movie.title ?? ""
    }
}
```

`MoviesListViewModelInput`과 `MoviesListViewModelOutput` 인터페이스를 사용해서 테스터블하게 만들 수 있다. 또한 MoviesListViewModelActions 클로저가 있어 MoviesSearchFlowCoordinator에 다른 뷰를 표시할 시기를 알려준다. 이 action클로저가 호출되었을 때 coorinator는 movie detail화면을 보여줄 것이다. 필요한 경우 나중에 더 많은 작업을 쉽게 추가할 수 있기 때문에 구조체를 사용하여 작업을 그룹화한다.

Presentation Layer는 ViewModel의 data에 바인딩되어있는 MoviesListViewController도 포함한다. 

UI는 비즈니스 로직이나 애플리케이션 로직에 접근할 수 없고 오직 뷰모델만이 가능하다. 이는 관심사의 분리이다. 비즈니스 모델을 뷰(UI)로 직접 전달할 수 없기 때문에 뷰모델 내부의 뷰모델에 매핑하고 전달한다.

또한 영화 검색을 시작하기 위해 View에서 ViewModel로 검색 이벤트 호출을 추가한다.

```swift
final class MoviesListViewController: UIViewController, StoryboardInstatiable, UISearchBarDelegate {
    private var viewModel: MoviesListViewModel!
    
    final class func create(with viewModel: MoviesListViewModel) -> MoviesListViewController {
        let vc = MoviesListViewController.instantiateViewController()
        vc.viewModel = viewModel
        
        return vc
    }
    
    overrid func viewDidload {
        super.viewDidLoad()
        bind(to: viewModel)
    }
    
    private func bind(to viewModel: MoviesListViewModel) {
        viewModel.items.observe(on: self) { [weak self] items in
            self?.moviesTableViewController?.items = items
        }
        viewModel.error.observe(on: self) { [weak self] error in 
            self?.showError(error)                                  
        }
    }
    
    func searchBarSearchButtonCliecked(_ searchBar: UISearchBar) {
        guard let searchText = searchBar.text, !searchText.isEmpty else { return }
        viewModel.didSearch(query: searchText)
    }
}
```
items를 관찰하고 item가 변할 때마다 뷰가 리로드되는 것에 주목하자. 

그리고 flow coordinator로부터 디테일 화면을 보여주기 위해 `MoviesSearchFlowCoordinator`내부의 `MovieListViewModel`의 action메서드인 `showMovieDetails(movie:)`함수를 할당한다. 

```swift
protocol MoviesSearchFlowCoordinatorDependencies {
    func makeMoviesListViewController() -> UIViewController
    func makeMovesDetailsViewController(movie: Movie) -> UIViewController
}

final class MoviesSearchFlowCoordinator {
    private weak var navigationCOntroller: UINavigationController?
    private let dependencies: MoviesSearchFlowCoordinatorDependencies

    init(navigationController: UInavigationController,
         dependencies: MoviesSearchFlowCoordinatorDependencies) {
        self.navigationController = navigationController
        self.dependencies = dependencies
    }
    
    func start() {
        let actions = MoviesListViewModelActions(showMovieDetails: showMovieDetails)
        let vc = dependencies.makeMoviesListViewController(actions: actions)
    
        navigationController?.pushViewController(vc, animated: false)
    }
    
    private func showMovieDetails(movie: Movie) {
        let vc = dependencies.makeMoviesDetailViewController(movie: movie)
        navigationController?.pushViewController(vc, animated: true)
    }
}
```

View Controller의 크기와 책임을 줄이기 위해 프레젠테이션 로직에 Flow Coordinator를 사용한다.
필요한 동안 Flow가 할당 해제되지 않도록 유지하기 위해 Flow에 대한 강력한 참조(액션 클로저, 자체 기능 포함)가 있다.

이 접근법으로 쉽게 동일한 ViewModel을 수정하지 않고 다른 뷰를 쉽게 사용할 수 있다. 

### Data Layer
Data Layer는 DefaultMoviesRepository를 포함한다. 이는 도메인 레이어 안에 정의된 인터페이스를 준수한다. 이곳에 JSON data를 매핑하는 코드와 코어데이터 엔티티를 도메인 모델로 매핑하는 코드를 작성한다.

```swift
final class DefaultMoviesRepository {
    private let dataTransferService: DataTransfer
    
    init(dataTransferService: DataTransfer) {
        self.dataTransferService = dataTransferService
    }
}

extension DefaultMoviesRepository: MoviesRepository {
    public func fetchMoviesList(query: MovieQuery, page: Int, completion: @escaping (Result<MoviesPage, Error>) -> Void) -> Cancellable? {
        let endpoint = APIEndpints.getMovies(query: MovieQuery, page: Int, completion: @escaping (Result<MoviePage, Error>) -> Void) -> Cancellable? {
            return dataTransferService.request(with: endpoint) { (response: Result<MoviesResponseDTO, Error>) in
                switch response {
                    case .success(let moviesResponseDTO):
                        completion(.success(moviesResponseDTO.toDomain()))
                    case .failure(let error):
                        completion(.failure(error))
                }
            }
        }
    }
}

// MARK: - Data Transfer Object (DTO)
// 내부에서 JSON 응답을 도메인으로 인코딩/디코딩하기 위한 중간 개체로 사용.
struct MoviesRequestDTO: Encodable {
    let query: String
    let page: Int
}

struct MoviesResponseDTO: Decodable {
    private enum CodingKeys: String, CodingKey {
        case page
        case totalPages = "total_pages"
        case movies = "results"
    }
    let page: Int
    let totalPages: Int
    let movies: [MovieDTO]
}
...
// MARK: - Mappings to Domain
extension MoviesResponseDTO {
    func toDomain() -> MoviesPage {
        return .init(page: page,
                     totalPages: totalPages,
                     movies: movies.map { $0.toDomain() })
    }
}
```
Data Transfer Object DTO는 JSON응답을 도메인으로 매핑해주는 중간 객체이다. 또한 만약 엔드포인트 응답을 캐시하려면 데이터 전송 객체를 영구 객체에 매핑하여 영구 저장소에 저장한다. 

일반적으로 Data Repository는 API 데이터 서비스 및 영구 데이터 저장소와 함께 주입될 수 있다. Data Repository는 데이터를 반환하는 이 두개의 의존성과 함께 동작한다. 규칙은 먼저 캐시된 데이터 출력에 대한 영구 저장소를 요청하는 것이다.(NSManagedObject는 DTO 개체를 통해 Domain에 매핑되고 캐시된 데이터 클로저에서 반환된다). 그런다음 최신으로 업데이트된 데이터를 반환하는 API Data Service를 호출한다. (DTO는 영구 개체에 매핑되고 저장된다) 그런다음 DTO는 도메인으로 매핑되고 업데이트된 데이터 클로저안에서 반환된다.
이렇게 하면 사용자가 즉시 데이터를 볼 수 있다. 인터넷에 연결되어 있지 않아도 사용자는 계속해서 영구 저장소의 최신 데이터를 볼 수 있다.

저장소와 API는 전적으로 다른 구현으로 교체될 수 있다. (CoreData에서 Realm으로). 다른 레이어들은 DB의 변화로 영향을 받지 않기 때문이다.

### Infrastructure Layer(Network)
네트워크와 관련된 계층으로 Almofire나 다른 프레임워크가 이에 해당한다. 이 부분은 네트워크 파라미터와 함께 설정될 수 있다. 또한 엔드포인트 정의와 데이터 매핑 메서드를 포함하고 있다.

```swift
struct APIEndpoints {
    static func getMovies(with moviesRequestDTO: MoviesRequestDTO) -> Endpoint<MoviesResponseDTO> {
        return Endpoint(path: "search/movie/",
                        method: .get,
                        queryParameterEncodable: moviesRequestDTO)
    }
}

let config = ApiDataNetworkConfig(baseURL: URL(string: appConfigurations.apiBaseURL)!,
                                 queryParameters: ["api_key": appConfigurations.apiKey])
let apiDataNetwork = DefaultNetworkService(session: URLSession.shard,
                                          config: config)
let endpoint = APIEndpoint.getMovies(with: MoviesRequestDTO(query: query.query, page: page))
dataTransferService.request(with: endpoint) { (response: Result<MoviesResponseDTO, Error>) in
    let moviesPage = try? response.get()
}
```

### MVVM
Model-View-ViewModel 패턴은 UI와 도메인 사이의 관심사를 분리한다.

클린아키텍처와 함께 이 패턴을 사용할 때 Presentation과 UI Layer사이의 관심사를 분리하는데 도움을 준다.

여러개의 뷰들은 하나의 뷰모델에 대응될 수 있다. 예를 들어 CarsAroundListView와 CarsAroundMapView는 둘 다 CarsAroundViewModel을 사용할 수 있다. 또한 어떤 뷰는 UIKit으로 또 어떤 뷰는 SwiftUI로 작성할 수 있다. 뷰모델에서는 UIKit, WatchKit, SwiftUI를 import하지 않는 것이 중요하다. 
<img src="https://hackmd.io/_uploads/HyHRjUpD2.png" width="500"/><br/>

뷰와 뷰모델 사이의 데이터바인딩은 클로저, 델리게이트, observable(RxSwift)를 통해 할 수 있다. 컴바인과 SwiftUI조합은 최소 배포타겟이 iOS13이상인 경우에 사용할 수 있다. 

`View`는 `ViewModel`에 직접적인 관계를 가지고 뷰에서 이벤트가 일어날 때마다 뷰모델에 알린다. 

예를들어 클로저와 didSet조합으로 third-party의존성을 피할 수 있다.

```swift
final class Observable<Value> {
    struct Observer<Value> {
        weak var observer: AnyObject?
        let block: (Value) -> Void
    }
    
    private var observers = [Observer<Value>]()
    
    var value: Value {
        didSet { notifyObservers() }
    }
    
    init(_ value: Value) {
        self.value = value
    }
    
    func observe(on observer: AnyObject, observerBlock: @escaping (Value) -> Void) {
        observers.append(Observer(observer: observer, block: observerBlock))
        observerBlock(self.value)
    }
    
    func remove(observer: AnyObject) {
        observers = observers.filter { $0.observer !== observer }
    }
    
    private func notifyObservers() {
        for observer in observers {
            for observer in observers {
                observer.block(self.value)
            }
        }
    }
}
```

뷰컨에서의 데이터바인딩은 다음과 같다.

```swift
final class ExampleViewController: UIViewController {
    private var viewModel: MoviesListViewModel!
    
    private func bind(to viewModel: ViewModel) {
        self.viewModel = viewModel
        viewModel.items.observe(on: self) { [weak self] items in
            self?.tableViewController?.items = items
        }
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        bind(to: viewModel)
        viewModel.viewDidLoad()
    }
}

protocol ViewModelInput {
    func viewDidLoad()
}

protocol ViewModelOutput {
    var items: Observable<[ItemViewModel]> { get }
}

protocol ViewModel: ViewModelInput, ViewModelOutput {}

```

TableViewCell의 데이터 바인딩 예시는 다음과 같다.
```swift
final class MoviesListItemCell: UITableViewCell {
    private var viewModel: MoviesListItemViewModel! { didSet { unbind(from: oldValue)}}
    
    func fill(with viewModel: MoviesListItemViewModel) {
        self.viewModel = viewModel
        bind(to: viewModel)
    }
    
     private func bind(to viewModel: MoviesListItemViewModel) {
        viewModel.posterImage.observe(on: self) { [weak self] in self?.imageView.image = $0.flatMap(UIImage.init) }
    }
    
    private func unbind(from item: MoviesListItemViewModel?) {
        item?.posterImage.remove(observer: self)
    }
}
```
만약 뷰가 재사용가능해야 한다면 unbind를 해야 한다.

### MVVM Communication

#### Delegation
MVVM의 뷰모델은 다른화면의 뷰모델과 delegation을 통해 소통한다.

<img src="https://hackmd.io/_uploads/rJbLUv6Dn.png" width="500"/><br/>

예를 들어 `ItemListViewModel`과 `ItemEditViewModel`이 존재한다고 가정해보자. 그렇다면 `ItemEditVideModelDelegate`프로토콜을 만들어서 ListItemViewModel가 이를 채택하여 대리자가 되게 만들 수 있다. 

```swift
protocol MoviesQueryListViewModelDelegate: AnyObject {
    func moviesQueriesListDidSelect(movieQuery: MovieQuery)
}

final class DefaultMoviesQueryListViewModel: MoviesListViewModel {
    private weak var delegate: MoviesQueryListViewModelDelegate?
    
    func didSelect(item: MoviesQueryListViewItemModel) {
        delegate?.moviesQueriesListDidSlect(movieQuery: MovieQuery(query: item.query))
    }
}

extension MoviesListViewModel: MoviesQueryListViewModelDelegate {
    func moviesQueriesListDidSelect(movieQuery: movieQuery: MovieQuery) {
        update(movieQuery: movieQuery)
    }
}
```
이 델리게이트 이름을 `ItemEditViewModelResponder`라고 칭할 수도 있다.

### 정리
- 내부 레이어는 외부 레이어를 알 수 없다. 이유는 테스터블하고 유지보수에 용이한 코드를 만들기 위해. 내부 레이어가 외부 레이어를 알게 된다면? 예를 들어 CreateUseCase가 DataRepository를 직접 생성한다면 레포지토리가 변경되었을 때 CreateUseCase를 수정해야 한다. 만약 이를 의존성 주입을 통해 의존성을 역전시켰다면 주입시키는 쪽만 수정하면 되는 일인데 말이다. 그리고 테스터블하지않는 이유는 DataRepository를 직접생성한다면 목 객체를 구성하기가 어렵기 때문에 실제 외부 데이터저장소를 조작해야 할 수도 있다. 이 때문에 테스터빌리티가 떨어진다. 
- 이를 해결하기 위해 의존성 주입을 사용하는 것이다. 데이터저장소가 갖는 공통적인 프로토콜을 타입으로 받고 주입하는 쪽에서는 이 프로토콜을 채택한 실제 데이터저장소를 주입한다면 내부 레이어는 외부레이어에 대한 인터페이스만을 갖고 있기 때문에 외부 레이어를 알 수 없게 되는 상태가 되어버린다. 
- 여기까지 정리한 뒤 클린아키텍처의 원 모양의 그림을 다시 살펴보았다. 외부레이어는 내부 레이어에 대한 의존성을 갖고 있다. 예를 들어 UI는 Presenter에 대해 Presenter는 UseCase에 대해 의존성을 갖고 있다. 이 얘기는 UI혹은 뷰모델이 변경되더라도 도메인 로직(UseCase)는 영향을 받지 않아야 한다는 얘기다. 또는 DB가 변경된다고 하더라도 핵심 비즈니스 로직들은 영향을 받지 않아야 한다. UI-Presenter-UseCase에서는 이러한 의존성 관계가 잘 형성되어있다. 하지만 DataRepository-UseCase관계에서는 UseCase에서 DataRepository에 대한 의존성을 갖게 되므로 의존성흐름에 위배되기 때문에 레포지토리의 인터페이스를 의존하는 것으로 의존성을 역전시키는 것이 핵심이다. 이 내용이 첫번째 정리한 내용과 동일하다. 
- 데이터레이어가 도메인레이어에 의존하는 예시를 살펴보면 UseCase에서 Repository의 메서드를 호출한다고 가정해보자. 그런데 UseCase내부의 메서드에 파라미터가 추가되었다고 한다면 Repository에도 추가해야 할 가능성이 존재한다. 이는 데이터레이어가 도메인레이어를 의존하고 있기 때문에, 즉 의존성을 갖기 때문에 변경될 수 있는 것이다. 이 반대는 해당되지않는다. 왜냐하면 DB서비스의 구현내용이 어떻든 간의 Repository 인터페이스만을 이용해 조작하기 떄문이다. 물론 인터페이스를 변경하는 경우에는 그럴 수 있겠지만 여기서의 주안점은 DB를 관리하는 서비스의 구체적인 구현내용이 변경하든 말든 유즈케이스에서는 상관이 없다는 것이다. 만약 인터페이스가 아닌 실제 데이터를관리하는 객체의 메서드를 직접 호출한다면? 위에서 계속해서 언급했듯이 데이터저장소를 변경하려는 경우 도메인 레이어 내의 해당 디비객체를 사용하는 수많은 코드를 수정해야 하는 일이 생긴다. 그렇기 때문에 프로토콜을 하나 정의하고 이를 주입하는 것이 핵심.
- 결론은 레이어들의 관심사를 분리시켜 재사용성, 확장성, 유지보수성, 테스터빌리티를 높이는 것이다.


### 흐름 정리
- UI-ViewModel-UseCase-Repository-Service의 흐름으로 이동한다. 모든 레이어들은 UseCase에 대해 의존성을 갖고 있다. 즉 UseCase가 변경되면 전부 변경될 여지가 있고 UseCase를 제외한 레이어들이 변경되더라도 UseCase는 아무런 영향을 끼치지 않는다. 그리고 이러한 흐름이 가능한 이유는 의존성 주입 때문이다. 의존성 주입 덕분에 UseCase는 Repository에 대해 의존성을 갖지 않고 Repository Interface에 의존성을 가진다. 이는 외부레이어의 변경으로 내부레이어가 변경되지않도록 만든다.



### 참고문서
- [Clean Architecture and MVVM on iOS](https://tech.olx.com/clean-architecture-and-mvvm-on-ios-c9d167d9f5b3)

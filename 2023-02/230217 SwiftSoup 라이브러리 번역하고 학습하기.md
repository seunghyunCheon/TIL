230217 SwiftSoup 라이브러리 번역하고 학습하기
===
TIL (Today I Learned)
===
학습내용
---
- SwiftSoup 라이브러리 번역하고 학습하기


## SwiftSoup 라이브러리 번역하고 학습하기

`SwiftSoup`는 실제 HTML의 코드를 추출하고 분석하는 크로스 플랫폼 스위프트 라이브러리이다. (WHATTWG HTML5 스펙까지 지원)
- `URL`, `file`, `string` 으로부터 HTML을 스크랩하고 분석.
- `DOM traversal`과 `CSS selector`를 이용하여 데이터를 찾고 추출. (Dom traversal은 Dom을 자유자재로 횡단하는 것을 의미)
- HTML element, attributes, text를 조작
- XSS 공격을 예방하기 위해 사용자가 보낸 내용을 정리
- 정돈된 HTML을 출력. `SwiftSoup`는 wild한 상태의 HTML을 다루기 위해 고안되었고 sensible한 parse tree를 만들어낸다.

### ✏️ To parse an HTML document
```swift
do {
    let html = "<html><head><title>First parse</title></head>"
            + "<body><p>Parsed HTML into a doc.</p></body></html>"
    let doc: Document = try SwiftSoup.parse(html)
    return try doc.text()
} catch Exception.Error(let type, let message) {
    print(message)
} catch {
    print("error")
}
```
- 닫히지않은 태그(e.g. `<p>hi<p>hello`는 `<p>hi</p><p>hello</p>`로 parse된다)
- 암묵적인 태그(e.g. `<td>Table data</td>`는 `<table><tr><td>...`으로 parse된다)
- 안정적으로 문서구조 생성

#### The object model of a document
- Documents는 Element와 TextNodes로 구성되어있다.
- 상속 chain: `Documents` extends `Element` extends `Node.TextNode` extends `Node`
- 하나의 Element는 여러개의 자식 Nodes를 포함하고 자식 Element들의 필터링된 리스트들을 제공.

### ✏️ Extract attributs, text, and HTML from elements

#### Problem
document로 부터 분석한 이후 element를 찾고 데이터에 접근하고 싶은 상태

#### Solution
- attribute의 값을 얻기 위해 `Node.attr(_ String key)` 메서드를 사용
- element의 텍스트를 얻기위해 `Element.text()`를 사용
- HTML을 얻기위해 `Element.html()`나 `Node.outerHtml()`을 사용 

```swift
do {
    let html: String = "<p>An <a href='http://example.com/'><b>example</b></a> link.</p>"
    let doc: Document = try SwiftSoup.parse(html)
    let link: Element = try doc.select("a").first()!
    
    let text: String = try doc.body()!.text() // "An example link."
    let linkHref: String = try doc.link.attr("href") // "http://example.com/"
    let linkText: String = try link.text()
    
    let linkOuterH: String = try link.outerHtml() // "<a href="http://example.com/"><b>example</b></a>"
    let linkInnerH: String = try link.html() // "<b>example</b>"
} catch Exception.Erro(let type, let message) {
    print(message)
} catch {
    print("error")
}
```

#### Description
위 메서드들은 데이터 접근에 핵심이지만 추가적인 것들이 존재한다.
- `Element.id()`
- `Element.tagName()`
- `Element.className()`, `Element.hasClass(_ String className)`
이 모든 메서드들은 데이터를 변경하는 setter 메서드를 가지고 있다.

### ✏️ Parse a document from a String

#### Problem
Swift String형태의 HTML을 가지고 있고 이에 포함된 내용을 얻고 수정을 하길 원하는 상태

#### Solution
`static method`인 `SwiftSoup.parse(_ html: String)`이나 `SwiftSoup.parse(_ html: String, _ baseUri: String)`을 사용

```swift
do {
    let html = "<html><head><title>First parse</title></head>"
        + "<body><p>Parsed HTML into a doc.</p></body></html>"
    let doc: Document = try SwiftSoup.parse(html)
    return try doc.text()
} catch Exception.Error(let type, let message) {
    print("")
} catch {
    print("")
}
```

#### Description
`parse(_ html: String, _ baseUri: String)`메서드는 입력한 html을 새로운 `Document`로 parse한다. baseUri 아규먼트는 상대 URL을 절대 URL로 확인하는데 사용되며 문서를 가져온 URL로 fetch되어야 한다. 만약 HTML이 base element를 알고있는 경우에는 `parse(_ html: String)` 메서드를 사용하면 된다.

non-null String을 전달하기만 하면 `head`와 `body` element를 담은 성공적인 Document를 얻을 수 있다.

### ✏️ Parsing a body fragment

#### Problem
body HTML의 조각들을 가지고 있고 CMS안에서 사용자가 댓글을 작성하고 수정하는 것들을 parse하길 원하는 상태

#### Solution
`SwiftSoup.parseBodyFragment(_ html: String)`을 사용
```swift
do {
    let html: String = "<div><p>Lorem ipsum.</p>"
    let doc: Document = try SwiftSoup.parseBodyFragment(html)
    let body: Element? = doc.body()
} catch Exception.Error(let type, let message) {
    print(message)
} catch {
    print("error")
}
```

#### Description
`parseBodyFragment` 메서드는 빈 shell docuemnt를 만들고 parse된 Element를 body Element에 넣는다. 위 예시에서의 결과는 동일하지만 명시적으로 input값을 body fragment로 다루는 것은 사용자가 제공한 것들이 `body` element에 들어간다고 확신할 수 있게 된다. 즉 head가 포함된 html을 parse할때는 그냥 `parse`를 body만 있는 html을 parse할 때는 `parseBodyFragment`를 사용한다.

#### Stay safe
만약 사용자로부터 HTML 입력값을 접근할 것이라면 cross-site scripting 공격에 주의할 필요가있다. 만약 이를 고려한다면 `clean(String bodyHtml, Whitelist whitelist)`를 사용한다.

### ✏️ Sanitize untrusted HTML(to prevent XSS)

#### Problem
신뢰할 수 없는 사용자에게 HTML을 제공할 수 있기 때문에 XSS 공격을 예방하기 위해 HTML을 clean할 필요가 있다.

#### Solution
`whitelist`의 옵션값과 함께 `Cleaner`를 사용한다.
```swift
do {
    let unsafe: String = "<p><a href='http://example.com/' onclick='stealCookies()'>Link</a></p>"
    let safe: String = try SwiftSoup.clean(unsafe, Whitelist.basic())!
} catch Exception.Error(let type, let message) {
    print(message)
} catch {
    print("error")
}
```

#### DisCussion
필요할 때 학습 [SwiftSoup](https://github.com/scinfu/SwiftSoup)


### ✏️ Set attribute values

#### Problem
디스크에 저장하거나 HTTML 응답을 보내기전에 parsed된 document를 수정하려고 하는 상태

#### Solution
attribute setter 메서드를 사용한다. `Element.attr(_ key: String, _ value: String)`, `Elements.attr(_ key: String, _ value: String)`

만약 element의 클래스 attribute를 수정하고 싶다면 `Element.addClass(_ className: String)`이나 `Element.removeClass(_ className: String)` 메서드를 사용한다.

```swift
do {
    try doc.select("div.comments a").attr("rel", "nofollow")
} catch Exception.Error(let type, let message) {
    print(message)
} catch {
    print("error")
}
```

#### Description
`Element`의 다른 메서드들 처럼 attr 메서드는 현재 `Element`를 반환한다. 즉 체이닝해서 사용할 수 있다.
```swift
try doc.select("div.masthead").attr("title", "swiftsoup").addClass("round-box")
```

### ✏️ Set the HTML of an element

#### Problem
HTML을 수정하고 싶은 상태

#### Solution
`Element`에서 setter 메서드를 사용한다.

```swift
do {
    let doc: Document = try SwiftSoup.parse("<div>One</div><span>One></span")
    let div: Element = try doc.select("div").first()! // <div>One</div>
    try div.html("<p>lorem ipsum</p>") // <div><p>loem ipsum</p></div>
    try div.prepend("<p>First</p>")
    try div.append("<p>Last</p>")
    print(div)
    // now div is: <div><p>First</p><p>lorem ipsum</p><p>Last</p></div>
    
    let span: Element = try doc.select("span").first()!
    try span.wrap("<li><a href='http://example.com/'></a></li>")
    print(doc)
    // now: <html><head></head><body><div><p>First</p><p>lorem ipsum</p><p>Last</p></div><li><a href="http://example.com/"><span>One</span></a></li></body></html>            
} catch Exception.Error(let type, let message) {
    print(message)
} catch {
    print("error")
}
```
중요한 사실은 `Element`의 setter 메서드를 사용하든 `attribute`의 setter를 사용하든 기준이 되는 doc프로퍼티가 변화한다는 것이다. 

#### Discussion
- `Element.html(_ html: String)`은 존재하는 inner HTML을 비우고 새로운 parse된 HTML로 대체한다.
- `Element.prepend(_ first: String)`, `Element.append(_ last: String)` 메소드들은 맨 앞에 HTML을 추가하거나 맨 뒤에 HTML을 추가하는 메서드이다.
- `Element.wrap(_ around: String)`는 외부 HTML로 감싼다.

#### See also
새로운 element를 추가하고 자식 element로 document에 삽입하기 위해 `Element.prependElement(_ tag: String)`, `Element.appendElement(_ tag: String)`메서드를 삽입하기도 한다.


### ✏️ Setting the text content of elements

#### Problem
HTML document내부의 텍스트를 수정하려고 하는 상태

#### Solution

`Element`의 text setter 메서드들을 사용한다.
```swift
do {
    let doc: Document = try SwiftSoup.parse("<div></div>")
    let div: Element = try doc.select("div").first()! // <div></div>
    try div.text("five > four") // <div> five &gt; four</div>
    try div.prepend("First ")
    try div.append(" Last")
    // now: <div>First five &gt; four Last</div>
} catch Exception.Error(let type, let message) {
    print(message)
} catch {
    print("error")
}
```

`Element`에서 `div.html()`을 사용해 `Element`자체를 수정할 때는 `String`형태로 반환하지만 `Element`내부의 텍스트를 수정할 때는 `Element`타입을 반환한다. 
그렇기에 `div.text()` 메서드는 `Element`가 확정된 상태이기 때문에 내부 `>`와 같은 문자는 `&gt`로 문자그대로 처리가 된다.


### ✏️ Use Dom methods to navigate a document

#### Problem
HTML document 구조를 알고있는 상태에서 특정 데이터를 추출하고 싶은 상태.

#### Solution
HTML을 Document로 parse한 후에 `DOM-like` 메서드들을 사용한다.

```swift
do {
    let html: String = "<a id=1 href='?foo=bar&mid&lt=true'>One</a> <a id=2 href='?foo=bar&lt;qux&lg=1'>Two</a>"
    let els: Elements = try SwiftSoup.parse(html).select("a")
    for link: Element in els.array() {
        let linkHref: String = try link.attr("href")
        let linkText: String = try link.text()
    }
} catch Exception.Error(let type, let message) {
    print(message)
} catch {
    print("error")
}
```

여러 메서드들은 필요할 때 참조 [SwiftSoup](https://github.com/scinfu/SwiftSoup)


### ✏️ Use selector syntax to find elements

#### Problem
CSS나 jQuery와 같은 selector 문법을 이용하여 element를 찾고 조작하고 싶은 상태

#### Solution
`Element.select(_ select: String)` 메서드를 사용한다.

```swift
do {
    let doc: Document = try SwiftSoup.parse("...")
    let links: Elements = try doc.select("a[href]") // a with href
    let pngs: Elements = try doc.select("img[src$=.png]")
    // img with src ending .png
    let masthead: Element? = try doc.select("div.masthead").first()
    // div with class=masthead
    let resultLinks: Elements? = try doc.select("h3.r > a") // direct a after h3
} catch Exception.Error(let type, let message) {
    print(message)
} catch {
    print("error")
}

```

#### 참고문헌
[SwiftSoup git](https://github.com/scinfu/SwiftSoup)

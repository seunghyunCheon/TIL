230301 HIG - Typography
===
학습내용
---
- HIG - Typography 읽기

### HIG - Typography

타이포그래피는 읽기 쉬운 텍스트를 제공하는 것 뿐만 아니라 정보 계층을 명시하고, 중요한 내용을 잘 전달하며 브랜드를 표현할 수 있게 만든다.

### ✏️ Best practices

### 많은 사람들이 쉽게 읽을 수 있는 최소 폰트사이즈를 유지한다.

디바이스 화면들의 차이는 픽셀밀도와 밝기와 같은 요인들로 적절한 최소폰트사이즈 결정에 영향을 줄 수 있다. 다른 요인으로는 독자가 디바이스에 얼마나 근접해있는지, 시력이 어떤지, 움직이고 있는지, 환경적으로 빛이 있는 조건에 있는지 등이 있다.

다이나믹 타입을 설정한다면 사람들이 정한 텍스트 크기로 앱을 볼 수 있기 때문에 적절하게 반응할 수 있게 된다.

### 중요한 정보를 강조하고 싶을때 폰트의 weight, size, color를 조정하여 시각적인 계층을 제공한다.

사람들이 텍스트 사이즈를 조정할 때 상대적인 계층을 유지하고 텍스트 요소의 시각적인 구분을 유지해야 한다.

### 인터페이스 안에서 사용하는 서체의 종류를 최소화한다.
너무 많은 서체의 혼합은 정보계층을 모호하게 만들고 가독성을 줄인다.

### 다른 문맥에서 가독성을 테스트한다.
예를 들어 텍스트 크기를 조정하는 것에 더하여 사람들은 밝은 곳에서 볼 수도있꼬 움직이면서 볼 수도 있고 멀리서 볼 수도 있다. 만약 테스트했을때 글자를 보기 어렵다면 text나 백그라운드를 수정하여 대조를 증가시킨다. 혹은 더 큰 타입 사이즈를 사용하거나 system 폰트와 같은 가동성에 최적화된 타입서체를 사용한다.

### 일반적으로 가독성을 위해 얇은 폰트는 피한다.
시스템 폰트를 사용한다면 Regular, Medium, Semibold, Bold를 사용하고 UItralight, Thin, Light는 피한다. (특히 텍스트 크기가 작을때)

### 텍스트 크기가 변할때 중요한 내용을 우선순위로 정한다.
모든 내용이 중요하지는 않다. 사용자가 더 큰 텍스트 사이즈를 선택할 때 모든 내용을 강조하기를 원하지는 않을 것이다. 예를 들어 메일의 경우 제목과 내용은 크게 보여지고 보낸사람과 날짜와 같은 부분들은 작은 사이즈를 주는 것이 좋을 것이다.

### ✏️ 시스템 폰트 사용

애플은 확장된 weight, size, style, language를 지원하는 두 가지 타입서체가 존재한다.

San Francisco (SF)는 SF Pro, SF Compact, SF Arabic, SF Mono를 포함하는 sans serit 타입 서체이다. 

New York(NY)는 SF font와 함께 잘 작동되도록 고안된 타입서체이다.

시스템은 다양한 폰트 포멧 안에서 SF와 NY폰트를 제공하는데 이는 하나의 파일안에 다른 폰트스타일들이 결합되어있다. 그리고 스타일 간 보간을 지원하여 중간 스타일을 만든다.

### 👏 Note
> 가변 글꼴은 다양한 사이즈에 맞게 다양한 타이포그래피의 조정하는 시각적 크기조정을 가능하게 한다. 시스템 폰트는 각 글리프와 문자형태를 point size에 맞게 만드는 동적 광학 크기를 사용하기 때문에 개별광학 크기를 사용할 필요가 없다. 

시각적 계층을 보여주고 문맥을 명확하게 이해하기 위해 시스템 폰트는 UITralight부터 Black까지 다양한 두께를 제공하고, Condensed부터 Expanded까지 제공한다. 
SF Symbol들은 동일한 두께를 사용하기 때문에 직접 스타일을 준 것과 관계없이 symbol과 인접한 텍스트 사이에 정확한 weight를 줄 수 있다.

### 👏 Note
> SF Symbol들은 SF 폰트에 자동적으로 두께와 사이즈를 맞추면서 원할하게 통합되는 포괄적인 라이브러리이다. 텍스트 내에서 개념을 전달하거나 묘사할 필요가 있을 때 사용을 고려한다.

text Style이라고 불리는 시스템은 타이포그래피 속성의 집합이다. 이는 두가지 타입서체와 함께 동작한다. 
text Style은 폰트의 weight, point size, 각 텍스트 사이즈에 대한 leading value들의 결합이다. 예를 들어 body text Style은 여러 줄의 텍스트를 읽기 편하게 지원하는 반면에 headline Style은 내용으로부터 heading을 구별할 수 있게 도와준다. 
이들을 같이 사용하면 다른 level의 내용을 표현하는데 사용할 수 있는 타이포그래피 계층을 형성한다. 그리고 사람들이 시스템 텍스트의 사이즈를 조정하거나 accessibility 조정을 할 때 그에 비례하여 텍스트를 확장시킨다. 

### ✏️ built-in text Style 사용을 고려한다. 

시스템 정의된 text Style은 정보를 계층적으로 만드는데 편하고 일관적인 방법을 제공한다. 그리고 Dynamic Type을 지원하기에 사람들이 텍스트 사이즈를 선택할 수 있다.

### ✏️ 필요하다면 built-in text Style을 수정한다.
시스템 API는 텍스트 스타일의 일부 측면을 수정할 수 있는``symbolic traits``이라고 하는 글꼴 조정을 정의합니다. 예를 들어 bold trait은 텍스트에 weight를 더해 계층 레벨을 만들게 한다. 그리고 symbolic traits를 사용해서 행간을 줄일 수도 있다. 그리고 만약 가독성을 높이고 공간을 절약하고 싶다면 symbolic traits를 이용해 행간을 조절할 수 있다. 예를 들어 넓은 열이나 긴 구절에 텍스트를 보여주고자할때 행간의 공간을 더 많이 준다면 사람들이 다음줄로 이동할 때 인식하기 쉽게 만든다. 반대로 높이가 제한된 공간에서 여러줄을 사용할 경우 행간을 줄이면 도움을 줄것이다.
줄의 수가 세 줄을 넘어갈 경우 행간을 너무 타이트하게 설정하는 것은 피하는 게 좋다.

### ✏️ 인터페이스 모형에서 필요에따라 추적을 조정한다.
시스템 폰트는 모든 포인트 크기에서 동적으로 조정한다. 

### ✏️ Using cusing fonts

### 커스텀 폰트가 잘 읽히도록 한다.
앱이 브랜드를 나타내거나 몰입할 수 있는 경험을 주기위해 커스텀 폰트를 꼭 써야하는 것이 아니라면 시스템 폰트를 사용한다. 만약 커스텀 폰트를 사용한다면 다양한 환경에서 테스트를 하는 것이 좋다.

### 커스텀 폰트에 대해 접근성을 구현한다.
시스템 폰트는 자동적으로 Dynamic Type을 지원해 접근성이 존재한다. 커스텀 폰트를 텍스트에 적용할 때 시스템폰트와 같은 행동을 하기위해 다음의 문서를 참고한다. [Applying custom fonts to text.](https://developer.apple.com/documentation/swiftui/applying-custom-fonts-to-text))

### 참고자료
- [HIG - typography](https://developer.apple.com/design/human-interface-guidelines/foundations/typography/)

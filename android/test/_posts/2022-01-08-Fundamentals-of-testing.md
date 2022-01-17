---
title: 안드로이드 앱 테스트 코드 기초
layout: single
---

### 테스트의 이점
---
개발 프로세스에 있어 테스트는 필수적인 부분이다. 너의 앱에 대해서 계속 테스트를 돌린다는것은 너의 앱의  정확성과 기능적인 부분 그리고 출시하기전 사용성등을 확인해볼수 있다.

테스트하기 위해 본인이 직접 여러기기들로 앱을 탐색하고 시스템 언어 변경, 에러 만들기, 유저 흐름등을 만들수 있지만 수동 테스트 한다는건 확장성이 좋지않고 앱의 회귀를 간과하기 쉬울수 있다.
자동화 된 테스트는 도구를 사용해 나를 대신해 빠르고 반복적인 테스트를 수행하고 개발상태에서 더 빨리 
실행가능한 피드백을 준다.

### 안드로이드 테스트 유형
---
####  주제 (Subject)
주제에 따른 여러 유형의 테스트가 있다.

- 기능적 테스트 : 앱이 제대로 작동하는가?
- 성능 테스트 : 앱이 빠르고 효과적인가?
- 접근성 테스트: 접근성 서비스에 대해 잘 동작하는가?
- 호환성 테스트 : 모든 기기와 API 레벨에 대해 잘 동작하는가?

####  범위 (Scope)
테스트는 격리 정도나 범위에 따라 다르다.

- 단위테스트 (unit test) 또는 작은 테스트 (small tests) 메소드하나나 클래스 처럼 앱의 작은 부분을 검증한다. 

- End to End 테스트 또는 큰 테스트 (big tests) 전체화면이라던가 유저 흐름같은 앱의 큰 영역을 검증한다.

- 통합테스트 (Medium tests) 는 두개나 그 이상의 유닛들을 통합시켜서 검증한다. (단위들의 모음등)

{% include image.html id="15WmG6uAngMyF2bZkxuW_Idn6VMF_DmfB" %}

### 계측 테스트 vs 로컬 테스트
---
당신은 테스트를 안드로이드 기기나 다른 컴퓨터를 통해 할수있다

- 계측테스트 (instumented tests) : 실제 안드로이드 기기나 에뮬레이터를 통해 테스트하는 것으로 앱을 띄워
상호작용 등을 테스트 하는 UI 테스트다.

- 로컬테스트 (local tests) : 개발기기 (내 컴퓨터가 될수도 있고 아님 서버등) 로 수행하는 테스트로
보통 작고 빠르며 앱의 남은 부분을 테스트 중인 대상과 분리한다.

{% include image.html id="14VYtAGvWisXuRbLDBd7tr-tv5qrIJi86" %}

단 주의해야할것은 모든 단위테스트가 로컬 테스트는 아니고 모든 e2e (end to end) 테스트가 기기에서 동작하는건 아니다. 예를들면

- Big Local Test : [Robolectric](http://robolectric.org/) 을 사용해 지역적으로 테스트를 할수 있다.
- Small Instrument Test : SQLite DB 기능이 잘 동작하는지 테스트 하기 위해 여러 기기에서 여러 버전의 SQLite 테스트를 확인해볼수도 있다. 

#### 계측 테스트 예제
```java
// When : Continue 버튼 클릭
onView(withText("Continue"))
    .perform(click())

// Then : Welcome 화면 노출됬는지 확인
onView(withText("Welcome"))
    .check(matches(isDisplayed()))
```

위의 코드는 Espresso 를 사용해  UI가 계측테스트에서 잘 상호작용하는지 테스트 한코드다. 버튼을 클릭하고 실제 버튼에 값이 표시됬는지 체크해볼수 있다.

#### 로컬 테스트 예제
```java
// Given : 뷰 모델 인스턴스 획득
val viewModel = MyViewModel(myFakeDataRepository)
// When : 데이터 로드됨
viewModel.loadData()
// Then : 데이터가 들어왔는지 확인
assertTrue(viewModel.data != null)
```

### 테스트 전략 정의하기
---
모든 코드에 대해 앱이 호환되는 모든 기기로 테스트하는것이 가장 이상적이라고 할수 있지만 실제로 그런 접근은
너무 느리고 비용이 많이 든다.  

좋은 테스트 전략은 충실도,속도,신뢰성 등에서 적절한 균형을 찾는것이다. 실제 기기와 유사한 테스트 환경은 
충실도를 높이지만 너무 느리고 많은 자원이 들어가고 반면에  개발기기의 JVM 환경에서 동작하는 테스트는  빠르지만 충실도가 낮다.

##### 불안정한 테스트 (Flaky tests)
정확하게 설계하고 수행하는 테스트에서도 에러는 발생할 수 있다. 예를 들면 실제 기기에서 테스트중에
자동으로 업데이트 팝업이 뜰경우 오류가 발생한다거나 코드에서 미묘하게 경쟁 상황을 걸어둿을때 낮은 확률로
오류가 발생할수도 있다. 즉 100프로 통과 못하는 테스트는 불안정하다.

### 테스트 가능한 아키텍쳐
---
테스트 가능한 아키텍쳐는 분리를 통해 각각 다른 파트들을 테스트 하기 쉽게 만들어준다. 게다가 더 읽기 쉬워지고
유지보수도 용이하며 재사용성과 확장성이 높아진다.

##### 디커플링 접근방법
만약 함수,클래스 또는 모듈등의 일부분을 추출할수 있다면 테스트는 더 쉬워지고 효과적이 된다.
디커플링 (decoupling) 이라 불리는 이 개념은 테스트 가능한 아키텍쳐에서 가장 중요하다.
보통의 디커플링 기술은 아래와 같다.

- Presentation , Domain , Data 등으로 영역을 나누고 앱 안의 모듈을 기능단위로 분리한다.

- 액티비티나 프래그먼트처럼 종속석이 큰 개체에 로직 추가는 피하자. 해당 구성요소는 앱의 진입점으로만 사용하고 ViewModel 이나 Compasable 이나 Domain 영역으로 UI 나 비지니스로직을 옮겨라.

- 비지니스로직을 포함한 클래스에서 직접 프레임워크 의존성을 가지는걸 피하자. ViewModel 에서 Android Context 를 사용하지 말라.

- 의존성이 교체가 쉬워야한다. 예를들어 구체화된 클래스보단 interface 를 사용하고 DI 프레임워크를 사용하지 않더라도 DI 는 사용하자.

##### 피해야할 접근방식
테스트 할수 없는 아키텍쳐는 다음의 문제를 만든다.

- 크고 느리고 불안정하다 . 단위테스트를 할수 없는 클래스는 더 큰 통합테스트나 UI 테스트에서 보완할수 밖에없다.
- 다른 시나리오에 대해 테스트할 기회가 거의 없다. 더 큰 테스트는 느리기 때문에 앱의 모든 가능한 상태를 
테스트 한다는건 비현실적이다.

즉 테스트가능한 아키텍쳐를 사용함으로 유지보수 및 확장성 좋은 코드를 작성해 테스트를 해야한다.


### 참조
---
[Fundamentals of testing Android apps](https://developer.android.com/training/testing/fundamentals)
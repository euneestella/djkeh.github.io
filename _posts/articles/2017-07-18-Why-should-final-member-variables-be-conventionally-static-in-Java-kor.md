---
layout: post
categories: articles
title:  "왜 자바에서 final 멤버 변수는 관례적으로 static을 붙일까?"
excerpt: "자바 final, static 키워드와 코딩 best practice 되짚어보기"
tags: [java]
date: 2017-07-18 23:19:51
last_modified_at: 2019-11-05 02:13:36
---

오늘도 기초 정리입니다! 오늘은 자바 개발에서 꽤나 보편적으로 볼 수 있는 코드를 하나 정리해보려고 합니다. 바로 클래스의 `private static final` 멤버 변수 이야기입니다. 저는 현업에서 클래스 상수나 로그 구현체를 이런 식으로 만드는 것을 자주 보았는데요, 관련한 내용들을 가능한 짧고 간단하게 메모 스타일로 정리해 보겠습니다.


## 왜 final 변수는 꼭 static 인가

흔히 클래스의 멤버 변수를 상수(`final`)로 만들고자 할 땐, 클래스 상수(`static final`)로 만들어주곤 합니다. 사실 이 말 속에 답이 대략 나타나는 것 같은데요^^; 하지만 이참에 자바 기본을 정리해 보죠.


## final 키워드

`final` 키워드는 프로그래밍 언어에서 'constant', '상수'와 같은 단어와 비교되는 단어인데요, 위키피디아에 따르면 자바에서 기본적으로 다음의 의미를 가집니다.

> final은 해당 entity가 오로지 한 번 할당될 수 있음을 의미합니다.

그래서 위 의미를 바탕으로 다음 세 경우에 따라 구체적으로 작용합니다.

* final 변수
  * 해당 변수가 생성자나 대입연산자를 통해 한 번만 초기화 가능함을 의미합니다. 상수를 만들 때 응용합니다.
* final 메소드
  * 해당 메소드를 오버라이드하거나 숨길 수 없음을 의미합니다.
* final 클래스
  * 해당 클래스는 상속할 수 없음을 의미합니다. 문자 그대로 상속 계층 구조에서 '마지막' 클래스입니다.
  * 보안과 효율성을 얻기 위해 자바 표준 라이브러리 클래스에서 사용할 수 있는데, 대표적으로 `java.lang.System`, `java.lang.String` 등이 있습니다.

### 몇가지 세부 분석

#### 1. final 멤버 변수가 반드시 상수는 아닙니다

왜냐면 `final`의 정의가 '상수이다'가 아니라 '한 번만 초기화 가능하다'이기 때문입니다. 다음의 코드를 보시죠.

```java
public class Test {
  private final int value;

  public Test(int value) {
    this.value = value;
  }

  public int getValue() {
    return value;
  }
}
```

이 코드에서 final 멤버 변수 `value`는 생성자를 통해 초기화 되었습니다. 즉 이 클래스의 인스턴스들은 각기 다른 `value` 값을 갖게 되겠죠. 각 인스턴스 안에서는 변하지 않겠지만, 클래스 레벨에서 통용되는 상수라고는 할 수 없습니다.

#### 2. private 메소드와 final 클래스의 모든 메소드는 명시하지 않아도 `final` 처럼 동작합니다

왜냐면 오버라이드가 불가능하기 때문이죠.

* 참고: [jls-8.4.3.3](http://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html#jls-8.4.3.3)

하지만 private 메소드에 여전히 `final` 명시는 가능합니다. 불필요하냐구요? 네 사실 그렇습니다 ㅎㅎ 그래도 일단 의미는 구분됩니다.

* private: 자식 클래스에서 안 보입니다. (오버라이드도 물론 금지입니다.)
* final: 자식 클래스에서 보이지만, 오버라이드가 금지됩니다.

이렇게 불필요한 명시를 그렇다고 특별 취급해서 막을 필요는 없기 때문에, 컴파일러가 에러를 내거나 경고하지는 않습니다. 비슷한 예로 인터페이스의 메소드에 `public`을 붙이거나, final 클래스 메소드에 `final`을 붙이는 등의 경우도 문제가 생기지는 않지요. 

그렇다면 private 메소드와 final 메소드는 inline 메소드로 컴파일러 최적화가 될까요? 확인해보진 않았습니다만, 가능은 하더라도 항상 보장되진 않을 것으로 보입니다. 최적화 동작은 컴파일러와 내부 세팅에 의해 일어나기 때문입니다.


## static 키워드

`static` 키워드는 프로그래밍 언어에서 '전역', '정적'의 의미로 통용됩니다.

> static은 해당 데이터의 메모리 할당을 컴파일 시간에 할 것임을 의미합니다.

이에 `static` 데이터는 런타임 중에 필요할 때마다 동적으로 할당 및 해제되는 동적 데이터와는 기능과 역할이 구분됩니다. 동적 데이터와 달리, `static` 데이터는 프로그램 실행 직후부터 끝날 때까지 메모리 수명이 유지됩니다.

자바에서는 구체적으로 다음과 같이 작용합니다.

* static 멤버 변수
  * 클래스 변수라고도 부릅니다.
  * 모든 해당 클래스는 같은 메모리를 공유합니다.
  * 특정한 인스턴스에 종속되지 않습니다.
  * 인스턴스를 만들지 않고 사용 가능합니다.
* static 메소드
  * 클래스 메소드라고도 부릅니다.
  * 오버라이드 불가합니다.
  * 상속 클래스에서 보이지 않습니다.
* static 블록
  * 클래스 내부에 만들 수 있는 초기화 블록입니다.
  * 클래스가 초기화될 때 실행되고, `main()` 보다 먼저 수행됩니다.
* static 클래스
  * 일반적인 클래스, 즉 top-level 클래스에 적용하면 문법 오류입니다.
    * 그러나 이것이 top-level 클래스가 `static`하지 않다는 뜻이 아닙니다.
  * 중첩 클래스(nested class)에만 사용할 수 있습니다.
    * static nested class: `static`으로 정의된 nested class
    * inner class: `static`으로 정의되지 않은(non-static) nested class
  * 부모 클래스의 멤버 필드 중에는 `static` 필드에만 접근할 수 있습니다.
  * 사실상 일반적인 top-level 클래스와 동일하게 동작하지만, 그 위치가 하나의 top-level 클래스 안에 들어있는 것입니다.
    * 이것은 유사한 클래스 집합을 하나로 묶고, 클래스 패키징 구조를 편리하게 정리하는 테크닉으로 사용될 수 있습니다.
* static import
  * 다른 클래스에 존재하는 static 멤버들을 불러올 때 사용합니다.
  * 멤버 메소드를 곧바로 사용할 수 있습니다.


## 그래서 왜 static final이라고? - 클래스 멤버 변수를 final로 지정하는 의도의 확인

그것은 클래스에서 사용할 해당 멤버 변수의 데이터와 그 의미, 용도를 고정시키겠다는 뜻이겠죠. 해당 클래스를 쓸 때 변하지 않고 계속 일관된 값으로 쓸 것을 멤버 상수로 지정할 것입니다. 예를 만들어 볼까요?

* `기독교` 클래스에서 멤버 변수 `신의 이름`을 만들어 사용한다면 해당 클래스를 언제 어디서 어떻게 쓰든 변함없이 `하나님`이겠죠?
* `중학교 성적` 클래스에서 `과목 최대 점수` 변수를 만든다면 `100`일 것입니다.

이 값들은 모든 클래스 인스턴스에서 똑같이 써야할 값이고, 프로그래머는 이들을 프로그램 처음부터 끝까지 바뀌지 않는 논리로 의도할 것입니다. 그렇다면 **인스턴스가 만들어질 때마다 새로 메모리를 잡고 초기화시키지 말고**, 클래스 레벨에서 한 번만 잡아서 하나의 메모리 공간을 쭉 쓰면 되지 않을까요? 그렇게 하면 어차피 다 같은 값을 가질 데이터를 위해 인스턴스 생성마다 매번 같은 메모리를 잡는 것보다 더 효율적일 것입니다. 상수로 만들 의도였으니 동시성 문제도 없고요. 그렇다면~

```
public static final String NAME_OF_GOD = "하나님";
public static final int MAX_SUBJECT_SCORE = 100;
```

와 같이 만들어두면 효율적이겠죠~

이것이 코딩 관례가 되어, 멤버 상수는 `static final`로 만드는 practice가 생겼다고 볼 수 있을 것 같습니다.


## (사족1) final 멤버 변수에 static을 사용하지 않는 경우가 있을까

위에서 잠시 언급한 것처럼, 각 인스턴스마다 서로 다른 final 멤버 변수를 생성자에서 초기화시키는 식으로 사용하는 경우에는 `static`을 사용하지 않겠네요! 즉, 인스턴스를 생성할 때 한 번만 초기화하고 쭉 변화 없이 사용할 내용이라면 아주 잘 어울릴 것 같습니다.

그런 경우가 있냐고요?
실무에서, 매우, 있습니다.

DI(Dependency Injection) 기법을 사용해 클래스 내부에 외부 클래스 의존성을 집어넣는 경우가 그것입니다. 대표적으로 [Spring Framework](https://spring.io/projects/spring-framework)가 있습니다. 코드 예제로 살펴볼까요?

```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

위 소스코드는 스프링 [공식 문서](https://docs.spring.io/spring/docs/5.2.1.RELEASE/spring-framework-reference/core.html#beans-autowired-annotation)에 포함된 예제를 그대로 가져온 것입니다. `MovieRecommender` 클래스가 `CustomerPreferenceDao`를 `private final` 멤버 필드로 가지고 있으며, 생성자를 통해 주입받아 한 번 초기화되고 있습니다. 이제 `MovieRecommender`의 인스턴스는 작동 내내 변하지 않는 `customerPreferenceDao` 멤버 필드를 사용하게 될 것입니다. 코드를 풀이하여 읽어본다면, *"영화 추천 클래스는 소비자 선호도 자료에 접근하는 외부 기능을 가져다 사용하고 있다(소비자 선호도 데이터 접근 기능에 의존성이 있다)"* 정도가 되지 않을까요?

이는 "영화 추천" 기능과 "소비자 선호도 자료 접근" 기능이 서로 독립적이며, "영화 추천" 기능 사용 중에 "소비자 선호도 자료 접근" 기능이 바뀌지 않을 것임을 의미합니다. 복잡한 기능을 갖춘 소프트웨어를 디자인할 때 이러한 설계가 아주 중요합니다. 위와 같은 상황에서는 오히려 `CustomerPreferenceDao` 멤버 필드를 `static`으로 만들지 않습니다. 멤버 필드로 의존성 주입을 표현할 때 `private static final`로 하는 것이 왜 적절하지 않은지, 실제로 스프링 프레임워크 상에서 사용해보면 어떤 오류가 발생하는지는 여기서 다루지 않겠습니다.

이것이 DI 입니다.


## (사족2) static 멤버 변수에 final을 사용하지 않는 경우가 있을까

이 역시 기술적으로 충분히 가능합니다. 명확한 목적이 있는 경우는 사용할 수 있을 겁니다. 하지만 보통의 경우엔 좋은 코딩 관례(practice)로 보기 어려울 듯 합니다. `static` 필드는 클래스 스코프(범위)의 전역 변수라 볼 수 있습니다. `final`을 쓰지 않았다면 값이 얼마든지 바뀔 수 있는 상태이므로, 이를 mutable 하다고 말합니다. 이는 모든 클래스 인스턴스에서 접근하여 그 값을 변경할 수 있음을 의미하므로, 값을 추론하거나 테스트하기 어렵게 만들 것입니다. 또한 동시성 프로그래밍을 어렵게 만드는 요인이 되겠죠.


## Reference

* [https://stackoverflow.com/questions/7026507/why-are-static-variables-considered-evil](https://stackoverflow.com/questions/7026507/why-are-static-variables-considered-evil)
* [https://stackoverflow.com/questions/1415955/private-final-static-attribute-vs-private-final-attribute](https://stackoverflow.com/questions/1415955/private-final-static-attribute-vs-private-final-attribute)
* [http://www.javaworld.com/article/2077399/core-java/private-and-final.html](http://www.javaworld.com/article/2077399/core-java/private-and-final.html)
* [https://en.wikipedia.org/wiki/Final_(Java)](https://en.wikipedia.org/wiki/Final_(Java))
* [https://en.wikipedia.org/wiki/Static_(keyword)](https://en.wikipedia.org/wiki/Static_(keyword))
* [http://ojava.tistory.com/50](http://ojava.tistory.com/50)
* [https://www.baeldung.com/java-static](https://www.baeldung.com/java-static)
* [https://ko.wikipedia.org/wiki/정적_변수](https://ko.wikipedia.org/wiki/정적_변수)
* [https://docs.spring.io/spring/docs/5.2.1.RELEASE/spring-framework-reference/core.html#beans-autowired-annotation](https://docs.spring.io/spring/docs/5.2.1.RELEASE/spring-framework-reference/core.html#beans-autowired-annotation)

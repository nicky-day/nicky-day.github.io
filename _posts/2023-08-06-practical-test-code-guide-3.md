---
layout: single
title: "테스트 코드는 왜 필요하고 어떻게 작성해야 하는가?(3)"
categories: Test-Code
tag: [JUnit5, TDD, Mockito]
author_profile: false
sidebar:
    nav: "counts"
search: true
use_math: false
---



## 7. 더 나은 테스트 코드 작성법

- 한 문단에 한 주제
  - 한 가지 테스트에서는 한 가지 목적의 검증만 수행해야 한다. 즉, DisplayName을 한 문장으로 구성할 수 있어야 한다. 또한 반복문이나 논리문같은 논리구조가 들어가지 않은 형태로 구성하는 것이 좋다.
- 완벽하게 제어하기
  - 제어할 수 없는 값은 상위구조로 분리해서 테스트 가능한 코드를 만들어라.(ex : LocalDateTime.now())
  - 외부 시스템은 Mocking 처리한다.
- 테스트 환경의 독립성을 보장하자
  - 다른 모듈을 참조해서 테스트간 결합도가 생길 수 있다. 그러면 테스트코드를 이해할 때 다른 모듈까지 확인해야 하는 허들이 생기고 다른 모듈에 문제가 생길 경우 영향을 받을 수 있다. 따라서 given 절을 구성할 때는 순수한 생성자나 빌더로 구성해야 한다.
- 테스트 간 독립성을 보장하자.
  - 각각 독립적으로 실행해도 항상 같은 결과가 나와야 한다. 따라서 공유자원을 사용하거나 순서에 의존하는 테스트 코드를 작성해서는 안된다.
- 한 눈에 들어오는 TestFixture 구성하기
  - TestFixture를 확인하기 위해 다른 파일이나 메서드를 참고해야 하는 번거로움을 없애야 한다.
- Test Fixture 클렌징
  - @DeleteAll 보다는 @DeleteAllInBatch를 사용하라.
- 테스트 수행도 비용이기 때문에 수행 환경을 통합하라
  - IntegrationTestSupport Abstract Class
- Private 메소드를 테스트하고 싶어진다면 객체 분리의 신호임을 의심해라.
- 프로덕션에는 필요 없는데 테스트에만 필요한 메서드가 있다면(ex : Request의 Builder) 만들어도 좋지만 보수적으로 접근해야 한다.



@Parameterized Test 

한 개의 메소드에 대해 여러 개의 테스트를 수행해야 하는 경우에 사용한다. 하나의 테스트 메소드로 여러 가지 파라미터에 대해 테스트할 수 있다.

https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests

```java
@DisplayName("상품 타입이 재고 타입인지를 체크한다.")
@CsvSource({"HANDMADE,false", "BOTTLE,true", "BAKERY,true"})
@ParameterizedTest
void containsStockType4(ProductType productType, boolean expected) {
    // when
    boolean result = ProductType.containsStockType(productType);

    // then
    assertThat(result).isEqualTo(expected);
}

private static Stream<Arguments> provideProductTypesForCheckingStockType() {
    return Stream.of(
            Arguments.of(ProductType.HANDMADE, false),
            Arguments.of(ProductType.BOTTLE, true),
            Arguments.of(ProductType.BAKERY, true)
    );
}

@DisplayName("상품 타입이 재고 관련 타입인지를 체크한다.")
@MethodSource("provideProductTypesForCheckingStockType")
@ParameterizedTest
void containsStockType5(ProductType productType, boolean expected) {
    // when
    boolean result = ProductType.containsStockType(productType);

    // then
    assertThat(result).isEqualTo(expected);
}
```



@DynamicTest

어떤 환경을 설정해주고 환경을 공유하면서 시나리오를 테스트해보고 싶을 때 사용할 수 있다. Runtime 중에 생성되는 동적 테스트이며, given 안에서 when이 연속적으로 이루어지는 형태를 가진다.

```java
@DisplayName("멀티맵 기능 확인")
@TestFactory
Collection<DynamicTest> multiMapLearningTest() {
    // given
    Multimap<String, String> multimap = ArrayListMultimap.create();
    multimap.put("커피", "아메리카노");
    multimap.put("커피", "카페라떼");
    multimap.put("커피", "카푸치노");
    multimap.put("베이커리", "크루아상");
    multimap.put("베이커리", "식빵");

    return List.of(
      	DynamicTest.dynamicTest("1개 value 삭제", () -> {
            // when
            multimap.remove("커피", "카푸치노");

            // then
            Collection<String> results = multimap.get("커피");
            assertThat(results).hasSize(2)
            		.isEqualTo(List.of("아메리카노", "카페라떼"));
        }),
        DynamicTest.dynamicTest("1개 key 삭제", () -> {
            // when
            multimap.removeAll("커피");

            // then
            Collection<String> results = multimap.get("커피");
            assertThat(results).isEmpty();
        })
    );
}      
```



Spring REST Docs

https://docs.spring.io/spring-restdocs/docs/current/reference/htmlsingle/

https://asciidoctor.org/

- 테스트 코드를 통한 API 문서 자동화 도구
- API 명세를 문서로 만들고 외부에 제공함으로써 협업을 원활하게 한다.
- 기본적으로 AsciiDoc을 사용하여 문서를 작성한다.
- Swagger와는 다르게 프로덕션 코드에 비침투적이다.



RestAssured

https://rest-assured.io/

- 일반적으로 인수테스트(혹은 End-To-End 테스트) 용도로 사용하는 테스트 라이브러리이다..



참고

- [인프런 : Practical Testing : 실용적인 테스트 가이드](https://www.inflearn.com/course/practical-testing-%EC%8B%A4%EC%9A%A9%EC%A0%81%EC%9D%B8-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EA%B0%80%EC%9D%B4%EB%93%9C/dashboard)
- https://velog.io/@bagt/Dynamic-Test-%EB%8F%84%EC%9E%85

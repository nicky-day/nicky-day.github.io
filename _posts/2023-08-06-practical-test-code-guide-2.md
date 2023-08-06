---
layout: single
title: "테스트 코드는 왜 필요하고 어떻게 작성해야 하는가?(2)"
categories: Test-Code
tag: [JUnit5, TDD, Mockito]
author_profile: false
sidebar:
    nav: "counts"
search: true
use_math: false
---



## 5. Spring & JPA 기반 테스트



일반적으로 사용하는 Layered Architecture는 Presentataion Layer, Business Layer, Persistence Layer로 이루어져 있다. 각각의 Layer마다 책임과 관심사가 다르기 때문에 테스트 코드 또한 Layer 별로 다른 목적을 가지고 작성해야 한다.



![Test_05-1]({{site.url}}/images/2023-08-06-practical-test-code-guide-2/Test_05-1.png)



또한 프로젝트가 커지게 되면 여러 객체나 모듈이 협력하게 되면서 단위 테스트로만으로는 커버하기가 어려운 영역이 생긴다. 이럴 때 필요한 것이 바로 통합테스트이다. 



통합 테스트

- 여러 모듈이 협력하는 기능을 통합적으로 검증하는 테스트
- 일반적으로 작은 범위의 단위 테스트만으로는 기능 전체의 신뢰성을 보장할 수 없다.
- 풍부한 단위 테스트 & 큰 단위를 검증하는 통합 테스트가 모두 필요하다.



Layered Architecture

- Presentational Layer
  - 외부 세계의 요청을 가장 먼저 받는 계층
  - 파라미터에 대한 최소한의 검증을 수행한다.
- Business Layer
  - 비즈니스 로직을 구현한 역할
  - Persistence Layer와의 상호작용(Data를 읽고 쓰는 행위)를 통해 비즈니스 로직을 전개시킨다.
  - 트랜잭션을 보장해야 한다.
- Persistence Layer
  - Data Access의 역할
  - 비즈니스 가공 로직이 포함되어서는 안된다. Data에 대한 CRUD에만 집중한 Layer이다.



즉, Persistence Layer의 테스트는 단위 테스트의 느낌에 가깝지만 Business Layer의 테스트는 Business Layer + Persistence Layer를 통합한 두 Layer를 한꺼번에 테스트하는 통합 테스트 느낌이 나도록 작성한다. 

또한 Presentational Layer의 경우 파라미터에 대한 최소한의 검증을 수행한다. 따라서 Business Layer와 Persistence Layer는 Mocking 해야 한다. 추가로 Domain Layer에서는 Entity가 가져야 할 초기 상태나 Domain 내부 함수를 검증해야 한다.



![Test_05-2]({{site.url}}/images/2023-08-06-practical-test-code-guide-2/Test_05-2.png)

![Test_05-3]({{site.url}}/images/2023-08-06-practical-test-code-guide-2/Test_05-3.png)

![Test_05-4]({{site.url}}/images/2023-08-06-practical-test-code-guide-2/Test_05-4.png)

Spring에서 각 Layer별로 일반적으로 다음 어노테이션을 클래스 상단에 붙이거나 객체를 사용한다.

- Presentational Layer
  - @WebMvcTest
  - MockMvc : Mock(가짜) 객체를 사용해 스프링 MVC 동작을 재현할 수 있는 테스트 프레임워크
  - MockBean : 컨테이너에 Mock 객체로 만든 Bean을 넣어준다.
  - objectMapper
- Business Layer
  - @SpringBootTest
  - @Transactional
- Persistence Layer
  - @DataJpaTest



## 6. Mock을 마주하는 자세



Test Double

- Dummy : 아무것도 하지 않는 깡통 객체
- Fake : 단순한 형태로 동일한 기능은 수행하나, 프로덕션에서 쓰기에는 부족한 객체 (ex : FakeRepository : HashMap으로 구현)

- Stub : 테스트에서 요청한 것에 대해 미리 준비한 결과를 제공하는 객체. 그 외에는 응답하지 않는다.
- Spy : Stub이면서 호출한 내용을 기록하여 보여줄 수 있는 객체. 일부는 실제 객체처럼 동작하고 일부만 Stubbing 할 수 있다.
- Mock : 행위에 대한 기대를 명세하고, 그에 따라 동작하도록 만들어진 객체



또한 Stub과 Mock은 가짜 객체인것은 유사하지만 Stub의 목적은 상태 검증(State Verification), 즉 객체(Stub)의 상태가 어떻게 바뀌었는지 검증하는 것이고, Mock의 목적은 행위 검증(Behavior Verification)이다.

참고 : https://martinfowler.com/articles/mocksArentStubs.html



Mockito의 어노테이션의 의미는 아래와 같다.

- @Mock : Mock 객체를 만들어 반환한다. 실제 인스턴스 없이 가상의 Mock 인스턴스를 직접 만들어 사용한다. 주로 @InjectMocks와 조합하여 사용한다.

  ```java
  @DisplayName("메일 전송 테스트")
  @Test
  void sendMail() {
      // given
      Mockito.when(mailSendClient.sendEmail(anyString(), anyString(), anyString(), anyString()))
              .thenReturn(true);
  
      // when
      boolean result = mailService.sendMail("", "", "", "");
  
      // then
      assertThat(result).isTrue();
      Mockito.verify(mailSendHistoryRepository, Mockito.times(1)).save(any(MailSendHistory.class));
  }
  ```

- @Spy : Spy 객체를 만들어 반환한다. 실제 인스턴스를 사용하여 Mocking 하고, Spy 객체는 행위를 지정하지 않으면 객체를 만들 때 사용한 실제 인스턴스의 메서드를 호출한다. 주로 @InjectMock와 조합하여 사용한다.

  ```java
  @DisplayName("메일 전송 테스트")
  @Test
  void sendMail() {
      // given
      Mockito.doReturn(true)
              .when(mailSendClient)
              .sendEmail(anyString(), anyString(), anyString(), anyString());
  
      // when
      boolean result = mailService.sendMail("", "", "", "");
  
      // then
      assertThat(result).isTrue();
      Mockito.verify(mailSendHistoryRepository, Mockito.times(1)).save(any(MailSendHistory.class));
  }
  ```


- @InjectMocks : @Mock이나 @Spy 객체를 자신의 멤버 클래스와 일치하면 주입한다.
- @MockBean : ApplicationContext에 Mock 객체를 추가한다. Mock과 달리 spring-boot-test에서 제공하는 어노테이션이다. 주로 @Autowired와 조합해서 사용한다.
- @SpyBean : ApplicationContext에 Spy 객체를 추가한다. Spy와 달리 spring-boot-test에서 제공하는 어노테이션이다. 주로 @Autowired와 조합해서 사용한다.



또한 Mock 객체의 행위를 명세하는 것은 given 단계에 해당하는데 Mockito에서는 Mockito.when().thenReturn() 과 같은 형식으로 작성하게 되어 있다.

```java
// given
Mockito.when(mailSendClient.sendEmail(anyString(), anyString(), anyString(), anyString()))
       .thenReturn(true);
```

이런 문제점을 해결하기 위해서는 더 BDD스럽게 작성할 수 있도록 도와주는 BDDMockito를 사용하면 된다.

```java
@DisplayName("메일 전송 테스트")
@Test
void sendMail() {
    // given
    BDDMockito.given(mailSendClient.sendEmail(anyString(), anyString(), anyString(), anyString()))
              .willReturn(true);

    // when
    boolean result = mailService.sendMail("", "", "", "");

    // then
    assertThat(result).isTrue();
    Mockito.verify(mailSendHistoryRepository, Mockito.times(1)).save(any(MailSendHistory.class));
}
```



마지막으로 Mock을 사용하여 검증한다고 하면 실제 프로덕션 코드에서 런타임 시점에 일어날 일을 정확하게 Stubbing 했다고 볼 수 있는가? 

Mock은 외부 세계에 영향을 주는 코드가 포함된 기능을 검증할 때 외부 시스템이 잘 동작할 것이라고 가정할 때 사용하면 좋다.

![Test_06-1]({{site.url}}/images/2023-08-06-practical-test-code-guide-2/Test_06-1.png)



참고

- [인프런 : Practical Testing : 실용적인 테스트 가이드](https://www.inflearn.com/course/practical-testing-%EC%8B%A4%EC%9A%A9%EC%A0%81%EC%9D%B8-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EA%B0%80%EC%9D%B4%EB%93%9C/dashboard)
- https://cornswrold.tistory.com/479
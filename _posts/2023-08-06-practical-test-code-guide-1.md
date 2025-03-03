---
layout: single
title: "테스트 코드는 왜 필요하고 어떻게 작성해야 하는가?(1)"
categories: Test
tag: [JUnit5, TDD, Mockito]
author_profile: false
sidebar:
    nav: "counts"
search: true
use_math: false
---

## 1. 테스트는 왜 필요할까?

테스트 코드는 소프트웨어의 품질을 보증하는 방법 중 하나이다. 




소프트웨어의 크기가 커지면 다음과 같은 문제가 발생할 수 있다.

- 커버할 수 없는 영역 발생

- 경험과 감에 의존

- 늦은 피드백

- 유지보수 어려움

- 소프트웨어 신뢰 ↓

  

그러나 테스트를 통해 우리는 다음과 같은 것을 얻을 수 있다.

- 빠른 피드백

- 자동화

- 안정감

  


자동화된 테스트 코드를 작성하지 않으면 다음과 같은 문제가 발생된다.

- 변화가 생기는 매순간마다 발생할 수 있는 모든 Case를 고려해야 한다.

- 변화가 생기는 매순간마다 모든 팀원이 동일한 고민을 해야 한다.

- 빠르게 변화하는 소프트웨어의 안정성을 보장할 수 없다.

  


그러나 무작정 테스트 코드를 작성했을 때 위와 같은 이점을 얻을 수 있는 것은 아니다.  잘못된 테스트 코드는 다음과 같은 병목을 야기한다.

- 프로덕션 코드의 안정성을 제공하기 힘들어진다.

- 테스트 코드 자체가 유지보수하기 어려운, 새로운 짐이 된다.

- 잘못된 검증이 이루어질 가능성이 생긴다.

  


따라서 개발자는 올바른 테스트 코드를 작성해야 하며, 이는 다음과 같은 장점을 가져다준다.

- 자동화 테스트로 비교적 빠른 시간 안에 버그를 발견할 수 있고, 수동 테스트에 드는 비용을 크게 절약할 수 있다.

- 소프트웨어의 빠른 변화를 지원한다.

- 팀원들의 집단 지성을 팀 차원의 이익으로 승격시킨다.

  

그렇다..! 테스트 코드는 귀찮지만 반드시 해야 하는 작업이다. 나 역시도 테스크 코드를 작성하지 않고 프로젝트를 빠르게 완성하는데 급급했던 적이 있는데 그럴 때마다 발생할 수 있는 모든 케이스에 대해 제대로 검증을 하지 않았던 부분에서 문제가 발생한 적이 있었다. 




## 2. 단위테스트

수동 테스트와 자동화된 테스트의 차이는 최종 확인 주체가 사람인지 기계인지에 따라 다르다. 수동 테스트의 경우에 상황을 직접 만들어내기 때문에 해당 상황만 검증할 수 있을 뿐 아니라 그 테스트를 작성하지 않은 사람은 어떤 것이 맞는 상황이고 틀린 상황인지 알 수 없게 된다. 그래서 자동화된 단위 테스트를 지원하기 위해 JUnit 프레임워크가 등장하게 되었다.




단위테스트란?

- 작은 코드 단위 (클래스 or 메서드)를 독립적으로 검증하는 테스트

- 외부 상황에 의존하는 테스트가 아닌 내부 상황만 테스트한다.

- 검증 속도가 빠르고, 안정적이다.

  


JUnit5

- 단위 테스트를 위한 테스트 프레임워크

- Kent Back의 XUnit 시리즈(SUnit - SmallTalk, JUnit - Java, NUnit - .Net ...)

  


AssertJ

- 테스트 코드 작성을 원활하게 돕는 테스트 라이브러리

- 풍부한 API, 메서드 체이닝 지원

  


테스트 케이스 세분화하기

- 해피 케이스

- 예외 케이스

- 경계값 테스트 : 범위(이상, 이하, 초과, 미만), 구간, 날짜 등

  

또한, 테스트 케이스를 작성하다보면 테스트 하기 어려운 영역이 발생하게 된다. 예를 들면 특정 시간에 따라 결과가 달라지는 기능이 있을 수 있다. 이런 경우에는 테스트 하기 어려운 영역을 구분하고 외부로 분리해야 한다. 외부로 분리할 수록 테스트 가능한 코드는 많아진다.




테스트하기 어려운 영역

- 관측할 때마다 다른 값에 의존하는 코드
  - 현재 날짜 / 시간, 랜덤 값, 전역 변수 / 전역 함수, 사용자 입력
- 외부 세계에 영향을 주는 코드
  - 표준 출력, 메시지 발송, 데이터베이스에 기록하기 등



테스트하기 쉬운 영역

- 순수 함수(pure functions)
  - 같은 입력에는 항상 같은 결과
  - 외부 세상과 단절된 형태
  - 테스트하기 쉬운 코드



## 3. TDD : Test Driven Development

TDD란? 프로덕션 코드보다 테스트 코드를 먼저 작성하여 테스트가 구현 과정을 주도하도록 만드는 방법론이다. TDD의 핵심가치는 바로 "피드백"이다. TDD를 통해 내가 작성하는 프로덕션 코드에 대해 자주, 빠르게 피드백을 받을 수 있다는 의미다. 그래서 TDD를 한다는 것은, Red-Green-Refactor 사이클을 반복한다는 의미이다.



![Test_01-1]({{site.url}}/images/2023-08-06-practical-test-code-guide/Test_01-1.png)



선 기능 구현 후, 테스트를 작성하게 되면

- 테스트 케이스 자체의 누락 가능성
- 특정 케이스 케이스(해피 케이스)만 검증할 가능성
- 잘못된 구현을 다소 늦게 발견할 가능성



선 테스트 작성, 후 기능을 구현하게 되면

- 복잡도가 낮은(유연하며 유지보수하기 쉬운), 테스트 가능한 코드로 구현할 수 있게 한다.
- 쉽게 발견하기 어려운 엣지(Edge) 케이스를 놓치지 않게 해준다.
- 구현에 대한 빠른 피드백을 받을 수 있다.
- 과감한 리팩토링이 가능해진다.



TDD는 테스트는 구현부 검증을 위한 보조 수단이라는 관점에서 테스트와 상호 작용하며 발전하는 구현부라는 관점으로의 변화이다. 즉, TDD는 클라이언트 관점에서 피드백을 주는 Test Driven 개발 방법론이다.



## 4. 테스트는 문서이다.

테스트 코드는 문서의 역할도 수행한다. 



테스트는 문서다.

- 프로덕션 기능을 설명하는 테스트 코드 문서
- 다양한 테스트 케이스를 통해 프로덕션 코드를 이해하는 시각과 관점 보완
- 어느 한 사람이 과거에 경험했던 고민의 결과물을 팀 차원으로 승격시켜서 모두의 자산으로 공유할 수 있다.



테스트 코드를 문서로 깔끔하게 작성하는 팁

- DisplayName을 섬세하게 작성한다.

  - 명사의 나열보다 문장으로 작성한다.
  - 테스트 행위에 대한 결과까지 기술한다.
  - 도메인 용어를 사용하여 한층 추상화된 내용을 담는다.(메서드 자체의 관점보다 도메인 정책 관점으로)
  - 테스트의 현상을 중점으로 기술하지 않고 검증하고 싶은 내용을 기술한다.

- BDD 스타일로 작성하기 : BDD(Behavior Driven Development)

  - TDD에서 파생된 개발방법
  - 함수 단위의 테스트에 집중하기보다, 시나리오에 기반한 테스트케이스(TC) 자체에 집중하여 테스트한다.
  - 개발자가 아닌 사람이 봐도 이해할 수 있을 정도의 추상화 수준(레벨)을 권장한다.

  - Given / When / Then
    - Given : 시나리오 진행에 필요한 모든 준비 과정(객체, 값, 상태 등)
    - When : 시나리오 행동 진행
    - Then : 시나리오 행동에 대한 결과 명시, 검증
    - 어떤 환경에서(Given) 어떤 행동을 진행했을 때(When) 어떤 상태 변화가 일어난다.(Then)
    - DisplayName에 명확하게 작성할 수 있다.



참고

- [인프런 : Practical Testing : 실용적인 테스트 가이드](https://www.inflearn.com/course/practical-testing-%EC%8B%A4%EC%9A%A9%EC%A0%81%EC%9D%B8-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EA%B0%80%EC%9D%B4%EB%93%9C/dashboard)

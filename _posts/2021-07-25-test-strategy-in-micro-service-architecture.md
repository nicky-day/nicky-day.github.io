---
layout: single
title: "테스트 전략 : 마이크로서비스 아키텍처"
categories: Test
tag: [MSA]
author_profile: false
sidebar:
    nav: "counts"
search: true
use_math: false
---

## 마이크로서비스 아키텍처의 테스트 복잡성을 관리하기 위한 전략

### 1. 마이크로서비스 아키텍처란?

```마이크로서비스 아키텍처```란 아키텍처 수준에서 단일 책임 원칙을 적용한 결과이다. 따라서 기존 모놀리식 아키텍처에 비해 독립적인 배포 가능성, 언어, 다양한 구성 요소에 대한 플랫폼 및 기술 독립성, 뚜렷한 확장성, 아키텍처 유연성 향상과 같은 많은 이점이 있다.

마이크로 서비스는 ```HTTP를 통한 REST를 사용하여 통합되는 경우가 많다.``` 따라서 비즈니스 도메인 개념은 각 서비스에서 관리하는 하나 이상의 자원으로 모델링된다. 
통합 메커니즘의 대안으로는 경량 메시지 프로토콜, 발행-구독 모델 또는 Protobuf나 Thrift와 같은 전송 기술을 사용한다. 

마이크로서비스는 주로 다음과 같은 구조로 분할된다.

![MSA_Test-1]({{site.url}}/images/2021-07-25-test-strategy-in-micro-service-architecture/images_jsj3282_post_36a1e848-3401-4bbf-937d-c03e6c249cfa_image.png)

**Resources**

리소스는 서비스에 의해 노출되는 애플리케이션 프로토콜과 도메인을 나타내는 개체에 대한 메시지 사이에서 매퍼 억할을 한다. 일반적으로 요청 상태를 확인하고 비즈니스 트랜잭션의 결과에 따라 프로토콜별 응답을 제공하는 역할을 해야하는 책임이 있다.

**Service Layer, Domain, Repositories**

거의 모든 서비스 로직은 도메인에 있으며, 도메인은 비즈니스 도메인을 의미한다. 리소스는 서비스에 비즈니스 트랜잭션을 완료하기 위해 책임을 위임한다. 서비스 레이어는 여러개의 도메인에 걸쳐서 행위를 조정하며, 리포지토리는 도메인 엔터티 집합과 같은 개념으로 거의 영구적이다.

**Gateways, HTTP Client**

하나의 서비스가 외부의 서비스와 통신해야 할때, 게이트웨이는 도메인 개체에 대한 요청과 응답을 캡슐화하고 이런 요청-응답 주기를 처리하기 위해 HTTP Client를 사용한다. 

**Data Mappers / ORM**

도메인과 리포지토리 간 개체 관계 매핑 시 사용한다. (JSON, XML 등)

![MSA_Test-2]({{site.url}}/images/2021-07-25-test-strategy-in-micro-service-architecture/images_jsj3282_post_8be26ceb-2e2b-4119-a353-46c74972c377_image.png)

마이크로서비스는 네트워크를 통해 서로 연결되며 "외부" 데이터 저장소를 활용한다.

### 2. 단위 테스트

```단위테스트```는 일반적으로 클래스 수준이나 소규모 클래스 그룹에서 작성된다. 단위 테스트는 애플리케이션에서 테스트 가능한 가장 작은 부분을 실행하여 예상대로 작동하는지 여부를 확인한다.

![MSA_Test-3]({{site.url}}/images/2021-07-25-test-strategy-in-micro-service-architecture/images_jsj3282_post_47d6a046-49a2-474a-afd0-ade2ef42873c_image.png)

**Sociable unit testing**

기능과 그에 따른 종속성에 대한 도메인 객체 상태 변화를 관찰할 수 있는 테스트

**Solitary unit testing**

도메인 객체를 테스트 스텁으로 대체해 기능만 테스트

### 3. 통합 테스트

```통합테스트```는 인터페이스의 결함을 발견하기 위해 구성 요소 간의 통신 경로 및 상호작용을 확인한다. 일반적으로 통합 테스트에서는 모든 단위로 작성하기 보다는, 외부 구성 요소 간의 상호작용을 확인하는 데 사용된다. 통합 테스트가 유용할 수 있는 외부 구성 요소로는 다른 마이크로서비스, 데이터 저장소 및 캐시가 포함된다.

![MSA_Test-4]({{site.url}}/images/2021-07-25-test-strategy-in-micro-service-architecture/images_jsj3282_post_13cb07d9-3a75-4261-b829-450f0e6b19e9_제목 없음.png)

### 4. 구성 요소 테스트

```구성 요소 테스트```는 실행된 소프트웨어의 범위를 테스트 중인 시스템의 일부로 제한하고 내부 코드 인터페이스를 통해 시스템을 조작하고 테스트 스텁을 사용하여 테스트 중인 코드를 다른 구성 요소로부터 분리하는 것을 말한다. 마이크로 서비스 아키텍처에서 구성요소란 서비스 자체이다. 외부 서비스를 내부 API 엔드포인트로 교체하고, 데이터 저장소를 인메모리 등으로 교체하는 등 테스트 스텁을 사용한다.

![MSA_Test-5]({{site.url}}/images/2021-07-25-test-strategy-in-micro-service-architecture/images_jsj3282_post_4e315a8d-949b-4227-988a-cd33fa218351_제목 없음2.png)

내부 리소스 : 로그, 기능 플래그, 데이터베이스 명령, 시스템 메트릭 등, 상태 확인 리소스
이미 이를 다루는 많은 도구가 있기 때문에 내부 제어를 리소스로 노출하면 모니터링, 유지 관리 및 디버깅과 같은 테스트 외에도 여러 경우에 유용할 수 있다. 

### 5. 계약 테스트

```통합 계약 테스트```는 소비 서비스에서 기대하는 계약을 충족하는지 확인하는 외부 서비스 경계에서의 테스트이다. 계약은 일반적으로 입력 및 출력 데이터 구조, 부작용 및 성능 및 동시성 특성에 대한 것이다. 

![MSA_Test-6]({{site.url}}/images/2021-07-25-test-strategy-in-micro-service-architecture/images_jsj3282_post_15173296-5e39-4c86-97f4-b040cd5598ec_image.png)

### 6. 엔드 투 엔드 테스트

```엔드 투 엔드 테스트```는 시스템이 외부 요구 사항을 충족하고 목표를 달성하는지 확인하여 전체 시스템을 종단에서 끝까지 테스트한다. 엔드 투 엔드 테스트의 목적은 완전히 통합된 시스템의 동작을 테스트하는 것이다. 

![MSA_Test-7]({{site.url}}/images/2021-07-25-test-strategy-in-micro-service-architecture/images_jsj3282_post_60d500d2-9a82-4a06-9eb2-f6337e2b939d_image.png)

### 7. 테스트 피라미드

![MSA_Test-8]({{site.url}}/images/2021-07-25-test-strategy-in-micro-service-architecture/images_jsj3282_post_9f4bfc8e-25a9-41ea-bce2-43e26a81d149_image.png)

```탐색 테스트```는 시스템을 수동으로 탐색하여 시스템에 대해 학습하고 자동화된 테스트를 개선할 수 있다. 

### 8. 요약

![MSA_Test-9]({{site.url}}/images/2021-07-25-test-strategy-in-micro-service-architecture/images_jsj3282_post_b86f0439-b19c-443b-9f1d-cf00685c1ecb_제목 없음3.png)

참고
- https://martinfowler.com/articles/microservice-testing/
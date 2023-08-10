---
layout: single
title: "선착순 이벤트 시스템에서 발생할 수 있는 동시성 이슈와 해결 방법"
categories: Concurrency-Issue
tag: [Java, MySQL, Redis]
author_profile: false
sidebar:
    nav: "counts"
search: true
use_math: false
---

## 0. 선착순 이벤트 시스템에서 발생할 수 있는 동시성 이슈



정해둔 수량보다 많은 쿠폰이 발급되고, 트래픽이 몰려 에러까지 발생한다면 어떻게 해결해야 할까? 

Redis를 활용하여 쿠폰 발급 개수를 보장하고, Kafka를 사용하여 다른 페이지들에 대한 영향도를 줄일 수 있다.



## 1. Redis를 활용하여 동시성 이슈 해결하기



정해둔 수량보다 많은 쿠폰이 발급된 이유는 동시성 이슈 때문이다.

동시성 이슈란 여러 스레드가 동일한 자원을 공유할 때 발생할 수 있는 이슈를 말한다. 다른 말로 이를 Race Condition이 발생했다고 말한다.



Java의 synchronized를 사용한다면 서버가 여러대일 경우 다시 Race Condition이 발생한다. MySQL Lock을 사용하게 되면 쿠폰 개수를 가져오는 것 부터 쿠폰 생성까지 Lock을 걸어야 한다. 그렇다면 Lock을 거는 구간이 길어져 성능에 불이익이 발생하게 된다. 쿠폰 생성이 아닌 쿠폰 개수에 대한 정합성만 관리하려면 어떻게 해야 할까?



이 문제를 해결하기 위해서는 Redis Incr 기능을 활용하면 된다. Incr은 Key에 대한 Value를 1씩 증가시키는 명령어이다. Redis는 싱글스레드로 동작하기 때문에 Race Condition을 해결할 수 있을 뿐만 아니라 Incr 명령어는 성능도 빠르다. Incr을 활용하여 쿠폰 개수를 제어한다면 성능도 빠르며 데이터 정합성도 지킬 수 있다.



```java
public Long increment() {
	return redisTemplate
         .opsForValue()
         .increment("coupon_count");
}
```



그러나 Redis Incr 만으로는 쿠폰이 많이 발급될수록 RDB에 부하가 많이 가는 문제를 막을 수는 없다. 그래서 다른 서비스까지 장애가 발생될 수도 있다.

![Concurrency-Issue-In-Coupon-Event-1]({{site.url}}/images/2023-08-06-possible-concurrency-issue-in-firtst-come-and-first-served-event/Concurrency-Issue-In-Coupon-Event-1.png)

## 2. Kafka를 사용하여 RDB 부하 문제 해결하기



> Kafka란?
>
> 카프카는 오픈소스 분산 이벤트 스트리밍 플랫폼이다.
>
> [https://kafka.apache.org/](https://kafka.apache.org/)



![Concurrency-Issue-In-Coupon-Event-2]({{site.url}}/images/2023-08-06-possible-concurrency-issue-in-firtst-come-and-first-served-event/Concurrency-Issue-In-Coupon-Event-2.png)


토픽은 간단하게 말하면 큐이다. 토픽에 데이터를 삽입할 수 있는 것이 Producer이고, 토픽에 삽입된 데이터를 가져갈 수 있는 것이 Consumer이다.



카프카를 사용하면 API에서 직접 쿠폰을 생성할 때에 비해 처리량을 조절할 수 있게 된다. 처리량을 조절함에 따라 데이터베이스에 부하를 줄일 수 있다는 장점이 있다. 다만 쿠폰 생성까지는 약간의 텀이 생긴다는 단점이 있다.



## 3. 요구사항 변경 : 발급가능한 쿠폰개수를 1인당 1개로 제한하기



제일 간단하게 해결할 수 있는 방법은 데이터베이스에 유니크키를 거는 방법이다. 그러나 한명의 사용자가 같은 타입의 쿠폰을 중복해서 가지는 것이 가능한 케이스도 있기 때문에 이 방법은 실용적이지 않다. 



두번째 방법은 쿠폰 발급 여부를 먼저 조회해오고 쿠폰 발급 여부 확인부터 쿠폰 생성까지 Lock을 거는 방법이다. 그러나 실제 쿠폰 생성은 Kafka Consumer에서 이루어지기 때문에 쿠폰 생성까지 약간의 텀이 발생해 쿠폰 발급 여부 확인이 정확하지 않을 수 있다. API에서 직접 쿠폰을 발급한다고 해도 Lock 범위가 너무 넓어져 성능이 나빠진다.



이럴 때 사용 가능한 Set이라는 자료구조가 있다. Redis에서도 Set이라는 자료구조가 있기 때문에 Redis를 사용해서 문제를 해결할 수 있다.

Redis에 sadd라는 명령어를 사용하면 Set에 Value를 추가한다.



```java
public Long add(Long userId) {
    return redisTemplate
            .opsForSet()
            .add("applied_user", userId.toString());
}
```



### 4. 시스템 개선하기



쿠폰을 발급하다가 Consumer에서 에러가 발생해 쿠폰 발급 개수는 발급해야 하는 쿠폰 개수만큼 올라가 있는데, 실제 DB에서 쿠폰 발급 개수를 조회해보면 발급해야 하는 쿠폰 개수보다 적을 수 있다. 

이 문제를 해결하기 위해 Consumer에서 에러가 발생했을 때 FailedEvent 즉, 백업데이터와 로그를 남긴 후 배치 프로그램으로 쌓인 FailedEvent를 주기적으로 읽어서 쿠폰을 발급해준다면 결과적으로 발급해야 하는 쿠폰 개수만큼의 쿠폰이 모두 발급될 것이다.



![Concurrency-Issue-In-Coupon-Event-3]({{site.url}}/images/2023-08-06-possible-concurrency-issue-in-firtst-come-and-first-served-event/Concurrency-Issue-In-Coupon-Event-3.png)



참고

- [인프런 : 실습으로 배우는 선착순 이벤트 시스템](https://www.inflearn.com/course/lecture?courseSlug=%EC%84%A0%EC%B0%A9%EC%88%9C-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EC%8B%A4%EC%8A%B5&unitId=159888&tab=curriculum&category=questionDetail&q=952111)

---
layout: single
title: "재고 시스템에서 발생할 수 있는 동시성 이슈와 해결 방법"
categories: Concurrency-Issue
tag: [Java, MySQL, Redis]
author_profile: false
sidebar:
    nav: "counts"
search: true
use_math: false
---



## 0. 재고 시스템에서 발생할 수 있는 동시성 이슈



쇼핑몰 재고 수량을 수정했는데, 다시 조회했더니 저장한 것과 다른 수량이 반환된다면?

동시성 이슈란 여러 스레드가 동일한 자원을 공유할 때 발생할 수 있는 이슈를 말한다. 다른 말로 이를 Race Condition이 발생했다고 말한다.

백엔드 개발자라면 동시성을 고려한 프로그래밍을 할 줄 알아야 하며, 프로젝트를 시작할 때 동시성을 고려하지 않고 개발을 시작하게 되면 데이터 정합성이 중요한 상황에서 여러가지 문제가 발생할 수 있다.



그렇다면 동시성 이슈를 해결할 수 있는 방법은 하나의 스레드가 작업이 완료된 후에 다른 스레드가 데이터에 접근할 수 있도록 하면 된다.



## 1. Application Level : Java의 synchronized 사용하기

문제점 1 : synchronized와 @Transactional을 같이 사용할 경우

- 트랜잭션은 스프링에서 AOP로 동작하며, 트랜잭션 종료 시점에 데이터를 DB에 업데이트하는데 이때 다른 스레드가 트랜잭션 종료 전에 synchronized가 붙어 있는 메서드를 호출할 수 있다. 그렇게 하면 다른 스레드는 갱신 전의 데이터를 가져와서 이전과 동일한 문제가 발생한다.

문제점 2 :  자바의 synchronized는 하나의 프로세스 안에서만 보장이 된다. 

- 따라서 서버가 한 대일때는 괜찮지만 서버가 여러 대일 경우에는 데이터 접근을 여러 군데에서 할 수 있다. 실제 운영 중인 서버는 대부분 2대 이상을 사용하기 때문에 synchronized는 거의 사용되지 않는다.

![Concurrency-Issue-In-Stock-1]({{site.url}}/images/2023-08-06-possible-concurrency-issue-in-inventory-system/Concurrency-Issue-In-Stock-1.png)

## 2. Database Lock : MySQL

[https://dev.mysql.com/doc/refman/8.0/en/locking-functions.html](https://dev.mysql.com/doc/refman/8.0/en/locking-functions.html)

[https://dev.mysql.com/doc/refman/8.0/en/metadata-locking.html](https://dev.mysql.com/doc/refman/8.0/en/metadata-locking.html)



### 1) Pessimistic Lock 활용해보기

- 실제로 데이터에 Lock을 걸어서 정합성을 맞추는 방법이다. Exclusive Lock을 걸게되면 다른 트랜잭션에서는 Lock이 해제되기 전에 데이터를 가져갈 수 없게 된다. 데드락이 걸릴 수 있기 때문에 주의하여 사용해야 한다.

  > 데드락(Dead Lock, 교착상태)
  >
  > 두 개 이상의 프로세스나 스레드가 서로 자원을 얻지 못해서 다음 처리를 하지 못하는 상태로
  >
  > 무한히 다음 자원을 기다리게 되는 상태를 말한다.
  >
  > 시스템으로 한정된 자원을 여러 곳에서 사용하려고 할 때 발생한다.
  >
  > DeadLock이란 : https://hckcksrl.medium.com/deadlock-%EC%9D%B4%EB%9E%80-8100261a66c3
  
  ```java
  @Lock(value = LockModeType.PESSIMISTIC_WRITE)
  @Query("select s from Stock s where s.id = :id")
  Stock findByIdWithPessimisticLock(Long id);
  ```

- JPA 쿼리를 보면 select ... for update 형태이다.
- 첫번째 트랜잭션이 종료된 이후에야 두번째 트랜잭션이 Lock을 획득하고 update를 처리한다.
- 장점으로는 충돌이 빈번하게 일어난다면 Optimistic Lock보다 성능이 좋다. 또한 Lock을 통해 update를 제어하기 때문에 데이터 정합성이 어느 정도 보장된다.
- 단점으로는 별도의 Lock을 사용하기 때문에 성능 감소가 있을 수 있다.



### 2) Optimistic Lock 활용해보기

- 실제로 Lock을 이용하지 않고 버전을 이용함으로써 정합성을 맞추는 방법이다. 먼저 데이터를 읽은 후 update를 수행할 때 현재 내가 읽은 버전이 맞는지 확인하며 업데이트한다. 내가 읽은 버전에서 수정사항이 생겼을 경우 application에서 다시 읽은 후에 작업을 수행해야 한다.

- 엔티티가 변경되면 업데이트 시점에 버전 값이 증가한다.

  ```java
  @Entity
  public class Stock {
  
      @Id
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      private Long id;
  
      private Long productId;
  
      private Long quantity;
  
      @Version
      private Long version;
  }
  ```

  ```java
  @Lock(value = LockModeType.OPTIMISTIC)
      @Query("select s from Stock s where s.id = :id")
      Stock findByIdWithOptimisticLock(Long id);
  ```

- 장점으로는 별도의 Lock을 잡지 않으므로 Pessimisitc Lock보다 성능상 이점이 있다. update가 실패하면 재시도하는 로직을 개발자가 작성해야 한다. 

- 단점으로는 충돌이 빈번하게 일어난다면 성능상 Pessimistic Lock을 사용하는 게 더 유리하다.

  

### 3) Named Lock 활용해보기

- 이름을 가진 metadata Locking 이다. 이름을 가진 Lock을 획득한 후 해제할 때까지 다른 세션은 이 Lock을 획득할 수 없도록 한다. 주의할 점으로는 Transaction이 해제될 때 이 Lock이 자동으로 해제되지 않는다. 별도의 명령어로 해제를 수행해주거나 선점시간이 끝나야 해제된다.

  ![Concurrency-Issue-In-Stock-2]({{site.url}}/images/2023-08-06-possible-concurrency-issue-in-inventory-system/Concurrency-Issue-In-Stock-2.png)

- Pessimistic Lock과 동작 방식이 유사한데, 차이점은 Pessimistic Lock은 Row나 Table에 Lock을 걸어준다면 Named Lock은 Row나 Table이 아니라 metadata에 Locking을 하는 방법이다.

  ```java
  @Query(value = "select get_lock(:key, 3000)", nativeQuery = true)
  void getLock(String key);
  
  @Query(value = "select release_lock(:key)", nativeQuery = true)
  void releaseLock(String key);
  ```

- 주의할 점으로는 Named Lock은 커넥션을 2개 사용한다. Lock 획득에 필요한 커넥션 1개, 트랜잭션 로직에 사용하는 커넥션 1개이다. 따라서 커넥션 풀이 부족해지는 것을 방지하기 위해 각각의 커넥션에 필요한 커넥션 풀을 분리하는 것이 좋다. 

- Named Lock은 주로 분산락을 구현할 때 사용한다. Pessimistic Lock은 Timeout을 구현하기 힘들지만 Named Lock은 손쉽게 구현할 수 있다. 그 이외에 삽입시 데이터 정합성을 맞춰야 하는 경우에도 사용할 수 있다. 



## 3. Redis Distributed Lock

> **분산락이란?**
>
> 분산락이란 경쟁 상황(Race Condition) 이 발생할때, 하나의 공유자원에 접근할때 데이터에 결함이 발생하지 않도록 원자성(atomic) 을 보장하는 기법이다. (데이터베이스에서 제공하는 Lock과는 별개이다.)
>
> 참고 : https://hudi.blog/distributed-lock-with-redis/



분산락 기능을 제공하는 Redis 라이브러리는 아래와 같다.



### 1) Lettuce

- setnx(set if not exist) 명령어를 사용하여 분산락 구현

  ![Concurrency-Issue-In-Stock-3]({{site.url}}/images/2023-08-06-possible-concurrency-issue-in-inventory-system/Concurrency-Issue-In-Stock-3.png)

  ```java
  public Boolean lock(Long key) {
  		return redisTemplate
             .opsForValue()
             .setIfAbsent(generateKey(key), "lock", Duration.ofMillis(1_1000));
  }
  ```

- spin lock 방식 : Lock을 획득하려는 쓰레드가 Lock을 사용할 수 있는지 반복적으로 확인하면서 Lock 획득을 시도하는 방식

  ```java
  public void decrease(Long key, Long quantity) throws InterruptedException {
  		while (!redisLockRepository.lock(key)) {
      		Thread.sleep(100);  // lock 실패시에는 Thread sleep을 이용함으로써 Redis에 갈수 있는 부하를 줄여줌
      }
      try {
          stockService.decrease(key, quantity);
      } finally {
          redisLockRepository.unlock(key);
      }
  }
  ```

- Retry 로직을 개발자가 작성해야 함

- MySQL의 Named Lock과 유사하나 Redis를 사용한다는 점과 세션에 신경쓰지 않아도 된다는 점이 다르다.

- spin lock 방식이기 때문에 Redis에 부하를 줄 수 있다. 그래서 Thread.sleep 등을 활용해 락 획득 재시도 간에 텀을 둠으로써 부하를 줄여야 한다.

- 구현이 간단하다. 또한 Spring Data Redis를 이용하면 lettuce가 기본이기 때문에 별도의 라이브러리를 사용하지 않아도 된다.





### 2) Redisson

- pub-sub 방식 : 채널을 하나 만들고 Lock을 점유하고 있는 쓰레드가 Lock을 사용하려고 대기 중인 쓰레드에게 채널로 Lock 해제를 알려주면 안내를 받은 쓰레드가 Lock 획득을 시도하는 방식

  ![Concurrency-Issue-In-Stock-4]({{site.url}}/images/2023-08-06-possible-concurrency-issue-in-inventory-system/Concurrency-Issue-In-Stock-4.png)

- Lettuce와 다르게 대부분의 경우 별도의 retry 로직을 작성하지 않아도 된다. Lock 획득 재시도를 기본으로 제공한다.

  ```java
  public void decrease(Long key, Long quantity) throws InterruptedException {
      RLock lock = redissonClient.getLock(key.toString()); 						// Lock 객체를 가져온다.
  
      try {
          boolean available = lock.tryLock(5, 1, TimeUnit.SECONDS);   // 획득, 점유 시간 설정
          if (!available) {
              System.out.println("lock 획득 실패");
              return;
          }
  
          stockService.decrease(key, quantity);
  
      } catch (InterruptedException e) {
          throw new RuntimeException();
      } finally {
          lock.unlock();
      }
  }
  ```

- Lettuce에 비해 재시도를 한 번 혹은 몇 번만 하기 때문에 Redis에 가는 부하가 적음.

- 구현이 복잡하고 별도의 라이브러리를 사용해야 한다.



따라서 재시도가 필요하지 않은 Lock(ex : 선착순 1명만 구매 가능)은 Lettuce를 활용하고, 재시도가 필요한 Lock(ex : 주문할 때)은 Redisson을 활용한다.



## 4. MySQL과 Redis 비교

MySQL

- 이미 MySQL을 사용하고 있다면 별도의 비용없이 사용 가능하다.
- 어느정도의 트래픽까지는 문제 없이 사용 가능하다.
- Redis 보다는 성능이 좋지 않다.

Redis

- 활용중인 Redis가 없다면 별도의 구축비용과 인프라 관리비용이 발생한다.
- MySQL 보다 성능이 좋다.



## 5. 추가로 공부해볼 만한 것

- 동시성 이슈를 테스트할 때 ExecutorService와 CountDownLatch를 사용하면 편리하다.
- Exclusive Lock(배타적 잠금), Shard Lock(공유 잠금)의 차이
- JPA의 LockModeType
- [https://techblog.woowahan.com/2631/](https://techblog.woowahan.com/2631/)



참고

- [인프런 : 재고시스템으로 알아보는 동시성이슈 해결방법](https://www.inflearn.com/course/%EB%8F%99%EC%8B%9C%EC%84%B1%EC%9D%B4%EC%8A%88-%EC%9E%AC%EA%B3%A0%EC%8B%9C%EC%8A%A4%ED%85%9C#)
- https://kdhyo98.tistory.com/59#%F0%9F%98%85%EC%98%88%EC%A0%9C-1
- https://hckcksrl.medium.com/deadlock-%EC%9D%B4%EB%9E%80-8100261a66c3
- https://willbfine.tistory.com/576

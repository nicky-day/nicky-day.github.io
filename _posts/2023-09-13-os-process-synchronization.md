---
layout: single
title: "프로세스 동기화"
categories: OS
author_profile: false
sidebar:
    nav: "counts"
search: true
use_math: false
---

## 1. 프로세스 간 통신

프로세스는 독립적으로 실행되기도 하지만 다른 프로세스와 데이터를 주고받으며 통신을 하는 경우도 있다. 통신은 한 컴퓨터 내에서 실행되고 있는 다른 프로세스와 할 수도 있고, 네트워크로 연결된 다른 프로세스와 할 수도 있다.



프로세스 간 통신의 종류는 다음과 같다.

- 첫번째는 한 컴퓨터 내에서 프로세스 간 통신을 하는 방법으로 파일과 파이프를 이용하는 방법이다.
  - 파일을 이용하는 방법은 통신을 하려는 프로세스들이 하나의 파일을 이용해 읽고 쓰는 방법이다.
  - 파이프를 이용하는 방법은 운영체제가 생성한 파이프를 이용해 데이터를 읽고 쓰는 방법이다.
- 두번째는 쓰레드를 이용한 방법이 있다. 이 방법은 한 프로세스 내에서 쓰레드 간 통신을 하는 방법이며 쓰레드는 코드, 데이터, 힙 영역을 공유하고 스택만 자신의 영역을 가지고 있다. 여기서 데이터 영역에 있는 전역 변수나 힙을 이용하면 통신이 가능하다.
- 세번째는 네트워크를 이용한 방법이다.
  - 운영체제가 제공하는 소켓통신을 이용한다.
  - 다른 컴퓨터에 있는 함수를 호출하는 RPC(원격 프로시저 호출)를 이용해 통신을 하는 방법이 있다.



## 2. 공유자원과 임계구역

프로세스 간 통신을 할 때 공동으로 이용하는 변수나 파일들이 있는데 이런 것들을 공유자원이라고 한다. 그러나 공유자원은 여러 프로세스가 공유하고 있기 때문에 각 프로세스의 접근 순서에 따라 결과가 달라질 수 있는 문제점이 있다. 또한 컨텍스트 스위칭으로 시분할 처리를 하기 때문에 어떤 프로세스가 먼저 실행되고 어떤 프로세스가 나중에 실행되는지 예측하기가 힘들다. 따라서 연산 결과를 예측하기가 힘들고 여기서 발생한 문제를 동기화 문제라고 부른다.



따라서 여러 프로세스가 동시에 사용하면 안되는 영역을 정의했는데 이를 임계구역(Critical Section)이라고 한다. 공유자원을 서로 사용하기 위해 경쟁하는 것을 경쟁조건(Race Condition)이라고 한다. 임계구역 문제를 해결하기 위해서는 상호 배제(Mutual Exclusion)의 메커니즘이 필요하다.



상호 배제 메커니즘의 요구 사항은 세 가지가 있다.

- 첫번째는 주어진 시간에 항상 하나의 프로세스만 임계 구역에 접근할 수 있다.
- 두번째는 동시에 여러 개의 요청이 있더라도 하나의 프로세스만 진입하도록 허용해야 한다.
- 세번째는 임계 구역에 들어간 프로세스는 최대한 빠르게 나와야 한다. 그렇지 않으면 다른 프로세스들이 오래 기다려야 한다.



## 3. 세마포어

회사에서 직원들이 문서 작업을 한다고 가정하자. 직원들은 자기 문서의 작업을 마치면 프린터로 작업 결과물을 출력한다. 프린터는 네트워크를 통해 여러 컴퓨터에 연결되어 있다. 여기서 공유자원은 프린터가 된다. 직원 A와 직원 B가 우연히 동시에 프린터 출력을 하게 되면 직원 A의 컴퓨터와 직원 B의 컴퓨터가 프린터를 차지하기 위해 경쟁 조건이 되어 버린다. 이렇게 되면 프린터의 결과물이 직원 A의 결과물과 직원 B의 결과물이 섞여 나오기 때문에 쓸모가 없어진다.



이런 상황을 막기 위해 프린터실을 따로 만들고 거기에 컴퓨터와 프린터를 설치한다고 가정하자. 그리고 밖에는 이 방의 열쇠를 관리하는 직원을 따로 둔다. 이제 프린터를 사용하고 싶은 직원 A는 관리자에게 열쇠를 받아서 프린터실로 입장해 프린터를 혼자 사용할 수 있다. 조금 늦게 온 직원 B도 프린터를 사용하고 싶어서 프린터실로 들어가려고 했지만 열쇠 관리자는 이미 열쇠를 직원 A에게 준 상태라 직원 B에게 잠시 대기하고 있으라고 말한다. 직원 A가 프린터를 다 사용하면 방에서 나오고 열쇠 관리자에게 열쇠를 반납한다. 그러면 이제 직원 B가 열쇠를 받고 프린터실로 들어가 프린터를 사용할 수 있다.



위의 예시가 세마포어 메커니즘이다. 세마포어는 상호 배제 메커니즘의 한 가지 종류이며 단순해보이지만 동기화에서 가장 중요한 개념이다. 위의 상황을 운영체제에서 쓰는 용어로 매칭시켜보자. 프린터를 사용하려는 직원들은 프로세스이다. 프린터는 여러 프로세스들이 같이 쓰고 있는 공유자원이다. 프린터를 쓰기 위해 프로세스가 기다리는 공간은 대기큐이다. 열쇠 관리자는 운영체제이다. 그리고 열쇠 관리자가 가지고 있는 열쇠가 바로 세마포어이다. 실제로 세마포어는 정수형 변수이다.



세마포어 메커니즘을 사용했을때 공유자원을 사용하고 반납하기 위해 제공되는 함수가 wait()과 signal()이다. wait() 함수는 열쇠를 받을 때까지 기다렸다가 열쇠를 획득하면 방에 들어가 문을 잠그는 함수이다. signal() 함수는 공유자원을 사용하고 나서 열쇠를 반납하는 함수이다.



<img src="{{site.url}}//images/2023-09-13-os-process-synchronization/스크린샷 2023-09-13 오후 11.05.26.png" alt="os-process-synchronization-1" width="400px">




위의 그림을 보면 제일 처음에는 공유변수가 하나이기 때문에 세마포어의 초기값을 1로 설정해준다. 임계구역에 들어가기 전에 wait() 함수로 세마포어 값을 1 감소, 임계구역에서 나오면 signal() 함수로 세마포어 값을 1 증가시켜 준다. 세마포어는 정수형 변수인데 공유자원이 2개라면 세마포어의 값은 2가 된다. 



이렇게 세마포어를 이용하면 공유 자원의 여러 프로세스가 동시에 접근하지 못하기 때문에 동기화 문제가 발생하지 않는다. 그러나 wait() 함수와 signal() 함수의 순서를 이상하게 호출하면 세마포어를 잘못 사용할 가능성이 생긴다. 



## 4. 모니터

모니터는 위에 나와있는 세마포어의 단점을 해결한 상호 배제 메커니즘이다. 모니터는 운영체제가 처리하는 것이 아니라 프로그래밍 언어 자체에서 지원하는 방법이다. 대표적으로 자바에서 모니터를 지원하고 있다. 자바에서 synchronized라는 키워드가 붙으면 이 키워드가 붙은 함수들은 동시에 여러 프로세스에서 실행시킬 수 없다. 즉 상호 배제가 완벽하게 이루어진다. 



자바에서 synchronized 메소드는 인스턴스 단위로 Lock을 걸지만 synchronized 키워드가 붙은 메소드들에 대해서만 Lock을 공유한다. 쉽게 말해서 한 스레드가 synchronized 메소드를 호출하는 순간, 모든 synchronized 메소드에 Lock이 걸리므로 다른 쓰레드가 어떠한 synchronized 메소드도 호출할 수 없다. 단, 일반 메소드는 호출 가능하다.



```java
public class Method {

    public static void main(String[] args) {

        Method method = new Method();
        Thread thread1 = new Thread(() -> {
            System.out.println("스레드1 시작 " + LocalDateTime.now());
            method.syncMethod1("스레드1");
            System.out.println("스레드1 종료 " + LocalDateTime.now());
        });

        Thread thread2 = new Thread(() -> {
            System.out.println("스레드2 시작 " + LocalDateTime.now());
            method.syncMethod2("스레드2");
            System.out.println("스레드2 종료 " + LocalDateTime.now());
        });

        Thread thread3 = new Thread(() -> {
            System.out.println("스레드3 시작 " + LocalDateTime.now());
            method.method3("스레드3");
            System.out.println("스레드3 종료 " + LocalDateTime.now());
        });

        thread1.start();
        thread2.start();
        thread3.start();
    }

    private synchronized void syncMethod1(String msg) {
        System.out.println(msg + "의 syncMethod1 실행중" + LocalDateTime.now());
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private synchronized void syncMethod2(String msg) {
        System.out.println(msg + "의 syncMethod2 실행중" + LocalDateTime.now());
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private void method3(String msg) {
        System.out.println(msg + "의 method3 실행중" + LocalDateTime.now());
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```



모니터의 구현만 완벽하다면 프로그래머는 세마포어처럼 wait() 함수나 signal() 함수를 임계영역에 감싸지 않아도 되서 편리하고 안전하게 코드를 작성할 수 있다.



참고

- [그림으로 쉽게 배우는 운영체제](https://www.inflearn.com/course/%EB%B9%84%EC%A0%84%EA%B3%B5%EC%9E%90-%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9C/dashboard)
- [https://steady-coding.tistory.com/517](https://steady-coding.tistory.com/517)
- [https://velog.io/@klloo/semaphore-mutex#monitor-%EB%AA%A8%EB%8B%88%ED%84%B0](https://velog.io/@klloo/semaphore-mutex#monitor-%EB%AA%A8%EB%8B%88%ED%84%B0)

- [https://steady-coding.tistory.com/556](https://steady-coding.tistory.com/556)

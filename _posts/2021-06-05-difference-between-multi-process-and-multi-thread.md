---
layout: single
title: "멀티 스레드 / 멀티 프로세스의 이해"
categories: Computer-Science
tag: [Java]
author_profile: false
sidebar:
    nav: "counts"
search: true
use_math: false
---

### 1. T 메모리

![multi-process-and-multi-thread-1]({{site.url}}/images/2021-06-05-difference-between-multi-process-and-multi-thread/images_jsj3282_post_331c2501-a1b8-4acb-b6bd-d95150861dcf_9951443D5C6E85BE02.png)

- 스태틱 영역 : 클래스들의 놀이터
- 스택 : 메서드들의 놀이터
- 힙 : 객체의 놀이터

### 2. 멀티 스레드 / 멀티 프로세스

```멀티 스레드(Multi Thread)```의 T 메모리 모델은 스택 영역을 쓰레드 개수만큼 분할해서 쓰는 것이다. 

![multi-process-and-multi-thread-2]({{site.url}}/images/2021-06-05-difference-between-multi-process-and-multi-thread/images_jsj3282_post_8dce08f3-d69c-4da8-a3f9-aba993710a77_image.png)

- 멀티 스레드는 하나의 T 메모리만 사용하는데 스택 영역만 분할해서 사용하는 구조다.
- 멀티 스레드는 하나의 T 메모리 안에서 스택 영역만 분할한 것이기 때문에 
하나의 스레드에서 다른 스레드의 스택 영역에는 접근할 수 없지만 스태틱 영역과 힙 영역은 공유해서 사용하는 구조다.
- 멀티 프로레스 대비 메모리를 적게 사용할 수 있는 구조다. 

```멀티 프로세스(Multi Process)```는 다수의 데이터 저장 영역, 즉 다수의 T 메모리를 갖는 것이다.

![multi-process-and-multi-thread-3]({{site.url}}/images/2021-06-05-difference-between-multi-process-and-multi-thread/images_jsj3282_post_fdcc85c5-05a5-4e12-8b9a-0cbf816d72f7_image (1).png)

- 멀티 프로세스는 각 프로세스마다 각자의 T 메모리가 있고 각자 고유한 공간이므로 서로 참조할 수 없다. 
- 멀티 프로레스는 하나의 프로세스가 다른 프로세스의 T 메모리 영역을 절대 침범할 수 없는 안전한 구조이다.
- 멀티 프로세스는 각 프로세스마다 각 T 메모리가 있기 때문에 메모리 사용량이 크다.

### 3. 멀티 스레드에서 전역 변수 사용의 문제점

두 개의 쓰레드로 구성된 프로그램이 있다고 해보자. 스레드1이 공유 영역(스태틱과 힙)에 있는 전역 변수 A에 10을 할당했다고 해보자. 그런데 CPU 사용권이 스레드2로 넘어가고 스레드2가 전역변수 A에 20을 할당하고 다시 CPU 사용권이 스레드1로 넘어가서 A의 값을 출력하면 어떻게 될까? 스레드1의 입장에서는 갑자기 20이라는 값이 출력되는 문제가 발생한다.

![multi-process-and-multi-thread-4]({{site.url}}/images/2021-06-05-difference-between-multi-process-and-multi-thread/images_jsj3282_post_2a2446d0-6169-42ad-aebc-081c909a5b15_image (2).png)

이처럼 쓰기 가능한 전역 변수를 사용하게 되면 ```스레드 안전성이 깨진다```라고 표현한다. 물론 이를 보완하는 방법으로 ```락(lock)```을 거는 방법이 있지만 락을 거는 순간 멀티 스레드의 장점은 버린 것과 같다.



참고

- 스프링 입문을 위한 자바 객체지향의 원리와 이해

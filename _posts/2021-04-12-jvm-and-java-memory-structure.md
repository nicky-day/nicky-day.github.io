---
layout: single
title: "JMV과 자바 메모리 구조(Runtime Data Area)"
categories: Java
tag: [JVM]
author_profile: false
sidebar:
    nav: "counts"
search: true
use_math: false
---

### 1. JVM이란?

Java의 가장 큰 장점은 OS에 종속적이지 않고 Java 파일 하나만 만들면 어느 디바이스든 JVM 위에서 실행 할 수 있다는 것이다. ```JVM```이란 Java Virtual Machine, 자바 가상머신의 약자이다. JVM은 Java Byte Code를 OS에 맞게 해석 해주기 때문에 자바는 운영체제와 독립적으로 동작이 가능하다. 

Java compiler는 .java 파일을 .class 라는 Java byte code로 변환 시켜 준다다. 그러나 Byte Code는 기계어가 아니기 때문에 OS에서 바로 실행되지 않는다. 이때 JVM은 OS가 ByteCode를 이해할 수 있도록 해석해준다. JVM의 역할은 자바 애플리케이션을 클래스 로더를 통해 읽어 들여 자바 API와 함께 실행하는 것이다. 하지만 JVM의 해석을 거치기 때문에 c언어 같은 네이티브 언어에 비해 속도가 느렸지만 JIT(Just In Time)컴파일러를 구현해 이점을 극복했다. 

JVM은 크게 Class Loader, Runtime Data Areas, Excution Engine 3가지로 구성되어있다. 또한 가장 중요한 메모리관리, garbage collection을 수행한다. 또한 JVM은 스택기반의 가상머신으로 ARM 아키텍처같은 하드웨어는 레지스터로 동작하는데 비해 JVM은 스택기반으로 동작한다.

![JVM-1]({{site.url}}/images/2021-04-12-jvm-and-java-memory-structure/1_slIuYO633BCuBh_gfYRmGg.png)

### 2. 자바 프로그램 실행 과정

1. 프로그램이 실행되면 JVM은 OS로부터 이 프로그램이 필요로 하는 메모리를 할당받는다. JVM은 이 메모리를 용도에 따라 여러 영역으로 나누어 관리한다.
2. 자바 컴파일러(javac)가 자바 소스코드(.java)를 읽어들여 자바 바이트코드(.class)로 변환한다.
3. Class Loader를 통해 .class 파일을 JVM으로 로딩한다.
4. 로딩된 class 파일들은 Execution engine을 통해 해석된다.
5. 해석된 바이트코드는 Runtime Data Areas에 배치되어 실질적인 수행이 이루어진다.
이러한 실행과정 속에서 JVM은 필요에 따라 Thread Synchronization과 같은 GC 작업을 수행한다.

### 3. JVM 구성

1. Class Loader<br/>
자바 컴파일러가 .java 파일을 컴파일하면 .class 파일(바이트 코드)가 생성된다. 이렇게 생성된 클래스 파일들을 엮어 Runtime 시점에 클래스를 로딩하게 해주며 클래스 인스턴스를 생성하면 Runtime Data Area 형태로 메모리에 적재하는 역할을 한다.

2. Execution Engine<br/>
Load된 Class의 ByteCode를 실행하는 Runtime Module이다. Class Loader를 통해 JVM 내의 Runtime Data Areas 에 배치된 바이트 코드는 Executin Engine에 의해 실행되며, Executin Engine은 메모리에 적재된 자바 바이트 코드(클래스 파일)를 기계어로 변경해 명령어 단위로 읽어서 실행한다.
최초 JVM 이 나왔을 당시에는 Interperter방식(한 줄씩 해석하고 실행)이였기 때문에 속도가 느리다는 단점이 있었지만 ```JIT complier(Just-In-Time)``` 방식을 통해 이 점을 보완했다. JIT는 ByteCode를 어셈블러 같은 NativeCode로 바꿔서 캐시에 보관하기 때문에 한 번 컴파일된 코드는 실행이 빠르지만 역시 변환하는데 비용이 발생한다. 이 같은 이유 때문에 JVM은 모든 코드를 JIT Compiler 방식으로 실행하지 않고 Interpreter 방식을 사용하다 어느 기준이 넘어가면 자주 쓸만한 코드들을 기계어로 변환시켜 놓고 저장해서 사용하는 JIT Compiler 방식으로 실행한다.

3. Runtime Data Areas<br/>
JVM이 프로그램을 수행하기 위해 OS로 부터 별도로 할당 받은 메모리 공간을 말하며, Runtime Data Areas는 크게 5가지 영역으로 나눌 수 있다.
   
   1. PC Register<br/>
    프로그램의 실행은 CPU에서 명령어, 즉 Instruction을 수행하는 과정으로 이루어진다. Instruction을 수행하는 동안 필요한 정보를 레지스터라고 하는 CPU 내의 기억장치를 사용한다. Java의 PC Register는 CPU 내의 기억장치인 레지스터와는 다르게 작동한다. Register-Base가 아닌 Stack-Base로 동작한다. **PC Register는 각 Thread 별로 하나씩 존재하며 현재 수행중인 Java Virtual Machine의 주소를 가지게 된다.** 만일 Native Method를 수행한다면 PC Register는 Undefined 상태가 된다. 이 PC Register에 저장되는 Instruction 주소는 Native Pointer 일 수도 있고 Method Bytecode 일 수도 있다. Native Method를 수행할 때에는 JVM을 거치지 않고 API를 통해 바로 수행하게 된다. 이는 Java는 Platform에 종속받지 않는다는 것을 보여준다.
   2. Java Virtual Machine Stack<br/>
   Java Virtual Machine Stacks은 Thread의 수행 정보를 Frame을 통해서 저장하게 된다. Java Virtual Machine Stacks는 Thread가 시작될 때 생성되며, 각 Thread 별로 생성되기 때문에 다른 Thread는 접근할 수 없다. Method가 호출되면 Method와 Method 정보는 Stack에 쌓기에 되며, Method 호출이 종료될 때 Stack point에서 제거된다. Method 정보는 해당 Method의 매개변수, 지역변수, 임시변수 그리고 어드레스(메소드 호출한 주소) 등을 저장하고 Method 종료 시 메모리 공간이 사라진다.
   3. Native Method Stack<br/>
   Java 외의 언어로 작성된 네이티브 코드들을 위한 Stack, 즉 JNI(Java Native Interface)를 통해 호출되는 C/C++ 등의 코드를 수행하기 위한 Stack이다. Natvie Method를 위해 Native Method Stack이라는 메모리 공간을 갖는다. Application에서 Natvie Method를 호출하게 되면 Native Method Stack에서 새로운 Stack Frame을 생성하여 push한다. 이는 JNI를 이용하여 JVM 내부에 영향을 주지 않기 위함이다.
   4. Method Area<br/>
   모든 쓰레드가 공유하는 메모리 영역이다. Method Area는 클래스, 인터페이스, 메소드, 필드, Static 변수 등 바이트 코드 등을 보관한다.
   5. Heap<br/>
   프로그램 상에서 런타임시 동적으로 할당하여 사용하는 영역이다. class를 이용해 instance를 생성하면 Heap에 저장된다. (ex. ClassA a = new ClassA();)
    
### 4. GC(Garbage Collector)

#### Heap(힙 영역)

인스턴스는 소멸 방법과 소멸 시점이 지역변수와 다르기 때문에 힙이라는 별도의 영역에 할당된다.

- 인스턴스와 배열이 동적으로 생성되는 공간
- 생성에 필요한 인스턴스와 배열의 메타 정보는 Heap 내의 Method Area에서 얻어온다.
- 모든 Thread가 공유하기 때문에 동기화 문제가 발생할 수 있다.
- Garbage Collection의 대상이 되는 영역
- 개발자는 객체를 제거하기 위해 별도의 코드를 작성할 필요가 없고 GC에 맡기면 된다.

힙은 객체를 저장하는 가상 메모리 공간이다. new 연산자로 생성된 객체와 배열을 저장한다. 물론 class area 영역에 올라온 클래스들만 객체로 생성할 수 있다. 힙은 세 부분으로 나눌 수 있다.

![JVM-2]({{site.url}}/images/2021-04-12-jvm-and-java-memory-structure/다운로드.jpeg)

![JVM-3]({{site.url}}/images/2021-04-12-jvm-and-java-memory-structure/993ADD3E5C7681222D.jpeg)

1. Permanent Generation
Perm 영역은 보통 Class Meta 정보나 Method의 메타 정보, static 변수와 상수 정보들이 저장되는 공간으로 흔히 메타데이터 저장 영역이라고 한다. 이 영역은 Java8부터 Native Memory 영역으로 이동하였다. (기존의 Perm 영역에 존재하는 static object는 Heap 영역으로 옮겨졌다.)


2. New/Young 영역
- Eden : Object(객체)가 최초로 Heap에 할당되는 장소이다. 만일 Eden 영역이 가득 찼다면, Object의 참조 여부를 파악하고 Live Object는 Survivor 영역으로 넘긴다. 그리고 참조가 사라진 Garbage Object이면 남겨 놓는다. 그리고 모든 Live Ojbect가 Survivor 영역으로 넘어갔다면 Eden 영역을 모두 청소한다.
- Survivor 0/1 : Survivor 영역은 Survivor0과 Survivor1로 구성되며 Eden 영역에 살아남은 Object들이 잠시 머무르는 곳이며 Live Object들은 하나의 Subject 영역만 사용하게 되며 이러한 전반적인 과정을 Minor GC라고 한다.

3. Old 영역
Old 영역은 새로 Heap에 할당된 Object가 들어오는 것이 아닌, Survivor 영역에서 살아남아 오랫동안 참조되었고 앞으로도 사용될 확률이 높은 Object들을 저장하는 영역이다. 이러한 Promotion 과정 중 Old 영역의 메모리가 충분하지 않으면 해당 영역에서 GC가 발생하는데 이를 Major GC라고 한다. (Tenured 영역에서 발생한 GC)

#### GC의 종류

1. Minor GC<br/>
새로 생성된 대부분의 객체(Instance)는 Eden 영역에 위치한다. Eden 영역에서 GC가 한 번 발생한 후 살아남은 객체는 Survivor 영역 중 하나로 이동된다. 이 과정을 반복하다가 계속해서 살아남아 있는 객체는 일정시간 참조되고 있다는 뜻이므로 Old 영역으로 이동시킨다.

1. Major GC<br/>
Old 영역에 있는 모든 객체들을 검사하여 참조되지 않은 객체들을 한꺼번에 삭제한다. 시간이 오래걸리고 실행 중인 프로세스가 정지된다. 이것을 'stop-the-world'라고 하는데 Major GC가 발생하면 GC를 실행하는 쓰레드를 제외한 나머지 쓰레드는 모두 작업을 멈춘다. GC 작업을 완료한 이후에야 중단했던 작업을 다시 실행한다.

#### Garbage Collection의 동작 원리

자바 객체의 2가지 특징으로 인해 객체의 생명주기를 직접 관리해서 불필요한 객체를 메모리에서 해재시키도록 하는 개념(GC)가 등장하게 되었다. 첫 번째 특징은 대부분의 객체는 금방 접근 불가능 상태(unreachable)가 되는 것이다. 두 번째 특징은 오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재한다는 것이다. 즉 Heap의 인스턴스 중 Thread의 JVM Stack에서 도달할 수 없는 것들(unreachable)이 GC의 대상이 된다. 중요한 건 도달할 수 없는 것들을 pick 하는 것이 아니라 도달할 수 있는 것들을 pick 한 후 나머지들을 메모리에서 제거한다. 이 방식을 ```Mark and Sweep```라고 한다.
Garbage Collector는 힙 내의 객체 중에서 가비지(Garbage)를 찾아내고 찾아낸 가비지를 처리해서 힙의 메모리를 회수한다. 참조되고 있지 않는 객체(Instance)를 가비지라고 하며 객체가 가비지인지 아닌지 판단하기 위해서 ```reachability```라는 개념을 사용한다. 어떤 힙 영역에 할당된 객체가 유효한 참조가 있으면 reachability, 없다면 unreachablity로 판단한다. 하나의 객체는 다른 객체를 참조하고, 다른 객체는 또 다른 객체를 참조할 수 있기 때문에 참조 사슬이 형성되는데, 이 참조 사슬 중 최초에 참조한 것을 Root Set이라고 한다. 힙 영역에 있는 객체들은 총 4가지 경우에 대한 참조를 하게 된다.

![JVM-4]({{site.url}}/images/2021-04-12-jvm-and-java-memory-structure/99B649355C6708ED08.png)

1. 각각의 Thread의 Stack에 있는 변수가 가르키는 refernece, 즉 Java 메서드 실행 시에 사용하는 지역변수와 파라미터들에 의한 reference
2. 메소드 영역의 정적 변수가 가르키는 reference
3. native stack이 가르키는 reference
4. 힙 내의 다른 객체에 의한 참조

4번의 경우 무조건 적으로 자신을 가리키는 더 상위객체가 존재하게된다. 그러나 1, 2, 3번의 경우 반드시 최상위 노드가 되는데(자기 자신보다 더 상위 객체가 없음) 이러한 최상위 진입점을 ```Root Set```이라고 한다.

따라서 Root Set부터 그래프 구조를 순회하면서 모조리 마크하면서 체크한다. 이렇게 마킹된 객체를 Reachable Object라고 부르고 반대로 마킹된 객체들을 제외한 객체들을 Unreachable Object라고 부른다.

참고
- [https://www.holaxprogramming.com/2013/07/16/java-jvm-runtime-data-area/](https://www.holaxprogramming.com/2013/07/16/java-jvm-runtime-data-area/)
- [https://jithub.tistory.com/40](https://jithub.tistory.com/40)
- [https://smjeon.dev/etc/jvm-rda/](https://smjeon.dev/etc/jvm-rda/)
- [https://medium.com/@lazysoul/jvm-%EC%9D%B4%EB%9E%80-c142b01571f2](https://medium.com/@lazysoul/jvm-%EC%9D%B4%EB%9E%80-c142b01571f2)
- [https://velog.io/@litien/JVM-%EA%B5%AC%EC%A1%B0](https://velog.io/@litien/JVM-%EA%B5%AC%EC%A1%B0)
- [https://smjeon.dev/etc/jvm-rda/](https://smjeon.dev/etc/jvm-rda/)
- [https://asfirstalways.tistory.com/159](https://asfirstalways.tistory.com/159)
- [https://jithub.tistory.com/296](https://jithub.tistory.com/296)
- [https://drkein.tistory.com/95](https://drkein.tistory.com/95)
- [https://swiftymind.tistory.com/112](https://swiftymind.tistory.com/112)
- [https://kamang-it.tistory.com/entry/Java%EC%97%90%EC%84%9C%EC%9D%98-%EA%B0%80%EB%B9%84%EC%A7%80%EC%BB%AC%EB%A0%89%ED%84%B0Garbage-CollectorGC%EB%8F%8C%EC%95%84%EA%B0%80%EB%8A%94-%EC%9B%90%EB%A6%AC-%ED%8C%8C%ED%95%B4%EC%B9%98%EA%B8%B0](https://kamang-it.tistory.com/entry/Java%EC%97%90%EC%84%9C%EC%9D%98-%EA%B0%80%EB%B9%84%EC%A7%80%EC%BB%AC%EB%A0%89%ED%84%B0Garbage-CollectorGC%EB%8F%8C%EC%95%84%EA%B0%80%EB%8A%94-%EC%9B%90%EB%A6%AC-%ED%8C%8C%ED%95%B4%EC%B9%98%EA%B8%B0)
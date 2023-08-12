---
layout: single
title: "TCP/UDP와 3-way-handshake/4-way-handshake"
categories: Network
tag: [TCP/IP, TCP, UDP]
author_profile: false
sidebar:
    nav: "counts"
search: true
use_math: false
---

### 1. TCP/IP

<img src="{{site.url}}/images/2021-04-12-tcp-and-udp-and-3-way-handshake-and-4-way-handshake/tpc-ip-and-osi-model-cellbiol.com_.png" alt="tcp-and-udp1" width="650px">

```TCP/IP```는 패킷 통신 방식의 인터넷 프로토콜인 IP (인터넷 프로토콜)와 전송 조절 프로토콜인 TCP (전송 제어 프로토콜)로 이루어져 있다. IP는 패킷 전달 여부를 보증하지 않고, 패킷을 보낸 순서와 받는 순서가 다를 수 있다.(unreliable datagram service) TCP는 IP 위에서 동작하는 프로토콜로, 데이터의 전달을 보증하고 보낸 순서대로 받게 해준다. HTTP, FTP, SMTP 등 TCP를 기반으로 한 많은 수의 애플리케이션 프로토콜들이 IP 위에서 동작하기 때문에, 묶어서 TCP/IP로 부르기도 한다.

TCP/IP의 Transport 계층에서 TCP와 UDP 프로토콜이 사용된다. Transport 계층은 IP에 의해 전달되는 패킷의 오류를 검사하고 재전송 요구 등의 제어를 담당하는 계층이다.

### 2. TCP와 UDP

![tcp-and-udp2]({{site.url}}/images/2021-04-12-tcp-and-udp-and-3-way-handshake-and-4-way-handshake/TCP-VS-UDP.png)

TCP는 Transmission Control Protocol의 약자이고, UDP는 User Datagram Protocol의 약자이다. 두 프로토콜은 모두 패킷을 한 컴퓨터에서 다른 컴퓨터로 전달해주는 IP 프로토콜을 기반으로 구현되어 있지만, 서로 다른 특징을 가지고 있다.

#### TCP와 UDP의 특징

|<center>TCP</center>                                         | <center>UDP</center>                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 연결형(connection-oriented) 프로토콜<br> - 연결이 성공해야 통신 가능 | 비연결형(connectionless) 프로토콜<br> - 연결 없이 통신 가능  |
| 데이터 경계를 구분하지 않음<br> - 바이트 스트림(byte-stream) 서비스 | 데이터 경계를 구분함<br> - 데이터그램(datagram) 서비스       |
| 신뢰성 있는 데이터 전송<br> - 데이터를 재전송함              | 비신뢰적인 데이터 전송<br> - 데이터를 재전송하지 않음        |
| 1 대 1 통신(unicast)                                         | 1 대 1 통신(unicast)<br> 1 대 다 통신(broadcast)<br> 다 대 다 통신(multicast) |

#### TCP란?

```TCP(Trasmission Control Protocol)```는 연결 지향적 프로토콜이다. 연결 지향 프로토콜이란 클라이언트와 서버가 연결된 상태에서 데이터를 주고받는 프로토콜을 의미한다. 클라이언트가 연결 요청(SYN 데이터 전송)을 하고, 서버가 연결을 수락하면 통신 선로가 고정되고, 모든 데이터는 고정된 통신 선로를 통해서 순차적으로 전달된다. 그렇기 때문에 TCP는 데이터를 정확하고 안정적으로 전달할 수 있다. TCP는 호스트간 신뢰성 있는 데이터 전달과 흐름제어를 한다. TCP 프로토콜은 신뢰성 있는 데이터의 전송을 위해 확인작업을 거치는데 TCP는 패킷을 성공적으로 전송하면(ACK) 라는 신호를 날리고 만약에 ACK 신호가 제 시간에 도착하지 않으면 Timeout이 발생하여 패킷 손실이 발생한 패킷을 다시 전송해준다. TCP는 이렇게 데이터를 송신할때마다 확인 응답을 주고받는 절차가 있으므로 통신의 신뢰성이 올라간다. 주로 Client와 Server 또는 P2P Socket 통신 등, 네트워크를 사용한 통신을 할 때 TCP 통신을 많이 사용한다.


#### TCP의 단점
- 데이터로 보내기 전에 반드시 연결이 형성되어야함.
- 1 : 1 통신만 가능함.
- 고정된 통신 선로가 최단선(네트워크 길이)이 아닐경우 상대적으로 UDP보다 데이터 전송속도가 느림.

#### TCP의 특징
- 연결형 (connnection-oriented) 서비스로 연결이 성공해야 통신이 가능하다.
- 데이터의 경계를 구분하지 않는다. (바이트 스트림 서비스)
데이터의 전송 순서를 보장한다. (데이터의 순서 유지를 위해 각 바이트마다 번호를 부여)
- 신뢰성있는 데이터를 전송한다. (Sequence Number, Ack Number를 통한 신뢰성 보장)
- 데이터 흐름 제어(수신자 버퍼 오버플로우 방지) 및 혼잡 제어(패킷 수가 과도하게 증가하는 현상 방지)
- 연결의 설정(3-way handshaking)과 해제(4-way handshaking)
- 전이중(Full-Duplex), 점대점(Point to Point) 서비스
- UDP보다 전송속도가 느리다.

#### 전이중, 점대점 방식
- 전이중 (Full-Duplex) : 전송이 양방향으로 동시에 일어날 수 있다.
- 점대점 (Point to Point) : 각 연결이 정확히 2개의 종단점을 가지고 있다.

#### UDP란?

```UDP(User Datagram Protocol)```는 전송계층의 비연결 지향적 프로토콜이다. 비연결 지향적이란 데이터를 주고받을 때 연결 절차를 거치지 않고 발신자가 일방적으로 데이터를 발신하는 방식을 의미한다. 연결 과정이 없기 때문에 TCP보다는 빠른 전송을 할 수 있지만 데이터 전달의 신뢰성은 떨어진다. UDP는 발신자가 데이터 패킷을 순차적으로 보내더라도 이 패킷들은 서로 다른 통신 선로를 통해 전달 될 수 있다. 먼저 보낸 패킷이 느린 선로를 통해 전송될 경우 나중에 보낸 패킷보다 늦게 도착할 수 있으며 최악의 경우 잘못된 선로로 전송되어 유실될 수도 있다. 이럴 경우 TCP와는 다르게 UDP는 중간에 패킷이 유실이나 변조가 되어도 재전송을 하지 않는다. 

#### UDP의 단점
- 데이터의 신뢰성이 없다.
- 의미있는 서버를 구축하기위해서는 일일이 패킷을 관리해주어야 한다.

#### UDP의 특징
- 비연결형 서비스로 연결 없이 통신이 가능하며 데이터그램 방식을 제공한다.
- 데이터 경계를 구분한다. (데이터그램(datagram) 서비스)
- 정보를 주고 받을때 정보를 보내거나 받는다는 신호절차를 거치지 않는다.
- 신뢰성 없는 데이터를 전송한다. (데이터 재전송과 데이터 순서 유지를 위한 작업을 하지 않는다.
- 패킷관리가 필요하다.
- 패킷 오버헤드가 적어 네트워크 부하가 감소되는 장점이 있다.
- 상대적으로 TCP보다 전송속도가 빠르다.

### 3. 3-way-handshake와 4-way-handshake

3-way-handshake와 4-way-handshake는 TCP의 연결 및 해제 과정이다. 

![tcp-and-udp3]({{site.url}}/images/2021-04-12-tcp-and-udp-and-3-way-handshake-and-4-way-handshake/2.png)

#### TCP Connection (3-way handshake)

1. 먼저 open()을 실행한 클라이언트가 SYN을 보내고 SYN_SENT 상태로 대기한다.
2. 서버는 SYN_RCVD 상태로 바꾸고 SYN과 응답 ACK를 보낸다.
3. SYN과 응답 ACK을 받은 클라이언트는 ESTABLISHED 상태로 변경하고 서버에게 응답 ACK를 보낸다.
4. 응답 ACK를 받은 서버는 ESTABLISHED 상태로 변경한다.

#### TCP Disconnection (4-way handshake)

1. 먼저 close()를 실행한 클라이언트가 FIN을 보내고 FIN_WAIT1 상태로 대기한다.
2. 서버는 CLOSE_WAIT으로 바꾸고 응답 ACK를 전달한다. 동시에 해당 포트에 연결되어 있는 어플리케이션에게 close()를 요청한다.
3. ACK를 받은 클라이언트는 상태를 FIN_WAIT2로 변경한다.
4. close() 요청을 받은 서버 어플리케이션은 종료 프로세스를 진행하고 FIN을 클라이언트에 보내 LAST_ACK 상태로 바꾼다.
5. FIN을 받은 클라이언트는 ACK를 서버에 다시 전송하고 TIME_WAIT으로 상태를 바꾼다. TIME_WAIT에서 일정 시간이 지나면 CLOSED된다. ACK를 받은 서버도 포트를 CLOSED로 닫는다.

참고
- https://needjarvis.tistory.com/157
- https://ko.wikipedia.org/wiki/%EC%9D%B8%ED%84%B0%EB%84%B7_%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C_%EC%8A%A4%EC%9C%84%ED%8A%B8
- https://velog.io/@hidaehyunlee/TCP-%EC%99%80-UDP-%EC%9D%98-%EC%B0%A8%EC%9D%B4
- https://coding-factory.tistory.com/614
- https://velog.io/@hidaehyunlee/TCP-%EC%99%80-UDP-%EC%9D%98-%EC%B0%A8%EC%9D%B4

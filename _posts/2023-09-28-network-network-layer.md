---
layout: single
title: "네트워크 계층(Layer 3)"
categories: Network
author_profile: false
sidebar:
    nav: "counts"
search: true
use_math: false
---

## 1. 네트워크 계층

MAC 주소와 스위치를 이용하면 브로드캐스트 도메인을 나누지 못해 많은 컴퓨터가 연결되었을 때 너무 많은 트래픽으로 통신이 마비될 수 있다. 네트워크 계층은 브로드캐스트 도메인을 나누기 위해 등장했다. 논리적 주소인 IP 주소를 만들어 다른 브로드캐스트 도메인의 장치를 가리킬 수 있게 되었고, 브로드캐스트 도메인을 나누는 어떤 장치에 라우터라는 이름을 붙여 다른 브로드캐스트 도메인으로 최적의 경로를 계산해 전송한다. 



다른 컴퓨터에게 데이터를 전송하기 위해 출발지 MAC 주소에는 나의 MAC 주소, 목적지 MAC 주소에는 다른 컴퓨터의 MAC 주소가 아닌 라우터의 MAC 주소를 적고 전송한다. 그러면 스위치를 통해 라우터로 전송되는데 라우터는 데이터의 목적지 MAC 주소를 보고 자신에게 온 데이터임을 확인하고 목적지 IP를 살펴본다. 그리고 받은 메시지에서 IP 주소는 바꾸지 않고 출발지 MAC 주소는 라우터의 MAC 주소, 도착지 MAC 주소는 다른 컴퓨터의 MAC 주소를 적고 스위치로 전송한다. 이렇듯 네트워크 계층은 다른 LAN 영역을 연결하기 위해 논리적인 주소인 IP 주소를 만들어 라우터라는 장비를 이용하는 계층이다.



호스트는 자신이 속한 브로드캐스트 도메인을 LAN(Local Area Network)라고 부르고 라우터 너머에 있는 다른 브로드캐스트 도메인을 WAN(Wide Area Network)라고 부른다.



## 2. IP

IP(Internet Protocal) 주소는 브로드캐스트 도메인을 넘어 다른 WAN 영역으로 가기 위해 모든 호스트에게 부여하는 논리적인 주소이다. 데이터를 전송하면 출발지 IP 주소와 목적지 IP 주소는 중간에 바뀌지 않고 해당 목적지 IP를 참조해 실제 데이터는 MAC 주소를 이용해 전송한다.



IP 주소는 32비트로 구성되어 있으며 각 8비트를 10진수로 표현하고 닷(.)으로 구분한다. ex) 192.168.0.1

컴퓨터의 IP 주소를 확인하는 방법은 터미널에 ipconfig -all을 입력하면 된다.



## 3. IP 클래스와 서브넷 마스크

IP 주소는 8비트 4개로 총 32비트로 구성되어 있다. 8비트는 2의 8승으로 0부터 255까지 256개의 수를 표현할 수 있다. 따라서 IP 주소는 0.0.0.0부터 255.255.255.255까지 대략 43억개가 될 수 있다. 43억개나 되는 IP 주소를 효율적으로 관리하기 위해 클래스라는 것이 등장했다. 클래스는 A, B, C, D로 총 4가지가 있다.



- 클래스 A
  - 클래스 A는 32비트 중 가장 앞에 있는 비트가 0으로 시작하는 클래스이다.
  - 가장 앞 8비트를 네트워크 주소로 사용하고 나머지 24비트를 호스트 주소로 사용해 128(2^8)개의 네트워크로 구성되며, 각 네트워크는 16,777,216(2^24)개의 호스트에 주소를 부여할 수 있다.
  - 클래스 A는 0.0.0.0 ~ 127.255.255.255 범위에 있다.
- 클래스 B
  - 클래스 B는 32비트 중 가장 앞에 있는 비트가 10으로 시작하는 클래스이다.
  - 가장 앞 16비트를 네트워크 주소로 사용하고 나머지 16비트를 호스트 주소로 사용해 네트워크 주소를 나타내는 16비트에서 가장 앞 비트는 1, 0으로 고정되어 있으므로 16,384(2^14)개의 네트워크로 구성되며, 각 네트워크는 65,536(2^16)개의 호스트에 주소를 부여할 수 있다.
  - 클래스 B는 128.0.0.0 ~ 191.255.255.255 범위에 있다.
- 클래스 C
  - 클래스 C는 32비트 중 가장 앞에 있는 비트가 110으로 시작하는 클래스이다.
  - 가장 앞 24비트를 네트워크 주소로 사용하고 나머지 8비트를 호스트 주소를 사용해 2,097,152(2^21)개의 네트워크로 구성되며, 각 네트워크는 256(2^8)개의 호스트에 주소를 부여할 수 있다.
  - 클래스 C는 192.0.0.0 ~ 223.255.255.255 범위에 있다.
- 클래스 D
  - 클래스 D는 32비트 중 가장 앞에 있는 비트가 1110으로 시작하는 클래스이다.
  - 32비트 모두를 네트워크 주소로 사용하고 일반적인 용도가 아닌 멀티캐스트 용도로 사용되는 특수한 클래스이다.
  - 클래스 D는 240.0.0.0 ~ 255.255.255.255 범위에 있다.



실제로는 호스트 할당 개수에서 2개를 빼야 하는데, 그 이유는 모든 비트가 0인 경우는 해당 네트워크를 나타내는 주소로, 모든 네트워크가 1인 경우는 해당 네트워크의 모든 호스트를 나타내는 브로드캐스트 주소로 사용하기로 약속되었기 때문이다.



IP 주소에 클래스를 표현하고 싶다면 IP 주소 뒤에 슬래시(/)를 적고 IP 주소의 앞에서 몇 번째까지가 네트워크 주소로 사용되는지 적어주면 된다. 만약 10.5.1.2라는 클래스 A의 IP 주소를 표기한다면 10.5.1.2/8로, 192.168.0.4라는 클래스 C의 IP 주소를 표기한다면 192.168.0.4/24로 표기한다.



클래스 덕분에 네트워크를 사용하는 목적과 규모에 따라서 적절히 나눌 수 있지만 세분화해서 나누지는 못한다. 만약 300개의 호스트가 필요한 네트워크라면 클래스 C는 254개의 호스트를 할당할 수 있으므로 부족하고 클래스 B는 65,534개나 할당할 수 있으므로 65,234개가 낭비되는 상황이 발생한다. 이런 이유로 네트워크와 호스트를 세분화해서 나누기 위해 서브넷 마스크라는 것을 만들었다. 



서브넷 마스크는 클래스로 나누기에 한계가 있어서 등장한 기술로, 만약 300개의 호스트가 필요해 130.5.0.0/16으로 65,534개의 호스트를 부여할 수 있는 클래스 B의 네트워크를 할당받은 상황이라면 여기서 서브넷 마스크를 이용해 IP주소와 각 비트를 AND 연산을 하고 연산 결과를 네트워크 부로 사용한다. 예를 들어 서브넷 마스크가 255.255.254.0이라면 네트워크부는 130.5.0.0이 되고 2^7개의 호스트 주소를 할당받을 수 있어 130.5.0.0 ~ 130.5.1.255까지 호스트 범위가 된다.



서브넷 마스크로 IP를 절약하기는 했지만 한 사람만 보더라도 그 사람이 사용하는 PC, 노트북, 핸드폰, 프린터 등 IP 주소를 여러 개 사용한다. 이렇게 사용자의 모든 장치에 고유한 IP를 부여하면 너무 많은 IP 주소가 사용되므로 외부와 통신하는 곳에서만 고유한 IP를 부여하고 LAN 환경에서는 가상의 IP를 부여하기로 한다. 여기서 가상의 IP를 프라이빗 IP라고 한다. 그리고 외부와 통신하는 곳에 부여하는 고유한 IP를 퍼블릭 IP라고 한다. 



프라이빗 IP 주소의 범위는 아래와 같이 정해져 있다.

- 클래스 A : 10.0.0.0 ~ 10.255.255.255/8 
- 클래스 B : 172.16.0.0 ~ 172.31.255.255/12 
- 클래스 C : 192.168.0.0 ~ 192.168.255.255/16



LAN 영역에서 사용하는 프라이빗 IP 주소는 라우터를 통해 다른 네트워크로 이동하기 전에 퍼블릭 IP로 바뀌어서 나가게 되고 이를 NAT라고 한다. NAT 덕분에 사용자의 모든 기기에 퍼블릭 IP를 부여하지 않을 수 있게 되어서 IP 주소를 절약할 수 있게 되었다. 하지만 이 방법은 NAT Traversal이라는 문제가 발생하는 경우가 있고 IP가 부족한 근본적인 해결 방법이 되지 못하므로 IPv6가 개발되었다. 기존에는 IPv4를 쓰고 있었기 때문에 현재는 IPv4와 IPv6를 혼합해 사용하고 있다.



## 4. IP 헤더(패킷)

데이터를 전송한다면 애플리케이션 계층에서 데이터가 만들어지고 이 데이터는 트랜스포트 계층으로 전달된다. 트랜스포트 계층에서는 받은 데이터에 트랜스포트 계층의 약속을 나타내는 헤더를 붙이고 이렇게 만들어진 전체 데이터를 데이터그램이라고 부른다. 데이터그램은 다시 네트워크 계층으로 전달되어 네트워크 계층에서는 받은 데이터그램에 네트워크 계층의 약속을 나타내는 IP 헤더를 붙이고 이렇게 만들어진 데이터는 패킷이라고 부른다. 



<img src="{{site.url}}/images/2023-09-28-network-network-layer/ip_header.png" alt="network-network-1" width="700px" />


- Version : IP 헤더의 버전을 나타내는 필드로 4bit로 구성된 필드이다. 우리가 사용하고 있는 것은 IPv4이므로 4(0110)를 사용한다. 만약 IPv6을 사용하고 있다면 6(0110)이 된다.
- Header Length : 현재 IP 헤더의 크기를 나타내는 4bit로 구성된 필드이다. 단위는 32비트이다. 대부분 IP 헤더의 길이는 20Byte 이므로 이 값은 거의 5로 고정되어 있다(5 * 4Byte = 20).
- TOS(Type Of Service) : 이 필드는 서비스의 우선순위를 나타내는데 거의 사용되지는 않는다.
- Total Packet Length : 이 필드는 IP 패킷의 크기를 바이트 단위로 나타내는 16비트로 나타내는 필드이다. 16비트이므로 최대 패킷의 크기는 2^16 = 65535Byte가 패킷 하나의 최대 크기이다.
- Identifier : 이 필드는 16비트로 구성되어 있으며 IP는 패킷을 분할할 수 있는 기능이 있는데, Identifier 필드는 분할된 패킷을 구분하기 위한 용도로 사용된다.
- Flag : 이 필드는 3비트로 구성되어 있으며 패킷을 분할할지, 하지 않을지를 나타낸다.
- Fragment Offset : 이 필드는 13비트로 구성되어 있으며 분할된 패킷이 원래 데이터에서 얼마나 떨어져 있는지를 나타내므로 분할 전 데이터그램을 복원할 때 사용된다.
- Time To Live : 이 필드는 8비트로 구성되어 있으며 몇 개의 라우터를 이동할 수 있는지 나타낸다. 만약 이 필드의 값이 5라면 5개의 라우터까지 통과할 수 있고 0이 되면 이 패킷은 버려진다.
- Protocol ID : 이 필드는 8비트로 구성되어 있으며 상위계층의 프로토콜이 어떤 프로토콜인지 나타낸다.
- Header Checksum : 이 필드는 16비트로 구성되어 있으며 IP 헤더가 손상되었는지 체크한다.
- Source IP Address : 출발지 IP 주소를 나타내는 32비트로 구성된 필드이다.
- Destination IP Address : 목적지 IP 주소를 나타내는 32비트로 구성된 필드이다.
- Options : 주로 사용되지는 않지만 테스트용으로 사용되는 가변길이 필드이다.
- Padding : Options를 사용하는 경우 길이가 맞지 않을 때 덧붙이는 용도로 사용되는 필드이다.
- Data : 상위계층에서 전달된 데이터를 나타낸다.



참고 : [https://www.eit.lth.se/ppplab/IPHeader.htm](https://www.eit.lth.se/ppplab/IPHeader.htm)



## 4. 라우터

<img src="{{site.url}}/images/2023-09-28-network-network-layer/router.png" alt="network-network-2" width="400px" />


라우터는 브로드캐스트 도메인을 나눌 수 있는 네트워크 계층의 장치이다. 라우터는 CPU와 메모리가 있는 하나의 컴퓨터이다. 오직 네트워크에서 데이터를 전송하기 위한 컴퓨터이다. 라우터는 사용하는 곳에 따라서 크기와 가격이 천차만별이며 우리나라에서는 라우터를 공유기라고도 부른다. 



스위치로 만들어진 브로드캐스트 도메인은 LAN 영역이라고 부르고 이런 LAN 영역을 넘어서 통신하기 위해서 라우터로 연결되고 라우터를 넘어간 다른 영역을 WAN 영역이라고 부른다.



인터넷에서는 수많은 라우터가 존재하는데 ISP(통신사)들이 설치한 고성능 라우터들을 코어 라우터라고 부르고 여러 ISP의 코어 라우터를 연결한 망을 백본이라고 부른다. 



데이터를 전송할 때 수많은 라우터 중 어떤 경로를 선택해야 최대한 빨리 갈 수 있는지 결정하는 것이 라우팅 프로토콜이다.



## 5. 라우팅 프로토콜

라우터와 IP 주소를 이용해 다른 네트워크까지 데이터를 전송할 수 있다. 라우터는 목적지 네트워크까지 가는 경로를 라우팅 테이블이라는 곳에 미리 저장해두고 데이터가 들어오면 해당 네트워크로 전달하는 역할을 한다. 

그렇다면 라우팅 테이블은 어떤 것이고 어떻게 만들어지는 것일까? 라우팅 테이블은 어떤 네트워크에서 어떤 네트워크로 이동할 때 해당 라우터에서 어떤 인터페이스로 이동해야 하는지가 적혀 있는 표이다. 라우팅 테이블을 만드는 것은 라우팅 프로토콜을 이용하는데 라우팅 프로토콜은 크게 두 가지로 나뉜다.

라우팅 프로토콜

- 첫번째는 스태틱 라우팅으로 네트워크 관리자가 직접 라우터의 라우팅 테이블에 경로를 입력하는 방식이다. 
- 두번째는 다이내믹 라우팅으로 라우터들이 다른 라우터들과 정보를 공유해 라우터 스스로 라우팅 테이블을 만드는 방식이다.

또한 라우팅 테이블의 목적지와 일치하는 정보가 저장되어 있지 않으면 디폴트 라우터로 데이터를 보내게 된다. 라우팅 테이블의 디폴트 라우터는 0.0.0.0 혹은 default로 표현한다. 디폴트 라우터 외에도 다른 약속된 주소가 있는데 바로 127.0.0.1로 이를 루프백 주소라고 부르며 로컬 호스트라고 부른다. 루프백 주소는 127.0.0.0부터 127.255.255.255까지 있지만 보통 127.0.0.1을 사용한다. 이 주소로 데이터를 전송한다면 네트워크로 나가지 않고 자신이 다시 받게 된다. 루프백 주소는 개발할 때 주로 사용된다.



## 6. 스태틱 라우팅

스태틱 라우팅은 네트워크 관리자(사람)가 직접 라우팅 테이블을 작성하는 방법이다. 사람이 수동으로 경로를 지정해주기 때문에 라우터의 CPU, 메모리를 적게 사용해 라우터에 부담이 적다. 또한 인접 라우터와 정보를 공유하지 않기 때문에 네트워크 대역폭을 절약할 수 있고 보안이 더 좋다.

하지만 다른 라우터에 문제가 생겼을 때 문제가 생긴 라우터를 우회하지 못하므로 관리자가 직접 경로를 수정해주거나 라우터를 고치기 전까지는 통신할 수 없는 상황이 발생할 수도 있다. 또한 대규모 네트워크에서는 설정이 복잡해져 소규모 네트워크에 적합한 방식이다. 이러한 단점을 해결하기 위해 다이나믹 라우팅이 등장했다.



## 7. 다이내믹 라우팅

다이내믹 라우팅은 다시 여러가지 프로토콜로 나뉜다. 인터넷상에서 라우터는 셀 수 없을 만큼 많이 존재한다. 여기서 하나의 큰 조직이 많은 라우터를 연결해 만든 네트워크를 Autonomous System, 즉 AS라고 부른다. 이 조직들은 대학교, 회사, 통신사들이 될 수 있다. 우리나라의 AS 정보는 한국인터넷진흥원에서 확인할 수 있다. 이렇게 AS가 있을 때 AS 내부에서 쓰는 라우팅 프로토콜을 Interior Routing Protocol(IRP)이라고 부른다. 그리고 서로 다른 AS를 연결하기 위한 라우팅 프로토콜을 Exterior Routing Protocol(ERP)이라고 부른다. Interior Routing Protocol에 있는 라우팅 알고리즘은 동작에 따라서 다시 두 가지 방식으로 구분되는데 한 가지는 Distance Vector 방식이고 나머지 한 가지는 Link State 방식이다. Distance Vector 방식의 대표적인 프로토콜은 Routing Information Protocol(RIP)가 있고 Link State 방식의 대표적인 프로토콜은 Open Shortest Path First(OSPF)가 있다.그리고 Exterior Routing Protocol의 방식은 Path Vector가 있는데 대표적인 프로토콜은 Border Gateway Protocol(BGP)가 있다.



- Interior Routing Protocol

  - Distance Vector : RIP(Routing Information Protocol)
  - Link State : OSPF(Open Shortest Path First)

- Exteror Routing Protocol

  - Path Vector : BGP(Border Gateway Protocol)

  

## 8. RIP(Routing Information Protocol) 

RIP(Routing Information Protocol)는 Distance Vector 방식의 라우팅 프로토콜로 목적지까지의 거리(Distance)와 인접 라우터의 방향(Vector)를 라우팅 테이블에 저장하는 방식의 라우팅 프로토콜이다.

라우터는 라우팅 테이블의 정보를 인접 라우터와 30초마다 공유한다. 라우터는 네트워크의 모든 정보를 가지고 있지 않기 때문에 적은 메모리를 요구하고 구성이 간단해 많은 라우터에서 기본적으로 지원하는 프로토콜이다.

하지만 다음 라우터의 방향과 거리만 나타내므로 전체 경로를 알 수 없으므로 순환경로에 빠지는 문제가 발생할 수 있다. 또한 라우팅 테이블에 변화가 없어도 30초마다 인접 라우터와 라우팅 테이블을 교환하기 때문에 쓸데없는 트래픽이 많이 발생한다. 라우팅 테이블에 변화가 생겼을 때 모든 라우터가 업데이트를 완료하는 시간을 Coverage Time이라고 부르는데 RIP는 하나의 라우터를 건너기 위해서는 30초라는 시간이 걸리므로 5개의 라우터만 건너더라도 Coverage Time이 300초가 된다. 따라서 대규모 네트워크에서 사용하기가 힘들고 소규모 네트워크에서 사용되는 라우팅 프로토콜이다.



## 9. OSPF(Open Shortest Path First)

OSPF(Open Shortest Path First)는 AS 내에서 사용되는 Link State 방식의 라우팅 프로토콜이다. AS 내 라우터들은 자신과 연결된 모든 이웃 라우터에  LSA(Link State Advertisement)를 전달한다. LSA에는 라우터의 ID, 연결된 포트의 IP 주소, 비용 등이 적혀 있다. 여기서 비용이란 연결된 케이블의 속도를 말하고, 속도가 빠를수록 비용이 적다. 

모든 라우터는 서로 LSA를 교환해 이 정보를 각 라우터의 LSDB(Link State Database)에 저장한다. 모든 라우터가 LSA를 교환한다면 모두 같은 LSDB를 가지게 된다. 그리고 LSDB를 이용해서 최단경로를 계산한다. 최단경로는 다익스트라 알고리즘을 통해 계산한다. 

이렇게 모든 라우터가 다른 모든 라우터의 정보를 기억하고 최단 경로를 계산해야 하므로 RIP와는 다른 장단점이 존재한다. 장점으로는 모든 라우터에 최신 정보가 업데이트 되는 시간이 짧다. 즉, Coverage Time이 작다. 또한 RIP보다 더 적은 트래픽이 발생하고 홉에 제한이 없어서 대규모 네트워크에서 사용이 가능하다. 단점으로는 하드웨어의 성능이 좋아야 하므로 비용이 많이 들고 다른 라우팅 프로토콜보다 복잡해서 배우기 어렵다.



## 10. BGP(Border Gateway Protocol)

BGP는 서로 다른 AS를 연결하기 위한 Path Vector 방식의 라우팅 프로토콜이다. 굳이 AS를 나눠서 다른 프로토콜을 사용하는 이유는 첫번째로는 정책적인 이점이 있다. 서로 경쟁중인 AS가 있다면 이해관계에 따라서 자신들의 데이터를 다른 AS로 전달할지 하지 않을지를 컨트롤할 수 있다. 두번째 이점은 한 AS 내부 라우터에 변화가 생겼을 때 다른 AS의 모든 라우터에 데이터를 전송하는 것이 아니라 외부와 연결된 라우터만 업데이트하면 되므로 트래픽이 감소하고 AS 내부의 다른 라우터들은 다른 AS 내 라우터의 변화를 신경쓰지 않을 수 있다.

각 AS에서 외부 AS와 연결되는 라우터를 ASBR(Autonomous System Boundary Router)이라고 부른다. 여기서 ASBR들은 서로 경로 속성(Path Attributes)를 공유한다. 경로 속성에는 라우터의 우선순위나 경로 같은 값들이 존재하며 경로 속성은 연결된 모든 ASBR과 양방향으로 교환한다. 그러면 모든 경로를 이동하는 경로표가 만들어져 AS 간의 라우팅을 처리할 수 있다. AS 내부에서 사용하는 라우팅 프로토콜(IRP)은 RIP나 OSPF 등이 있지만 서로 다른 AS 간에 사용하는 라우팅 프로토콜(ERP)은 BGP 하나만 사용한다.



## 11. ICMP(Internet Control Message Protocol)

ICMP(Internet Control Message Protocol)은 네트워크에서 해당 목적지까지 데이터가 도달할 수 있는지, 도달할 수 없다면 그 이유가 무엇인지 알아내기 위한 프로토콜이다. 개발로 예를 들면 디버깅 도구라고 할 수 있다. ICMP 데이터가 전달될 때 ICMP의 헤더에 IP 헤더와 Ethernet 헤더가 붙어서 전송된다.



<img src="{{site.url}}/images/2023-09-28-network-network-layer/icmp.png" alt="network-network-2" width="700px" />


ICMP 헤더는 간단한 구조로 되어 있다.

- Type : 현재 ICMP가 어떤 타입의 ICMP인지 나타내는 8bit로 이루어진 필드이다. 만약 목적지까지 도달할 수 없다면 3이라는 숫자가 설정된다.
  - 0번 : Echo 응답을 보낼 때 사용된다. 만약 우리가 목적지에 Ping 요청을 했다면 응답 메시지에 타입 0번으로 설정되어 돌아온다.
  - 3번 : 어떤 이유로 목적지에 도달할 수 없을 때 사용한다.
  - 8번 : Echo 요청을 보낼 때 사용한다. 목적지에 Ping 요청을 보낼 때 사용된다.
  - 11번 : 요청한 패킷이 목적지까지 도달하는데 시간이 초과했을 때 사용된다.
- Code : Type 필드에 대해 자세히 설명하는 필드이다. 만약 타입이 3번, 코드가 0번이라면 타입 3번으로 목적지에 도달할 수 없다는 것을 알 수 있고 코드 0번으로 네트워크에 도달할 수 없다는 것을 알 수 있다.
- Checksum : 16bit로 이루어진 필드로 메시지에 오류가 있는지 체크하는 필드이다.
- Rest of the Headers : 타입과 코드에 따라 변경되는 필드이다.



Ping은 ICMP 메시지를 전송하는 명령어이다.



참고 : [https://en.wikipedia.org/wiki/Internet_Control_Message_Protocol](https://en.wikipedia.org/wiki/Internet_Control_Message_Protocol)



## 12. ARP(Address Resolution Protocol)

ARP(Address Resolution Protocol)은 IP 주소로 MAC 주소를 알아내기 위한 프로토콜이다. ARP 패킷(요청 메시지)에 목적지 IP 주소를 적고 MAC 주소는 브로드캐스트 주소로 설정해 전송하면 ARP 요청 메시지를 받은 호스트는 자신의 MAC 주소를 응답으로 보낸다. 

목적지의 MAC 주소를 전달받으면 ARP 캐시라는 곳에 IP 주소와 MAC 주소를 매핑시키고 다음부터 데이터를 전송할 때는 브로드캐스트가 아니라 ARP 캐시를 참조해 목적지 IP 주소의 MAC 주소를 적어서 전송한다. arp -a 명령어를 입력하면 IP와 그에 맞는 MAC 주소가 저장된 것을 확인할 수 있다. 

만약 같은 네트워크가 아니라 라우터를 건너서 다른 네트워크로 전달하는 경우에는 출발지 IP 주소는 나의 IP 주소, 출발지 MAC 주소는 나의 MAC 주소, 목적지 IP 주소는 목적지 IP 주소, 목적지 MAC 주소는 라우터의 MAC 주소로 설정된다.



참고 : [http://www.ktword.co.kr/test/view/view.php?no=2188](http://www.ktword.co.kr/test/view/view.php?no=2188)



## 13. NAT(Network Address Translation)와 PAT(Port Address Translation)

NAT(Network Address Translation)은 Public IP 주소를 Private IP 주소로, Private IP 주소를 Public IP 주소로 변환시켜주는 기술이다. NAT는 크게 아래와 같이 4가지로 나눌 수 있다.

- Static NAT
  - Static NAT은 관리자가 라우터에 직접 Private IP 주소에 Public IP를 매핑하는 방식이다.
  - Private IP와 Public IP가 1:1로 매핑되는 방식으로 외부로 나가는 패킷(Outbounding)과 LAN으로 들어오는 패킷(Inbounding) 모두에서 사용할 수 있는 NAT이다.
  - Private IP와 Public IP가 1:1로 매핑되므로 주소를 절약하기 위한 용도가 아닌 보안의 목적으로 쓰인다.
- Dynamic NAT
  - Dynamic NAT은 관리자가 여러 개의 Public IP 주소를 준비해두고 Public IP 주소가 필요한 사용자가 요청하면 남는 Public IP 주소를 빌려주는 방식이다.
  - 요청이 있을 때마다 자동으로 할당해주는 편리한 방식이지만 남는 Public IP 주소가 없는 경우 인터넷을 사용하지 못하는 사용자가 발생할 수 있다. 따라서 요즘은 쓰이지 않는 방식의 NAT이다.
- Static PAT
  - PAT(Port Address Translation)은 IP만을 이용하는 NAT와는 다르게 트랜스포트 계층의 포트(애플리케이션 계층에서 말하는 포트는 애플리케이션을 구분하는 숫자이다)까지 이용해 맵핑 테이블을 작성해 주소를 구분하는 확장된 NAT이다.
  - Static PAT은 하나의 Public IP 주소에 다른 포트를 부여해 여러 Private IP 주소로 변환할 수 있고 흔히 포트포워딩이라고 불린다.
  - 사용자가 일일이 등록해줘야 하는 번거로움이 있다.
- Dynamic PAT
  - 오늘날 가장 많이 사용하는 NAT이다. Static PAT과 비슷하지만 포트는 겹치지 않는 임의의 숫자를 자동으로 할당해주는 방식이다.
  - 내부에서 외부로 통신할 때 매핑 테이블에 Private IP과 Port, Public IP과 Port, binding lifetime이 저장되며 매핑 테이블에 저장된 정보는 binding lifetime만큼 유효하다가 시간이 0이 되면 테이블에서 제거한다.
  - 이런 이유로 굉장히 편리한 방식이지만 LAN 내부에서 외부로만 '먼저' 최초 통신이 가능하므로 외부에서 먼저 최초 통신을 시도하는 경우 통신에 실패하는 단점이 있다. 이 문제를 NAT Traversal이라고 한다.
  - 이런 이유로 만약 두 사용자 모두 Dynamic PAT을 사용하는 경우 통신 자체가 불가능한 상황이 올 수도 있다.




위의 4개의 유형을 간단한 표로 정리하면 다음과 같다.

<table>
  <tr>
  	<td>유형</td>
    <td>연결 방식</td>
    <td>먼저 통신할 수 있는 방향</td>
    <td>현재 사용 여부</td>
  </tr>
  <tr>
  	<td>Static NAT</td>
    <td>1:1</td>
    <td>양방향</td>
    <td>사용됨</td>
  </tr>
  <tr>
  	<td>Dynamic NAT</td>
    <td>N:N</td>
    <td>양방향</td>
    <td>사용 안 됨</td>
  </tr>
  <tr>
  	<td>Static PAT</td>
    <td>1:N</td>
    <td>양방향</td>
    <td>사용됨</td>
  </tr>
  <tr>
  	<td>Dynamic PAT</td>
    <td>1:N</td>
    <td>단방향(Outbound Only)</td>
    <td>사용됨</td>
  </tr>
</table>



추가로 NAT은 제조사에 다라 구현하는 방식이 천차만별이다. NAT의 유형은 Full Cone, Restricted Cone, Port Restricted Cone, Symmetric Cone으로 4가지로 나뉠 수 있다. Full Cone 방식은 주소 변환 테이블에 Private IP와 Port, Public IP와 Port가 매핑되었다면 누구든 해당 Public IP와 Port로 접근하면 매핑된 Private IP와 Port에 접근할 수 있는 방식이다.



참고

- [그림으로 쉽게 배우는 운영체제](https://www.inflearn.com/course/%EB%B9%84%EC%A0%84%EA%B3%B5%EC%9E%90-%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9C/dashboard)

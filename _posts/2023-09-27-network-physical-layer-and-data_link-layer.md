---
layout: single
title: "물리 계층과 데이터링크 계층(Layer 1 ~ 2)"
categories: Network
author_profile: false
sidebar:
    nav: "counts"
search: true
use_math: false
---

## 1. 물리 계층과 데이터링크 계층

멀리 떨어져 있는 곳에 데이터를 전송한다는 것은 물리적인 무언가를 전송한다는 것이다. 물리적인 무언가는 전기나 전자기파(빛, 전파)가 될 수 있다. 일반적으로 전기신호는 케이블을 통해 보낸다. 전압을 측정해 전압이 낮으면 0, 높으면 1로 해석한다. 전기는 에너지이기도 하지만 그 자체로 신호이기 때문에 전기의 투입은 에너지 투입과 동시에 신호의 투입이라고 할 수 있다. 전파는 전자기파의 일종으로 전자기파는 전파보다 더 넓은 스펙트럼을 가지고 있다. 빛 또한 전자기파의 일종이다. 전자기파는 진동수, 즉 주파수에 따라 구분하는데 우리가 보는 빛이라는 것은 가시광선이다. 우리가 전파라고 말하는 것은 우리가 보지 못하는 주파수 대역대로 전자기파의 일종이다. 전자기파를 이용한 통신은 가시광선을 이용한 광섬유, 즉 유선통신이 있고 전파를 이용하는 무선통신이 있다. 전기신호와 전자기파신호 같은 물리적인 신호를 아날로그 신호라고 부르는데 전자기기가 처리하기 위해서는 디지털 신호로 바꿔줘야 한다.



물리 계층은 네트워크에 필요한 물리적인 작업(미디어 타입, 커넥터 타입, 신호표현 방법 등)을 담당하는 계층으로 디지털 신호를 아날로그 신호(전자, 전자기파)로 바꾸고 목적지까지 아날로그 신호를 전달한다. 그리고 목적지에서는 받은 아날로그 신호를 다시 디지털 신호로 바꾸는 처리를 한다.



데이터 계층은 같은 네트워크에서 목적지를 구분하는 계층이다. MAC 주소를 이용해 연결된 컴퓨터를 구분하고 정확히 전송할 수 있다. 데이터링크 계층은 물리 계층과 아주 밀접한 관계가 있는 계층으로 정확히 분리하기 힘든 경우도 있다. 대표적으로 LAN 카드가 있다.



## 2. 케이블

2대 이상의 컴퓨터가 서로 통신하려면 서로를 연결하는 케이블이나 전파가 있어야 한다. 이는 물리적으로 데이터를 전송하는 매체이므로 물리 계층에 해당한다. 케이블은 크게 3가지로 나뉜다. 첫번째는 UTP 케이블, 두번째는 동축 케이블, 세번째는 광 케이블이 있다.

<img src="{{site.url}}/images/2023-09-27-os-physical-layer-and-data_link-layer/cable.png" alt="network-physical-and-datalink-1" width="400px" />



UTP 케이블

- UTP 케이블은 우리 주변에서 흔히 볼 수 있는 케이블이며 랜선이라고도 부른다. 
- 데스크톱이나 노트북에 유선 인터넷을 연결하는 경우 대부분 UTP 케이블을 사용한다.
- UTP 케이블은 Unshielded Twisted Pair의 약자이다. 쉴드로 감싸지 않은 코인 구리선의 쌍이라는 뜻이다. 
- 총 4개의 쌍에서 2개의 쌍은 송신용, 2개의 쌍은 수신용이다. 송수신용이 따로 구분되었기 때문에 전이중통신이 가능한 케이블이다.
- UTP 케이블은 가격이 저렴하고 성능도 좋고 배선하기가 편하다는 장점이 있다.



<img src="{{site.url}}/images/2023-09-27-os-physical-layer-and-data_link-layer/2972_detail_096.jpg" alt="network-physical-and-datalink-2" width="400px" />

동축 케이블

- 동축(Coaxial) 케이블은 중앙에 데이터를 전송하는 하나의 구리선을 감싼 절연체로 이루어진 케이블이다.
- BNC 커넥터를 이용해 컴퓨터에 연결할 수도 있지만 요즘엔 거의 볼 수 없다. 대신 TV나 안테나에 사용돼서 안테나선이라고도 불린다.
- UTP 케이블보다 자기장 간섭이 적어서 더 멀리 전송할 수 있다. 그러나 선이 하나이기 때문에 반이중 통신만 가능하다.
- UTP 케이블보다 설치하기가 어렵고 비싸다는 단점이 있다.



<img src="{{site.url}}/images/2023-09-27-os-physical-layer-and-data_link-layer/1059318_20180406151001_107_0001.jpg" alt="network-physical-and-datalink-3" width="400px" />

광 케이블

- 광(Optical) 케이블은 유리나 플라스틱으로 만들어진 케이블로 빛을 이용하는 케이블이다. 
- 빛을 이용하기 때문에 외부 간섭이 적어서 오류 발생률이 낮고 속도도 빠르고 전송 거리도 매우 길다.
- 하지만 구부림에 약하고 전기 신호를 빛 신호로, 빛 신호를 전기 신호로 변환하는 트랜시버가 필요하다는 단점이 있다.



## 3. 랜카드

<img src="{{site.url}}/images/2023-09-27-os-physical-layer-and-data_link-layer/abe819787b309ccc1f36fff69265d062.jpg" alt="network-physical-and-datalink-4" width="400px" />

랜카드

- 랜카드는 네트워크 인터페이스 카드라고 불리는 하드웨어로 물리계층, 데이터링크 계층에 해당하는 장치이다. 
- 컴퓨터 메인보드에 내장되어 있거나 추가 슬롯에 연결해 사용한다. 
- 랜카드는 애플리케이션 계층부터 시작해 각 계층을 타고 내려온 최종 디지털 데이터인 프레임을 아날로그 데이터로 변환하기도 하고 케이블을 통해 전달된 아날로그 데이터를 디지털 데이터로 변환하는 장치이다. 
- 랜카드에는 고유한 주소인 MAC 주소가 할당되는데 이는 전세계에서 유일한 주소이다. 랜카드는 전파를 통해 전송된 아날로그 데이터를 디지털 데이터로 변환한 후 디지털 데이터에서 데이터링크 헤더에 적힌 목적지 MAC 주소가 자신의 MAC 주소와 일치하는지 확인하고 일치하면 CPU에게 데이터를 전달하고 일치하지 않으면 버린다.



## 4. MAC 주소

MAC 주소

- 데이터링크 계층에서 사용되는 기술이다. 
- MAC(Media Access Control) 주소는 각각의 기기를 구분하기 위한 용도로 만들어졌고 기기가 만들어질 때 제조사에서 고유한 MAC 주소를 부여한다. 
- MAC 주소는 48bit, 6바이트로 구성되어 있다. AB-CD-EF-12-34-56 또는 AB:CD:EF:12:34:56으로 표기한다.
- MAC 주소는 겹치지 않게 하기 위해 미국의 전기전자공학자협회(IEEE)에서 MAC 주소의 맨 앞 24비트에 제조사 코드를 부여한다. 그러면 제조사들은 MAC 주소의 앞 24bit를 전기전자공학자협회로부터 부여받은 값을 사용하고 나머지 24비트를 기기 고유의 값으로 사용한다. 
- ipconfig  /all 이라는 명령어를 통해 MAC 주소를 알 수 있다. 또한 https://uic.io.ko.mac 이라는 웹사이트에서 MAC 주소를 조회해 제조사를 확인할 수 있다.
- MAC 주소는 이렇게 유무선 랜카드 뿐만 아니라 공유기(라우터), 스위치에 할당되어서 기기를 구분하는 역할을 한다.



## 5. 리피터



<img src="{{site.url}}/images/2023-09-27-os-physical-layer-and-data_link-layer/273532_600.jpg" alt="network-physical-and-datalink-5" width="400px" />



아날로그 신호는 거리가 멀어지면 왜곡, 감쇄가 발생한다. 왜곡, 감쇄가 발생한다는 것은 원 데이터가 손상된다는 뜻이다. 따라서 일정 거리 이상이 되면 통신할 수 없게 된다. 리피터는 케이블 사이에서 왜곡, 감쇄가 발생해 데이터가 손상되기 전에 원래 신호를 복원해 다시 전달하는 역할을 하는 물리 계층의 장비니다.

## 6. 허브



<img src="{{site.url}}/images/2023-09-27-os-physical-layer-and-data_link-layer/990C003A5EB6A8D12F.jpeg" alt="network-physical-and-datalink-6" width="400px" />



허브는 여러 컴퓨터를 추가 랜카드 없이 연결해주는 장치이며 여러 대의 컴퓨터를 연결하는 중심이 되는 장치이다.  컴퓨터는 서로 통신하기 위해 직접 케이블로 연결하는 것이 아니라 중간에 있는 허브에 케이블로 연결한다. 허브는 데이터를 전송할 때 신호도 증폭해서 리피터의 역할까지 하기 때문에 멀티포트 리피터라고도 부른다. MAC 주소에 상관없이 단순히 물리신호를 브로드캐스팅함으로써 물리계층에 해당하는 장비이다. 



허브로 연결된 영역은 충돌이 발생할 수 있는데 그러면 원래 신호가 왜곡되어 통신할 수 없는 상황이 될 수도 있다. 충돌이 발생하지 않도록 한 순간에는 허브에 연결된 하나의 컴퓨터만 데이터를 전송할 수 있는데 이를 CSMA/CD라고 한다. 허브는 이렇게 구조상의 이유로 충돌이 발생할 수 있고 허브로 연결되어 CSMA/CD로 반이중 통신으로 된 영역을 콜리전 도메인(Collision Domain)이라고 부른다. 한 번에 하나의 컴퓨터만 데이터를 전송할 수 있으므로 많은 컴퓨터가 연결되면 속도가 느려진다는 한계가 있어 요즘은 거의 사용되지 않는 장치이다.



## 7. 이더넷과 이더넷 헤더

물리 계층에서는 데이터를 단순히 물리적인 신호로 바꿔서 전달만 하는 역할을 했다면 데이터링크 계층에서는 기기에 MAC 주소라는 것을 부여해 원하는 기기에만 데이터를 전달할 수 있도록 한다. 현재 데이터링크 계층에서 가장 많이 사용하고 있는 프로토콜은 이더넷이다. 이더넷은 구조가 간단하고 장비가 저렴해 데이터링크 계층을 대표하는 프로토콜로 근거리 통신에 필요한 모든 규격이 정리되어 있다(데이터링크 계층의 프로토콜로 토큰링, FDDI 등이 있지만 이더넷이 주류). 이더넷은 MAC 주소를 이용해 주변기기를 구분하고 실제 전송은 물리 계층(전기, 빛, 전파)를 이용한다.



초기 이더넷은 동축 케이블을 사용하는 버스형 네트워크였다. 이런 구조는 송수신을 동시에 하면 충돌이 발생했기 때문에 CSMA/CD라는 프로토콜이 만들어졌고 반이중 통신으로 안전하게 통신할 수 있었다. 하지만 반이중 통신은 속도에 한계가 있었고 이를 해결하기 위해 UTP 케이블과 광 케이블이 등장했다. 케이블 제조업체들은 케이블을 만들 때 [IEEE 802.3](https://ieeexplore.ieee.org/browse/standards/get-program/page/series?id=68)을 참조해서 상세스펙에 맞춰서 만들게 되고 이것을 표준을 지킨 케이블이라고 말한다. 덕분에 우리는 제조업체에 신경쓰지 않고 케이블만 구매해 통신을 할 수 있는 것이다.



<img src="{{site.url}}/images/2023-09-27-os-physical-layer-and-data_link-layer/ethernet.png" alt="network-physical-and-datalink-7" width="700px" />



데이터링크 계층은 네트워크 계층에서 전달되는 패킷에 데이터링크 계층의 약속을 나타내는 필드를 덧붙인다. 이더넷 헤더는 IEEE 802.3에 정의되어 있는데 데이터는 왼쪽부터 오른쪽으로 전송된다. 이더넷 프레임의 크기는 데이터필드에 따라서 64byte ~ 1518byte가 된다.

- Preamble, SDF : 데이터를 수신하는 랜카드에게 이더넷 프레임이라는 것을 알리는 역할을 한다. 총 8byte, 64bit로 구성되어 있다.
- Destination Address : 목적지의 MAC 주소를 저장하는 필드이다.
- Source Address : 출발지의 MAC 주소를 저장하는 필드이다.
- Length / Type : 2byte로 구성되어 있는데 값이 1500 이하라면 데이터의 크기를, 1500 이상이라면 상위 계층의 프로토콜 타입을 나타내는 필드이다.
- Data : 상위 계층에서 전달된 데이터로 네트워크 계층에서 만들어진 패킷을 저장하는 필드이다.
- Pad : Data 필드의 크기가 너무 작을 때 값을 채워 넣는 용도의 필드이다.
- FCS : 현재 프레임이 손상됐는지 체크하기 위한 필드이다. 손상되었다면 프레임을 폐기한다.



## 8. CSMA/CD

CSMA/CD는 Carrier Sense Multi Access / Collision Detection의 약자로 Carrier는 반도체 내에서 전류의 흐름을 발생시킨다. 즉 Carrier Sense라는 말은 전선의 Carrier를 감지하는 것이다. 동축 케이블 내 Carrier가 많다면 전류가 흐르고 있다고 판단하는 것이다. Multi Access라는 말은 두 대 이상의 컴퓨터가 동축 케이블에 접근하는 것을 말한다. 마지막으로 Collisoin Detection은 두 대 이상의 컴퓨터가 접근해 충돌이 발생하는 것을 탐지하는 것을 말한다. 즉 정리하면 CSMA/CD는 케이블에 전류가 흐르고 있는지, 동시에 접근해 충돌이 발생하면 충돌을 감지해 시차를 두어 전송해 반이중통신을 하는 프로토콜이다.



충돌을 감지한 컴퓨터는 데이터 전송을 중지하고 연결된 모든 컴퓨터에게 충돌이 났다는 Jamming 신호를 전송한다. 데이터를 전송하고자 하는 컴퓨터는 랜덤한 시간 동안 기다렸다가 재전송한다. 만약 기다렸다가 재전송할 때도 충돌이 발생하면 또다시 랜덤 시간 동안 기다렸다가 재전송하는데 충돌이 15번 반복한다면 더 이상 전송하지 않고 전송을 포기한다. 이는 하나의 컴퓨터가 고장이나 악의적인 목적으로 계속 데이터를 보낼 때 모든 컴퓨터가 통신이 불가능해지는 것을 막기 위한 것이다.



UTP 케이블은 전송용, 수신용이 따로 있어 전이중통신이 가능한데 왜 CSMA/CD를 이용해서 반이중 통신을 했을까? 1990년에 IEEE 802.3i의 10BASE-T라는 표준으로 UTP 케이블이 등장했다. 이때도 송수신선이 따로 있었지만 송신할 때는 수신을 하지 못해 논리적으로는 반이중통신만 가능했다. 그러나 2대 이상의 컴퓨터가 연결되어 있는 경우 동시에 데이터가 전송될 때 UTP 케이블의 수신선에 충돌이 발생할 수 있다. 충돌을 방지하기 위해서는 나중에 보낼 데이터를 기억할 메모리와 처리 장치가 필요하는데 허브는 가지고 있지 않다. 이후에 브릿지와 스위치가 만들어지면서 이 문제가 해결되었고 1997년 IEEE 802.3x에서 전이중통신의 표준이 발표되었다. 비로소 UTP 케이블의 송수신선을 제대로 이용할 수 있게 된 것이다. 현재는 전이중통신을 사용하기 때문에 CSMA/CD는 거의 사용되지 않는다.



## 9. 브리지

<img src="{{site.url}}/images/2023-09-27-os-physical-layer-and-data_link-layer/img.jpg" alt="network-physical-and-datalink-8" width="400px" />

동시에 데이터를 전송하면 허브에서 충돌이 발생할 수 있다. 여기서 사람들은 PC1과 PC2에서 동시에 데이터가 들어왔을 때 PC1의 데이터는 바로 보내고 PC2의 데이터는 메모리에 보관했다가 PC1 데이터가 전송되고 나면 보내면 되겠다는 생각을 하게 된다. 기존 허브는 들어온 데이터를 브로드캐스팅하는 기능만 있었지만 이제 메모리와 이를 처리하는 프로세서가 필요해졌다. 이런 아이디어로 허브에 메모리와 CPU를 추가한 장치가 브리지이다. 메모리와 처리장치가 있으므로 데이터를 저장할 수 있고 저장된 데이터를 처리장치로 처리까지 할 수 있다. 



브리지는 데이터의 목적지 MAC 주소를 보고 연결된 해당 장치에만 데이터를 전달한다. 허브에 연결된 컴퓨터들은 같은 콜리전 도메인이지만 브리지는 포트마다 콜리전 도메인이 존재한다. 즉, 콜리전 도메인을 나누는 역할을 한다. 브리지는 허브의 기능에서 MAC 주소를 이용해 해당 포트로만 전송할 수 있기 때문에 콜리전 도메인이 나눠져 충돌이 발생하지 않아 CSMA/CD를 쓰지 않아도 돼서 전이중통신이 가능해진다.



## 10. 스위치

<img src="{{site.url}}/images/2023-09-27-os-physical-layer-and-data_link-layer/1450349494_1204486.jpg" alt="network-physical-and-datalink-9" width="400px" />

스위치는 브리지와 기능적인 차이는 없고 성능이 더 좋아진 장치이다. 스위치는 스위칭 허브라고도 불리는데 브리지, 스위치, 스위칭 허브는 모두 같은 말이다. 스위치(브리지)는 MAC 주소를 이용하므로 데이터링크 계층에 속한 장치이다. 메모리와 처리장치가 있으므로 MAC 주소를 기억해 원하는 기기에만 데이터를 전송할 수 있다.

Learning으로 MAC 주소에 해당하는 포트를 매핑하고 MAC 주소 테이블에 찾는 주소가 없다면 Flooding으로 브로드캐스팅한다. 만약 찾는 주소가 있다면 Forwarding으로 원하는 기기에만 전송하고 데이터를 전달하지 않아도 될 때는 Filtering을 한다. 또한 Aging이라는 작업을 통해 메모리가 부족하지 않도록 일정 시간이 지나면 메모리에서 정보를 자동으로 제거한다. 보통 5분이 지나면 자동 삭제된다. 그러나 Aging Timer가 다 되기 전에 MAC 주소를 참조한다면 해당 MAC 주소의 Aging Timer는 초기화되어 시간이 연장된다.

브리지와 스위치는 기능은 같지만 성능 차이가 있다. 스위치는 브리지보다 더 많은 포트를 제공한다. 그리고 스위치는 ASIC이라는 하드웨어로 스위칭 작업을 수행하므로 소프트웨어적으로 처리하는 브리지보다 속도가 더 빠르다. 또한 스위치는 오류를 체크하는 기능이 있지만 브리지는 없다. 마지막으로 스위치는 버퍼를 가지고 있지만 브리지는 없는 경우도 있다. 



## 11. 스패닝 트리 프로토콜

스위치가 순환구조로 연결되면 브로드캐스팅 메시지를 서로 순환하며 무한대로 전송하는 문제가 발생한다. 이를 해결하기 위해서는 순환구조가 발생하지 않도록 만들어줘야 한다. 하지만 사람 손으로는 일일이 하기에는 번거로워서 이 작업을 스위치가 대신 해주는데 이때 스패팅 트리 프로토콜을 이용한다. 스패닝 트리란 트리 자료구조에서 모든 노드를 사이클 없이 연결하는 트리를 말한다. 



<img src="{{site.url}}/images/2023-09-27-os-physical-layer-and-data_link-layer/spanning_tree.png" alt="network-physical-and-datalink-10" width="700px" />



스패닝 트리 프로토콜은 여러 스패닝 트리를 구하고 여기서 가장 빨리 도달할 수 있는 스패닝 트리, 즉 최소 스패닝 트리로 데이터를 전송해 무한루프에 빠지지 않게 하는 프로토콜이다. 즉 스패닝 트리 프로토콜은 최소 스패닝 트리를 구해 무한정으로 데이터가 순환하지 않도록 만드는 프로토콜이다.



## 12. 스위치의 한계

허브로 연결된 영역은 충돌이 발생할 수 있으므로 콜리전 도메인이라고 부르며 CSMA/CD로 반이중통신을 한다. 또한 허브는 연결된 모든 PC에게 브로드캐스트 하므로 트래픽이 많이 발생한다는 단점이 있다. 스위치는 메모리와 CPU가 있으므로 MAC 주소를 이용해 원하는 기기에만 데이터를 전송할 수 있어서 콜리전 도메인을 나눌 수 있게 되었다.

그러나 스위치는 허브와 마찬가지로 브로드캐스트를 하는 경우가 있다. 사용자가 브로드캐스트 요청을 하거나 스위칭할 때 MAC 주소 테이블에 정보가 없다면 Flooding을 하는 경우이다. 브로드캐스트를 하게 되면 연결된 모든 PC로 데이터를 전송하게 되는데 이 영역을 브로드캐스트 도메인이라고 부른다. 브로드캐스트 도메인에 수십만대의 PC가 연결되었다면 브로드캐스트 시 엄청난 트래픽이 발생한다. 따라서 브로드캐스트 도메인을 나눌 필요가 생겼고 브로드캐스트 도메인을 나누기 위해 네트워크 계층이 등장하였다.



참고

- [그림으로 쉽게 배우는 운영체제](https://www.inflearn.com/course/%EB%B9%84%EC%A0%84%EA%B3%B5%EC%9E%90-%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9C/dashboard)

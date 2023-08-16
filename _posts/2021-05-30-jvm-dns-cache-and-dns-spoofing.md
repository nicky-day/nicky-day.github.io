---
layout: single
title: "JVM DNS Cache (DNS Spoofing)"
categories: Java
tag: [Security, DNS Spoofing]
author_profile: false
sidebar:
    nav: "counts"
search: true
use_math: false
---

### 1. JVM DNS Cache

JVM은 각 호스트의 DNS 레코드에 지정된 TTL(Time-to-Live) 값을 사용하는 대신 DNS 조회 정보를 정보를 JVM에 영구적으로 캐시한다. 특정 기간이 지나면 캐시는 리프레시 되어야 하지만 JVM에서는 1.6 버전 이전에서는 TTL값을 을 쓰지 않고 Forever 정책으로 영원히 가지고 있고 이후 버전에 대해서는 30초 DNS Caching이 들어간다. 그 이유는 DNS Spoofing을 막기 위해서이다. 그래서 서버의 IP 주소가 바뀌는 등 DNS 정보가 바뀌더라도 JVM이 재시작 되지 않으면 똑같은 DNS 정보로 보내게 되고 서버와 JVM의 연결이 끊길 수도 있는 문제가 발생한다.

또한, SecurityManager가 존재할 경우에는 보안상의 이유로 JVM이 구동 중에는 Cache가 만료되지 않고, SecurityManager가 존재하지 않을 경우에는 30초간 Cache하게 된다.

### 2. Cache 정책 변경

#### 1. Java 설정 파일에서 내용 변경(전체 변경)

$JAVA_HOME/jre/lib/security/java.security 파일에서 ```networkaddress.cache.ttl```이라는 키를 수정해 주면 된다. 변경을 원하는 경우 주석을 풀고 원하는 시간을 입력해주면 된다. 기준 시간은 sec이다.

```
networkaddress.cache.ttl = 0
```
#### 2. SecurityManager를 통한 설정(특정 어플리케이션만 수정)

Application initialize 과정에 아래와 같이 설정한다.

```java
java.security.Security.setProperty("networkaddress.cache.ttl", "60");
```

### 3. DNS Spoofing

DNS Spoofing이란 공격자가 사용자와 DNS 서버간의 정보 교환을 중간에서 가로채서 사용자가 일정한 서버를 이용할 때 정보를 속여 공격자가 원하는 서버로 접속하게 하는 공격이다. 

![DNS_Spoofing-1]({{site.url}}/images/2021-05-30-jvm-dns-cache-and-dns-spoofing/DNS_Spoofing.png)

참고
- [https://tech.yangs.kr/4](https://tech.yangs.kr/4)
- [https://developers.google.com/recaptcha/old/docs/java?hl=ko](https://developers.google.com/recaptcha/old/docs/java?hl=ko)
- [https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=kiminkyu11&logNo=40166445592](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=kiminkyu11&logNo=40166445592)
- [https://www.keycdn.com/support/dns-spoofing](https://www.keycdn.com/support/dns-spoofing)
- [https://luran.me/319](https://luran.me/319)
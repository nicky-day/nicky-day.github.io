---
layout: single
title: "구글 Chrome SameSite 이슈"
categories: Web
tag: [Cookie, CSRF, Security]
author_profile: false
sidebar:
    nav: "counts"
search: true
use_math: false
---

### 1. 구글 Chrome SameSite 이슈

> 2020년 2월 4일 릴리즈된 구글 크롬(Google Chrome) 80버전부터 새로운 쿠키 정책이 적용되어 Cookie의 SameSite 속성의 기본 값이 "None"에서 "Lax"로 변경되었다.

SameSite가 None으로 설정될 경우 모든 도메인에서 쿠키를 전송하고 사용할 수 있지만 사용자가 ```사이트간 요청 위조(CSRF : Cross-Site Request Forgery)``` 및 의도하지 않은 정보 유출에 취약해질 가능성이 있다. 이러한 취약점을 방지하기 위해 지금까지는 별도의 SameSite 속성을 명시하지 않고 쿠키를 생성했을 때 SameSite=None으로 설정한 것과 동일하게 동작했지만, Chrome80 버전 이후에는 SameSite=Lax로 명시한 것과 동일하게 동작한다는 것이다.

### 2. SameSite 속성

Cookie의 SameSite 속성은 서로 다른 도메인 간의 쿠키 전송에 대한 보안을 설정한다.

>
> A cookie with **"SameSite=Strict"** will only be sent with a same-site request. 
> A cookie with **"SameSite=Lax"** will be sent with a same-site request, or a cross-site top-level navigation with a "safe" HTTP method. 
> A cookie with **"SameSite=None"** will be sent with both same-site and cross-site requests.

- "None"은 동일 사이트와 크로스 사이트 모두에 쿠키 전송이 가능하다. Chrome80버전부터 SameSite를 None으로 설정할 경우 쿠키에 암호화된 HTTPS 연결이 필요함을 나타내는 Secure 속성을 태깅해야 한다.(SameSite=None; Secure) 즉, URL은 HTTPS로 처리되어야 한다.
- "Strict"는 서로 다른 도메인에서는 아예 전송이 불가능해지기 때문에 CSRF를 100% 방지할 수 있으나 사용자 편의성을 해친다.
- "Lax"는 Strict 설정에 일부 예외를 두어 ```HTTP get method / a href / link href```와 같은 특수한 경우에만 쿠키를 전송한다.

### 3. Lax에서 쿠키를 전송하는 경우

- a href 링크
```html
<a href=""></a>
```
- prerender 링크
```html
<link rel="prerender" href=".."/>
```
- HTTP GET 메소드
```html
<form method="GET" action="...">
```

Lax에서 쿠키를 전송하지 않는 경우

- HTTP POST 메소드
```html
<form method="POST" action="...">
```
- iframe
```html
<iframe src="..."></iframe>
```
- Ajax
```html
$.get("...")
```
- Image
```html
<img src="...">
```

### 4. SameSite 설정하기

SameSite 속성을 변경하는 방법은 쿠키 생성하는 시점부터 설정해주거나 필터 등을 이용하여 기존 쿠키에 속성을 추가하는 방법이 있고 Apache나 Nginx같은 HTTP/Proxy 서버를 사용 중이라면 서버 설정을 통해 일괄로 변경하는 방법도 있다.

#### 1. JavaScript

```javascript
document.cookie = "safeCookie1=foo; SameSite=Lax";
document.cookie = "safeCookie2=foo";
document.cookie = "crossCookie=bar; SameSite=None; Secure"
```

#### 2. Java

Java에서 많이 사용하는 javax.servlet.http.Cookie Class에서는 SameSite 관련 API를 지원하지 않기 때문에 SameSite 속성을 추가하려면 HttpServletResponse의 response 객체에 Set-Cookie 헤더(Header)를 직접 추가해야 된다.

```java
response.setHeader("Set-Cookie", "Test1=TestCookieValue1; Secure; SameSite=None");
response.setHeader("Set-Cookie", "Test2=TestCookieValue2; Secure; SameSite=None");
response.setHeader("Set-Cookie", "Test3=TestCookieValue3; Secure; SameSite=None");
```

주의해야할점은 setHeader를 통해 Set-Cookie 헤더를 처음 추가한 뒤에 setHeader를 다시 사용할 경우 동일 이름의 헤더가 오버라이드 되기 때문에 첫 추가 이후에는 addHeader를 통해 추가해줘야 한다.

만약 Spring Core 5.0 이상을 사용한다면 org.springframework.http.ResponseCookie 클래스의 API를 사용하면 조금 더 쉽게 처리 가능하다.

```java
Response cookie = Response.from("sameSiteCookie", "sameSiteCookieValue")
    .domain("")
    .sameSite("None")
    .secure(true)
    .path("/")
    .build();
response.addHeader("Set-Cookie". cookie.toString());
```

#### 3. Filter or Interceptor

필터나 인터셉터 등을 이용하여 response를 가로채 아래 로직을 통해 생성된 쿠키의 헤더에 Secure; SameSite=None 속성을 추가하여 일괄적으로 변경하는 것도 가능하다.

```java
public class CookieAttributeFilter implements Filter{
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException){
    
    	HttpServletResponse httpServletResponse = (HttpServletResponse) response;
        chain.doFilter(request, response);
        log.info("CookieAttributeFilter");
        addSameSite(HttpServletResponse, "None");
        
    }
    
    @Override
    public void init(FilterConfig filterConfig) throws ServletException{
    }
    
    @Override
    public void destroy(){
    }
    
    private void addSameSite(HttpServletResponse response, String sameSite){    
    	Collection<String> headers = response.getHeaders(httpHeaders.SET_COOKIE);
        boolean firstHeader = true;
        for(String header : headers){
            if(firstHeader){
            	response.setHeader(httpHeader.SET_COOKIE, String.format("%s; Secure; %s", header, "SameSite=" + sameSite));
                firstHeader = false;
                continue;
            }
            response.addHeader(HttpHeaders.SET_COOKIE, String.format("%s; Secure; %s", header, "SameSite=" + sameSite));
        }
    }
}               
```

#### 4. Tomcat 설정

Tomcat에서 지원하는 Cookie Processor Component를 이용하여 일괄로 쿠키에 대한 속성을 추가할 수 있다.

- context.xml
```xml
<context>
  ...
  <CookieProcessor sameSiteCookies="none"/>
</context>
```

#### 5. WEB Server 설정 (Http Proxy Server)

Apache 또는 Nginx와 같은 Http 웹서버나 Proxy 서버를 사용 중이라면 서버 설정을 통해 유저가 받는 모든 쿠키 속성을 한 번에 변경할 수 있다.

- Apache Configuration (httpd.conf)

```
Header always edit Set-Cookie (.*) "$1; Secure; SameSite=None;"
```

- Nginx Configuration (nginx.conf)

```
location / { 
    # your usual config ... 
    # hack, set all cookies to secure, httponly and samesite (strict or lax) 
    proxy_cookie_path / "/; secure; SameSite=None"; }

```

### 5. CSRF(Cross-Site Request Forgery) : 사이트 간 요청 위조

```사이트간 요청 위조(또는 크로스 사이트 요청 위조 Cross-Site request forgery CSRF, XSRF)```는 웹 사이트 취약점 공격의 하나로, 사용자가 자신의 의지와는 무관하게 공격자가 의도한 행위(수정, 삭제, 등록 등)를 특정 웹사이트에 요청하게 하는 공격을 말한다.

사이트간 요청 위조는 특정 웹 사이트가 사용자의 웹 브라우저를 신용하는 상태를 노린 것이다. 일단 사용자가 웹사이트에 로그인한 상태에서 사이트간 요청 위조 공격 코드가 삽입된 페이지를 열면, 공격 대상이 되는 웹사이트는 위조된 공격 명령이 믿을 수 있는 사용자로부터 발송된 것으로 판단하게 되어 공격에 노출된다.

#### 공격 과정

1. 이용자는 웹사이트에 로그인하여 정상적인 쿠키를 발급받는다.
2. 공격자는 다음과 같은 링크를 이메일이나 게시판 등의 경로를 통해 이용자에게 전달한다.

	http://www.geocitites.com/attacker
   
3. 공격용 HTML 페이지는 다음과 같은 이미지태그를 가진다.

```html
<img src= "https://travel.service.com/travel_update?.src=Korea&.dst=Hell">
```

해당 링크는 클릭시 정상적인 경우 출발지와 도착지를 등록하기 위한 링크이지만 위의 경우 도착지를 변조한다.

4. 이용자가 공격용 페이지를 열면, 브라우저는 이미지 파일을 받아오기 위해 공격용 URL을 연다.

5. 이용자의 승인이나 인지 없이 출발지와 도착지가 등록됨으로써 공격이 완료된다. 해당 서비스 페이지는 등록 과정에 대해 단순히 쿠키를 통한 본인확인밖에 하지 않으므로 공격자가 정상적인 이용자의 수정이 가능하게 된다.


참고
- https://ifuwanna.tistory.com/223
- https://sevendollars.tistory.com/178
- https://ko.wikipedia.org/wiki/%EC%82%AC%EC%9D%B4%ED%8A%B8_%EA%B0%84_%EC%9A%94%EC%B2%AD_%EC%9C%84%EC%A1%B0

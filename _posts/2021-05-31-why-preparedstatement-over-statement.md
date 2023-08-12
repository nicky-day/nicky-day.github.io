---
layout: single
title: "Statement보다 Preparedstatement을 사용해야 하는 이유(성능, 보안 측면)"
categories: Database
author_profile: false
sidebar:
    nav: "counts"
search: true
use_math: false
---

### 1. 성능 측면

![PreparedStatement_Over_Statement_1]({{site.url}}/images/2021-05-31-why-preparedstatement-over-statement/images_jsj3282_post_8587157c-7cf4-47a5-acce-c8b9fab630b5_wiRZS.png)

SQL 서버 엔진이 쿼리를 수행할 때 마다 다음의 과정을 거친다.

1. 구문 분석 및 정규화 단계 : 쿼리 문법이 제대로 작성되었는지 확인하고 해당 테이블과 칼럼이 데이터베이스에 존재하는지 확인한다.
2. 컴파일 단계 : 쿼리를 컴파일한다.
3. 쿼리 최적화 계획 : 쿼리를 실행할 수 있는 방법의 수와 쿼리를 실행하는 각 방법의 비용을 알아내 최적의 계획을 선택한다.
4. 캐시 : 쿼리 최적화 계획에서 선택된 계획은 캐시에 저장되므로 동일한 쿼리가 들어올때마다 1, 2, 3단계를 다시 실행하지 않고 다음에 동일한 쿼리가 들어오면 Cache 찾아 실행한다.
5. 실행 단계 : 쿼리가 실행되고 데이터가 ResultSet 객체로 사용자에게 반환된다.

Statement는 쿼리를 실행할 때마다 이 1~5단계를 수행해야 한다. PreparedStatement는 어떻게 실행되는가?

PreparedStatement는 완전한 SQL 쿼리가 아니고 SQL 쿼리의 틀을 미리 생성해 놓고 물음표를 대체할 값을 나중에 지정한다. 따라서 PreparedStatement가 처음 실행될 때 위의 1~3단계를 수행 후 사전 컴파일 되어 캐시에 저장된다. 이후에 ```Placeholder Replacement``` 라는 추가 단계가 있으며 런타임시에 사용자가 입력한 데이터로 set메서드를 사용해 ?를 대체한다. 

![PreparedStatement_Over_Statement_2]({{site.url}}/images/2021-05-31-why-preparedstatement-over-statement/images_jsj3282_post_3744ceb3-6cd8-4646-9566-15de6144b2d2_kWnd1.png)

?가 사용자가 입력한 데이터로 바뀐 후에는 최종 쿼리가 다시 구문 분석하거나 컴파일하지 않는다. 

### 2. 보안 측면

PreparedStatement는 SQL Injection 공격을 방지하기 때문에 보안 측면에서도 좋다. 

**SQL Injection**

프로그램의 보안 취약점을 이용해, 악의적인 SQL문을 실행시켜 데이터베이스를 비정상적으로 조작하는 해킹방법이다.
동작원리는 다음과 같다. 시스템의 관리자(admin) 계정의 ID만 알 뿐, 비밀번호를 모르는 상황에서 어떻게 로그인할 수 있을까? PHP에서 로그인 쿼리가 다음과 같다고 생각해보자.

```php
$username = $_POST["username"];
$password = $_POST["Password"];
$mysqli->query("SELECT * FROM users WHERE username='{$username}' AND password='{$password}'");
```
아이디에 admin을 입력하고 비밀번호에 or 1=1 --를 입력하면 로그인이 성공한다. 1=1은 항상 참이므로 공격자는 비밀번호를 입력하지 않고 로그인할 수 있는 것이다.

```php
SELECT * FROM users WHERE username='admin' and password='password' OR 1=1 --'
```

PreparedStatement에서는 위의 쿼리 실행 단계에서 알 수 있듯이, 쿼리가 컴파일 되어 캐시된 이후에 Placeholder Replacement단계에서 사용자의 데이터로 대체되기 때문에 이미 컴파일된 최종 쿼리는 다시 컴파일 과정을 거치지 않는다. 따라서 사용자의 데이터는 항상 간단한 문자열이여야 하며 쿼리의 원래 논리를 수정할 수 없다. 따라서 PreparedStatement를 사용한 쿼리는 SQL 주입 공격에 대한 영향을 받지 않는다.

참고
- [https://jaredablon-31568.medium.com/how-to-prevent-sql-injection-vulnerabilities-how-prepared-statements-work-f492c369614f](https://jaredablon-31568.medium.com/how-to-prevent-sql-injection-vulnerabilities-how-prepared-statements-work-f492c369614f)
- [https://stackoverflow.com/questions/1582161/how-does-a-preparedstatement-avoid-or-prevent-sql-injection](https://stackoverflow.com/questions/1582161/how-does-a-preparedstatement-avoid-or-prevent-sql-injection)
- [https://oper6210.tistory.com/157](https://oper6210.tistory.com/157)
- [https://medium.com/pocs/sql-injection%EC%9D%B4%EB%9E%80-3b57f2415ef4](https://medium.com/pocs/sql-injection%EC%9D%B4%EB%9E%80-3b57f2415ef4)
- [https://ko.wikipedia.org/wiki/SQL_%EC%82%BD%EC%9E%85](https://ko.wikipedia.org/wiki/SQL_%EC%82%BD%EC%9E%85)

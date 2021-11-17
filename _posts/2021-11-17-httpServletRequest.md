---
layout: single
title:  "서블릿"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

# HttpServletRequest, Http 요청 데이터(1)



## HttpServletRequest의 역할

- HTTP 요청 메시지 파싱 결과를 HttpServletRequest 객체에 담아서 제공



※ **임시 저장소 기능**

```java
request.setAttribute(name, value); // 저장
request.getAttribute(name); // 조회
request.getSession(create: true); //세션 관리 기능
```



---



## HTTP 요청 데이터

####  주로 다음 3가지 방법을 사용



####   1. GET - 쿼리 파라미터

- ###### /url?username=hello&age=20

- ###### 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달

- ###### EX) 검색, 필터, 페이징



####   **2. POST - HTML Form**

- content-type: application/x-www-form-urlencoded
- 메시지 바디에 쿼리 파라미터 형식으로 전달 username=hello&age=20
- EX) 회원가입, 상품 주문 HTML Form 사용



####   **3. HTTP message body에 데이터를 직접 담아서 요청 **

- ###### HTTP API에서 주로 사용, JSON, XML, TEXT

---



## GET 쿼리 파라미터

#### 	- 전달 데이터 : username=hello, age=20

​	

#### 	⁕ 쿼리 파라미터는 URL에 다음과 같이 ?를 시작으로 보낼 수 있다. 추가 파라미터는 &로 구분.

(http://localhost:8080/request-param?username=hello&age=20)



- #### 쿼리 파라미터 조회 메서드

```java
String username = request.getParameter("username"); //단일 파라미터 조회
Enumeration<String> parameterNames = request.getParameterNames(); //파라미터 이름들 모두 조회
Map<String, String[]> parameterMap = request.getParameterMap(); //파라미터를 Map으로 조회
String[] usernames = request.getParameterValues("username"); //복수 파라미터 조회
```



####    ※ 복수 파라미터에서 단일 파라미터 조회

#### 	[username=hello&username=kim]

####     파라미터 이름은 하나인데, 값이 중복이면?

####     <mark style='background-color: #fff5b1'>request.getParameterValues()</mark>를 사용!	



## HTTP 요청 데이터 - POST HTML Form

- #### 특징

  - content-type: **application/x-www-form-urlencoded**

  - 메시지 바디에 쿼리 파라미터 형식으로 데이터를 전달

    (username=hello&age=20)

  

  ![](/assets/images/2021-11-17-html (copy)/1.JPG)




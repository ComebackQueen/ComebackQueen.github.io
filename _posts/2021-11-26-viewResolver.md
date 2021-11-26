---
layout: single
title:  "스프링 MVC - 구조 이해"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

#  뷰 리졸버



## 뷰 리졸버- InternalResourceViewResolver



- **application.properties**

```java
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
```

 ※ 스프링 부트는 InternalResourceViewResolver라는 뷰 리졸버를 자동 등록,

이때 application.properties에 등록한 prefix, suffix 설정 정보를 사용해서 등록.

---



## 뷰 리졸버 동작 방식

- **스프링 부트가 자동 등록하는 뷰 리졸버**

```java
1 = BeanNameViewResolver : 빈 이름으로 뷰를 찾아서 반환한다. (예: 엑셀 파일 생성
기능에 사용)
2 = InternalResourceViewResolver : JSP를 처리할 수 있는 뷰를 반환한다.
```



**1. 핸들러 어댑터 호출**

- 핸들러 어댑터를 통해 new-form이라는 논리 뷰 이름을 획득



**2. ViewResolver 호출**

- new-form이라는 뷰 이름으로 viewResolver를 순서대로 호출
- BeanNameViewResolver는 new-form이라는 이름의 스프링 빈으로 등록된 뷰를 찾는다(만약 없으면)
- InternalResourceViewResolver가 호출된다



**3. InternalResourceViewResolver**

- 이 뷰 리졸버는 InternalResourceView를 반환한다



**4. 뷰 - InternalResourceView **

- InternalResourceView는 JSP처럼 포워드를 호출해서 처리할 수 있는 경우에 사용



**5. view.render()**

- view.render()가 호출되고 InternalResourceView는 forward()를 사용해서 JSP를 실행


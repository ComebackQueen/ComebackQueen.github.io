---
layout: single
title:  "빈 스코프"
toc: true
toc_sticky: true
categories:
  - Spring_Basic 
---

#  빈 스코프란?



## 스프링은 다음과 같은 다양한 스코프를 지원



- **싱글톤** : 기본 스코프, 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프
- **프로토타입** : 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입까지만 관여하고 더는 관리하지 않는 매우 짧은 범위의 스코프
- **웹 관련 스코프**
  - **request** : 웹 요청이 들어오고 나갈때 까지 유지되는 스코프
  - **session** : 웹 세션이 생성되고 종료될 때 까지 유지되는 스코프
  - **application** : 웹의 서블릿 컨텍스트와 같은 범위로 유지되는 스코프

- **컴포넌트 스캔 자동 등록**

```java
@Scope("prototype")
@Component
public class HelloBean {}
```



- **수동 등록**

```java
@Scope("prototype")
@Bean
PrototypeBean HelloBean() {
	 return new HelloBean();
}
```

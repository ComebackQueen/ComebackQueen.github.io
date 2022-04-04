---
layout: single
title:  "빈 스코프"
toc: true
toc_sticky: true
categories:
  - Spring_Basic
---

# 스코프와 프록시



## 스코프와 프록시



```java
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class MyLogger {
}
```

- `proxyMode = ScopedProxyMode.TARGET_CLASS`를 추가
  - 적용 대상이 인터페이스가 아닌 클래스면 `TARGET_CLASS`를 선택
  - 적용 대상이 인터페이스면 `INTERFACES`를 선택
- 이렇게 하면 MyLogger의 가짜 프록시 클래스를 만들어두고 HTTP request와 상관 없이 가짜 프록시 클래스를 다른 빈에 미리 주입해 둘 수 있다

---



## 웹 스코프와 프록시 동작 원리

- 주입된 myLogger 확인

```
myLogger = class hello.core.common.MyLogger$$EnhancerBySpringCGLIB$$b68b726d
```



### CGLIB라는 라이브러리로 내 클래스를 상속 받은 가짜 프록시 객체를 만들어서 주입

- `@Scope`의 `proxyMode = ScopedProxyMode.TARGET_CLASS`를 설정하면 스프링 컨테이너는 CGLIB라는 바이트코드를 조작하는 라이브러리를 사용해서, MyLogger를 상속받은 가짜 프록시 객체를 생성
- 결과를 확인해보면 MyLogger 클래스가 아니라 `MyLogger$$EnhancerBySpringCGLIB`라는 클래스로 만들어진 객체가 대신 등록된 것을 확인할 수 있다
- 그리고 스프링 컨테이너에 "myLogger"라는 이름으로 진짜 대신에 이 가짜 프록시 객체를 등록
- `ac.getBean("myLogger", MyLogger.class)`로 조회해도 프록시 객체가 조회되는 것을 확인할 수 있음
- 그래서 의존관계 주입도 이 가짜 프록시 객체가 주입됨  

![image-20220404164013602](/assets/images/2022-04-04-scopeProxy/image-20220404164013602.png)  



### 가짜 프록시 객체는 요청이 오면 그때 내부에서 진짜 빈을 요청하는 위임 로직이 들어있다

- 가짜 프록시 객체는 내부에 진짜 myLogger를 찾는 방법을 알고 있다
- 클라이언트가 `myLogger.logic()`을 호출하면 사실은 가짜 프록시 객체의 메서드를 호출한 것
- 가짜 프록시 객체는 request 스코프의 진짜 `myLogger.logic()`를 호출
- 가짜 프록시 객체는 원본 클래스를 상속 받아서 만들어졌기 때문에 이 객체를 사용하는 클라이언트 입장에서는 사쇨 원본인지 아닌지도 모르게, 동일하게 사용할 수 있다(다형성)  



### 동작 정리

- CGLIB라는 라이브러리로 내 클래스를 상속 받은 가짜 프록시 객체를 만들어서 주입
- 이 가짜 프록시 객체는 실제 요청이 오면 그때 내부에서 실제 빈을 요청하는 위임 로직이 들어있음
- 가짜 프록시 객체는 실제 request scope와는 관계 X(그냥 가짜, 내부에 단순한 위임 로직만 존재, 싱글톤 처럼 동작)  



### 특징 정리

- 프록시 객체 덕분에 클라이언트는 마치 싱글톤 빈을 사용하듯이 편리하게 request scope를 사용할 수 있다
- 사실 Provider를 사용하든, 프록시를 사용하든 핵심 아이디어는 진짜 객체 조회를 꼭 필요한 시점까지 지연처리
- 단지 애노테이션 설정 변경만으로 원본 객체를 프록시 객체로 대체 O(다형성과 DI컨테이너가 가진 큰 장점)
- 꼭 웹 스코프가 아니어도 프록시는 사용할 수 있다  



### 주의점

- 마치 싱글톤을 사용하는 것 같지만 다르게 동작하기에 주의해서 사용
- 이런 특별한 scope는 꼭 필요한 곳에만 최소화해서 사용(무분별하게 사용하면 유지보수 어려움)

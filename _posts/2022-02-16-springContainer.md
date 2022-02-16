---
layout: single
title:  "스프링 컨테이너와 스프링 빈"
toc: true
toc_sticky: true
categories:
  - Spring_Basic
---

#  스프링 컨테이너 생성



## 들어가기 전

```java
// 스프링 컨테이너 생성

ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
```

- `ApplicationContext`를 스프링 컨테이너라 한다(인터페이스)
- 스프링 컨테이너는 XML 기반, 애노테이션 기반의 자바 설정 클래스로 만들 수 있다  



---



## 스프링 컨테이너의 생성 과정



**1. 스프링 컨테이너 생성**

![](/assets/images/2022-02-16-springContainer/1.JPG)

- `new AnnotationConfigApplicationContext(AppConfig.class)`
- 스프링 컨테이너를 생성할 때는 구성 정보를 지정해주어야 함(`AppConfig.class`)  



**2. 스프링 빈 등록**

![](/assets/images/2022-02-16-springContainer/2.JPG)

- 스프링 컨테이너는 파라미터로 넘어온 설정 클래스 정보를 사용해서 스프링 빈을 등록

※ 주의 : 빈 이름은 항상 다른 이름을 부여(다른 빈이 무시 or 기존 빈을 덮어버림 or 설정에 따른 오류 발생)  



**3. 스프링 빈 의존관계 설정 - 준비**

![](/assets/images/2022-02-16-springContainer/3.JPG)  



**4. 스프링 빈 의존관계 설정 - 완료**

![](/assets/images/2022-02-16-springContainer/4.JPG)

- 스프링 컨테이너는 설정 정보를 참고해서 의존관계 주입(DI)

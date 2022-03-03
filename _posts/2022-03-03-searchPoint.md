---
layout: single
title:  "컴포넌트 스캔"
toc: true
toc_sticky: true
categories:
  - Spring_Basic
---

#  탐색 위치와 기본 스캔 대상



## 탐색할 패키지의 시작 위치 지정

```java
@ComponentScan(
 	basePackages = "hello.core",
}
```

- `basePackages` : 탐색할 패키지의 시작 위치를 지정
- `basePackageClasses` : 지정한 클래스의 패키지를 탐색 시작 위치로 지정
- 만약 지정하지 않으면 `@ComponentScan`이 붙은 설정 정보 클래스의 패키지가 시작 위치가 된다



**※권장하는 방법**

설정 정보 클래스의 위치를 프로젝트 최상단에 두는 것(`com.hello`)

---



## 컴포넌트 스캔 기본 대상



컴포넌트 스캔은 `@Component`뿐만 아니라 다음과 같은 내용도 추가로 대상에 포함

- `@Component` : 컴포넌트 스캔에서 사용
- `@Controller` : 스프링 MVC 컨트롤러에서 사용
- `@Service` : 스프링 비즈니스 로직에서 사용
- `@Repository` : 스프링 데이터 접근 계층에서 사용
- `@Configuration` : 스프링 설정 정보에서 사용



**※ 컴포넌트 스캔 외의 부가 기능**

-  `@Controller` : 스프링 MVC 컨트롤러로 인식
- `@Repository` : 스프링 데이터 접근 계층으로 인식, 데이터 계층의 예외를 스프링 예외로 변환
- `@Configuration` : 스프링 설정 정보로 인식, 스프링 빈이 싱글톤을 유지하도록 추가 처리
- `@Service` : 특별한 처리는 X, 개발자들이 핵심 비즈니스 계층을 인식하는데 도움이 된다

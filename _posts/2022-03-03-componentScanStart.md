---
layout: single
title:  "컴포넌트 스캔"
toc: true
toc_sticky: true
categories:
  - Spring_Basic
---

#  컴포넌트 스캔과 의존관계 자동 주입 시작하기



## 들어가기전

- 스프링 빈을 등록할 때는 자바 코드의 @Bean이나 XML의 `<bean>`을 통해서 설정 정보에 직접 등록할 빈을 나열
- 그렇지만 등록해야 할 빈의 갯수가 수십, 수백개가 되면 반복의 문제가 발생(정보커짐, 누락의 문제)
- 이를 위해, 컴포넌트 스캔이라는 기능을 제공한다
- 또한, 의존관계도 자동으로 주입하는 `@Autowired`라는 기능도 제공 

---

## 컴포넌트 스캔 및 의존관계 주입 과정



- **AutoAppConfig.java**

```java
@Configuration
@ComponentScan(
 excludeFilters = @Filter(type = FilterType.ANNOTATION, classes =
Configuration.class))
public class AutoAppConfig {
 
}
```

- 컴포넌트 스캔을 사용하려면 먼저 `@ComponentScan`을 설정 정보에 붙여주면 된다
- 기존의 AppConfig와 달리 @Bean으로 등록한 클래스가 하나도 없다



이제 각 클래스가 컴포넌트 스캔의 대상이 되도록 `@Component` 애노테이션을 붙여준다(일부 클래스 생략)



- **MemoryMemberRepository @Component 추가**

```java
@Component
public class MemoryMemberRepository implements MemberRepository{}
```



- **MemberServiceImpl @Component, @Autowired 추가**

```java
@Component
public class MemberServiceImpl implements MemberService {
    
 	private final MemberRepository memberRepository;
 
    @Autowired
 	public MemberServiceImpl(MemberRepository memberRepository) {
 		this.memberRepository = memberRepository;
 	}
}
```

- `@Autowired`를 사용하면 생성자에서 여러 의존관계도 한번에 주입받을 수 있다

---



## 컴포넌트 스캔과 자동 의존관계 주입의 동작 원리



### **1. @ComponentScan**

![](/assets/images/2022-03-03-componentScanStart/1.JPG)

- `@ComponentScan`은 `@Component`가 붙은 모든 클래스를 스프링 빈으로 등록

- 이때 스프링 빈의 기본 이름은 클래스명을 사용하되 맨 앞글자만 소문자를 사용

  - **빈 이름 기본 :** MemberServiceImpl 클래스 -> memberServiceImpl
  - **빈 이름 직접 지정 : **`@Component("memberService2")`

  

### **2. @Autowired 의존관계 자동 주입**

![](/assets/images/2022-03-03-componentScanStart/2.JPG)

- 생성자에 `@Autowired`를 지정하면, 스프링 컨테이너가 자동으로 해당 스프링 빈을 찾아서 주입
- 이때 기본 조회 전략은 타입이 같은 빈을 찾아서 주입
  - `getBean(MemberRepository.class)`와 동일하다고 이해하면 된다



![](/assets/images/2022-03-03-componentScanStart/3.JPG)

- 생성자에 파라미터가 많아도 다 찾아서 자동으로 주입


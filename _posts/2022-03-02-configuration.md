---
layout: single
title:  "싱글톤 컨테이너"
toc: true
toc_sticky: true
categories:
  - Spring_Basic
---

#  @Configuration과 바아트코드 조작의 마법



## 들어가기전

- 스프링 컨테이너는 싱글톤 레지스트리, 스프링 빈이 싱글톤이 되도록 보장해주어야함
- 스프링이 자바 코드까지 어떻게 하기는 어려움 -> 스프링은 클래스의 **바이트코드를 조작하는 라이브러리**를 사용
- 모든 비밀은 `@Configuration`을 적용한 `AppConfig`에 있다

---



```java
@Test
void configurationDeep() {
 ApplicationContext ac = new
AnnotationConfigApplicationContext(AppConfig.class);
 //AppConfig도 스프링 빈으로 등록된다.
 AppConfig bean = ac.getBean(AppConfig.class);
 
 System.out.println("bean = " + bean.getClass());
 //출력: bean = class hello.core.AppConfig$$EnhancerBySpringCGLIB$$bd479d70
}
```

- 스프링이 **CGLIB**라는 바이트코드 조작 라이브러리를 사용하여 AppConfig 클래스를 상속받은 임의의 다른 클래스를 만들고, 그 다른 클래스를 스프링 빈으로 등록한 것

![](/assets/images/2022-03-02-configuration/1.JPG)  



**AppConfig@CGLIB 예상 코드**

```java
@Bean
public MemberRepository memberRepository() {
 
 if (memoryMemberRepository가 이미 스프링 컨테이너에 등록되어 있으면?) {
 return 스프링 컨테이너에서 찾아서 반환;
 } else { //스프링 컨테이너에 없으면
 기존 로직을 호출해서 MemoryMemberRepository를 생성하고 스프링 컨테이너에 등록
 return 반환
 }
}
```

- @Bean이 붙은 메서드마다 빈이 존재하면 존재하는 빈을 반환, 빈이 없으면 생성해서 스프링 빈으로 등록 및 반환하는 코드가 동적으로 만들어진다
- 덕분에 싱글톤이 보장되는 것

---



## `@Configuration`을 적용하지 않고, `@Bean`만 적용하면?



- @Configuration 삭제 후 실행?

```java
// MemberRepository가 총 3번 호출
call AppConfig.memberService
call AppConfig.memberRepository
call AppConfig.orderService
call AppConfig.memberRepository
call AppConfig.memberRepository
    
// 인스턴스 비교 테스트결과
memberService -> memberRepository = 
hello.core.member.MemoryMemberRepository@6239aba6
orderService -> memberRepository = 
hello.core.member.MemoryMemberRepository@3e6104fc
memberRepository = hello.core.member.MemoryMemberRepository@12359a82
```



### **정리**

- @Bean만 사용해도 스프링 빈으로 등록되지만, 싱글톤을 보장하지 않는다
  - `memberRepository()`처럼 의존관계 주입이 필요해서 메서드를 직접 호출할 때 싱글톤을 보장하지 않음
- 스프링 설정 정보는 항상 `@Configuration`을 사용해야 한다!

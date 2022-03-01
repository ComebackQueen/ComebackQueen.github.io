---
layout: single
title:  "싱글톤 컨테이너"
toc: true
toc_sticky: true
categories:
  - Spring_Basic
---

#  싱글톤 컨테이너



## 싱글톤 컨테이너

- 컨테이너는 객체를 하나만 생성해서 관리한다
- 스프링 컨테이너는 싱글톤 컨테이너 역할을 한다(싱글톤 객체를 생성 및 관리하는 기능 = 싱글톤 레지스트리)
- 스프링 컨테이너의 이런 기능 덕분에 싱글톤 패턴의 모든 단점을 해결



**스프링 컨테이너를 사용하는 테스트 코드**

```java
@Test
@DisplayName("스프링 컨테이너와 싱글톤")
void springContainer() {
    
 	ApplicationContext ac = new
	AnnotationConfigApplicationContext(AppConfig.class);
 
    //1. 조회: 호출할 때 마다 같은 객체를 반환
 	MemberService memberService1 = ac.getBean("memberService",
	MemberService.class);
    
 	//2. 조회: 호출할 때 마다 같은 객체를 반환
 	MemberService memberService2 = ac.getBean("memberService",
	MemberService.class);
 
    //참조값이 같은 것을 확인
	System.out.println("memberService1 = " + memberService1);
 	System.out.println("memberService2 = " + memberService2);
 
    //memberService1 == memberService2
 	assertThat(memberService1).isSameAs(memberService2);
}
```



---



## 싱글톤 컨테이너 적용 후



![](/assets/images/2022-03-01-singletonCon/1.JPG)

- 스프링 컨테이너 덕분에 고객의 요청이 올 때 마다 객체를 생성하는 것이 아니라, 이미 만들어진 객체를 공유해서 효율적으로 재사용할 수 있다


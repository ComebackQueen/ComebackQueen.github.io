---
layout: single
title:  "싱글톤 컨테이너"
toc: true
toc_sticky: true
categories:
  - Spring_Basic
---

#  웹 애플리케이션과 싱글톤



## 들어가기 전  

- 스프링은 기업용 온라인 서비스 기술을 지원하기 위해 탄생한것
- 웹 애플리케이션은 보통 여러 고객이 동시에 요청을 함

![](/assets/images/2022-02-17-webappSingleton/1.JPG)



**스프링 없는 순수한 DI 컨테이너 테스트**

```java
public class SingletonTest {
 @Test
 @DisplayName("스프링 없는 순수한 DI 컨테이너")
 void pureContainer() {
     AppConfig appConfig = new AppConfig();
     //1. 조회: 호출할 때 마다 객체를 생성
     MemberService memberService1 = appConfig.memberService();
     //2. 조회: 호출할 때 마다 객체를 생성
     MemberService memberService2 = appConfig.memberService();
     //참조값이 다른 것을 확인
     System.out.println("memberService1 = " + memberService1);
     System.out.println("memberService2 = " + memberService2);
     //memberService1 != memberService2
     assertThat(memberService1).isNotSameAs(memberService2);
 	}
}
```

- 고객 트래픽이 초당 100이 나오면 초당 100개 객체가 생성되고 소멸된다! -> 메모리 낭비가 심하다
- 해결방안 -> 해당 객체가 딱 1개만 생성, 공유하도록 설계 (**싱글톤 패턴**)

---



## 싱글톤 패턴



- 클래스의 **인스턴스가 딱 1개만 생성**되는 것을 보장하는 디자인 패턴

- 객체 인스턴스를 2개 이상 생성하지 못하도록 막아야함(private 생성자 사용 -> 외부에서 new 키워드를 사용 못하도록 막아야함)  

  

**싱글톤 패턴을 적용한 예제 코드**

```java
public class SingletonService {
	
    // 1. static 영역에 객체를 딱 1개만 생성
    private static final SingletonService instance = new SingletonService();
	
    
   // 2. public으로 열어서 객체 인스턴스가 필요하면 이 static 메서드를 통해서만 조회하도록 허용
    public static SingletonService getInstance() {
        return instance;
    }
	
    // 3. 생성자를 private으로 선언하여 외부에서 new 키워드를 사용한 객체 생성을 못하게 막음
    private SingletonService() {
    }

    public void logic() {
        System.out.println("싱글톤 객체 로직 호출");
    }
}
```



**싱글톤 패턴 문제점**

- 구현하는 코드 자체가 많이 들어감
- 의존관계상 클라이언트가 구체 클래스에 의존 -> DIP를 위반
- 클라이언트가 구체 클래스에 의존해서 OCP 원칙을 위반할 가능성이 높음
- 테스트가 어렵다
- 내부 속성을 변경, 초기화 하기 어려움
- private 생성자로 자식 클래스를 만들기 어려움
- 유연성이 떨어짐
- 안티패턴...

---
layout: single
title:  "빈 스코프"
toc: true
toc_sticky: true
categories:
  - Spring_Basic 
---

#  프로토타입 스코프



## 싱글톤 빈 요청 VS 프로토타입 빈 요청



프로토타입 스코프를 스프링 컨테이너에 조회하면 스프링 컨테이너는 **항상 새로운 인스턴스를 생성**해서 반환



- **싱글톤 빈 요청**

![](/assets/images/2022-03-22-prototypeScope/1.JPG)

(1) 싱글톤 스코프의 빈을 스프링 컨테이너에 요청

(2) 스프링 컨테이너는 본인이 관리하는 스프링 빈을 반환

(3) 이후에 스프링 컨테이너에 같은 요청이 와도 객체 인스턴스의 스프링 빈을 반환  



- **프로토타입 빈 요청1**

![](/assets/images/2022-03-22-prototypeScope/2.JPG)

(1) 프로토타입 스코프의 빈을 스프링 컨테이너에 요청

(2) 스프링 컨테이너는 이 시점에 프로토타입 빈을 생성하고, 필요한 의존관계를 주입  



- **프로토타입 빈 요청2**

![](/assets/images/2022-03-22-prototypeScope/3.JPG)

(3) 스프링 컨테이너는 생성한 프로토타입 빈을 클라이언트에 반환

(4) 이후에 스프링 컨테이너에 같은 요청이 오면 항상 새로운 프로토타입 빈을 생성해서 반환



**정리**

**핵심은 스프링 컨테이너는 프로토타입 빈을 생성하고, 의존관계 주입, 초기화까지만 처리한다는 것!!**



---

## 코드 테스트

- **싱글톤 스코프 빈 테스트** 

```java
public class SingletonTest {

    @Test
    void singletonBeanFind(){
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SingletonBean.class);

        SingletonBean singletonBean1 = ac.getBean(SingletonBean.class);
        SingletonBean singletonBean2 = ac.getBean(SingletonBean.class);
        System.out.println("singletonBean1 = " + singletonBean1);
        System.out.println("singletonBean2 = " + singletonBean2);
        assertThat(singletonBean1).isSameAs(singletonBean2);

        ac.close();
    }

    @Scope("singleton")
    static class SingletonBean {
        @PostConstruct
        public void init() {
            System.out.println("SingletonBean.init");
        }

        @PreDestroy
        public void destroy() {
            System.out.println("SingletonBean.destroy");
        }
    }
}
```



- **실행결과**

```java
SingletonBean.init
singletonBean1 = hello.core.scope.SingletonTest$SingletonBean@c94fd30
singletonBean2 = hello.core.scope.SingletonTest$SingletonBean@c94fd30
org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing SingletonBean.destroy
```

- 빈 초기화 메서드를 실행, 같은 인스턴스의 빈을 조회, 종료 메서드까지 정상 호출  



- **프로토타입 스코프 빈 테스트**

```java
public class PrototypeTest {

    @Test
    void prototypeBeanFind() {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);
        System.out.println("find prototypeBean1");
        PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);
        System.out.println("find prototypeBean2");
        PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);
        System.out.println("prototypeBean1 = " + prototypeBean1);
        System.out.println("prototypeBean2 = " + prototypeBean2);
        assertThat(prototypeBean1).isNotSameAs(prototypeBean2);

        ac.close();
    }

    @Scope("prototype")
    static class PrototypeBean {
        @PostConstruct
        public void init() {
            System.out.println("PrototypeBean.init");
        }

        @PreDestroy
        public void destroy() {
            System.out.println("PrototypeBean.destroy");
        }
    }
}
```



- **실행결과**

```java
find prototypeBean1
PrototypeBean.init
find prototypeBean2
PrototypeBean.init
prototypeBean1 = hello.core.scope.PrototypeTest$PrototypeBean@76f4b65
prototypeBean2 = hello.core.scope.PrototypeTest$PrototypeBean@c94fd30
org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing   
```

- 싱글톤 빈은 스프링 컨테이너 생성 시점에 초기화 메서드 실행, 프로토입입 스코프의 빈은 스프링 컨테이너에서 빈을 조회할 때 생성, 초기화 메서드도 실행
- 프로토타입 빈을 2번 조회 -> 완전히 다른 스프링 빈이 생성, 초기화도 2번 실행
- 프로토타입 빈은 스프링 컨테이너가 생성과 의존관계 주입 그리고 초기화 까지만 관여(스프링 컨테이너가 종료될 때 `@PreDestroy`같은 종료 메서드가 전혀 실행 X)



**프로토타입 빈의 특징 정리**

- 스프링 컨테이너에 요청할 때 마다 새로 생성
- 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입 그리고 초기화까지만 관여
- 종료 메서드 호출 X
- 프로토타입 빈은 프로토타입 빈을 조회한 클라이언트가 관리해야 한다(종료 메서드에 대한 호출도 클라이언트가 직접 해야함)

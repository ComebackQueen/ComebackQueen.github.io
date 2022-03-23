---
layout: single
title:  "빈 스코프"
toc: true
toc_sticky: true
categories:
  - Spring_Basic 
---

#  프로토타입 스코프 - 싱글톤 빈과 함께 사용시 문제점



## 프로토타입 빈 직접 요청



싱글톤 빈과 함께 사용할 때는 의도한 대로 잘 동작하지 않으므로 주의해야 한다!



- **스프링 컨테이너에 프로토타입 빈 직접 요청1**

![](/assets/images/2022-03-23-prototypeScopeProblem/1.JPG)

(1) 클라이언트A는 스프링 컨테이너에 프로토타입 빈을 요청

(2) 스프링 컨테이너는 프로토타입 빈을 새로 생성해서 반환(**x01**)한다.(해당 빈의 count 필드 값은 0)

(3) 클라이언트는 조회한 프로토타입 빈에 `addCount()`를 호출하면서 count 필드를 +1한다

(4) 결과적으로 프로토타입 빈(**x01**)의 count는 1이 된다



- **스프링 컨테이너에 프로토타입 빈 직접 요청2**

![](/assets/images/2022-03-23-prototypeScopeProblem/2.JPG)

(1) 클라이언트B는 스프링 컨테이너에 프로토타입 빈을 요청

(2) 스프링 컨테이너는 프로토타입 빈을 새로 생성해서 반환(**x02**)한다.(해당 빈의 count 필드 값은 0)

(3) 클라이언트는 조회한 프로토타입 빈에 `addCount()`를 호출하면서 count 필드를 +1한다

(4) 결과적으로 프로토타입 빈(**x02**)의 count는 1이 된다



- **코드 확인**

```java
public class SingletonWithPrototypeTest1 {
    @Test
    void prototypeFind() {
        AnnotationConfigApplicationContext ac = new
                AnnotationConfigApplicationContext(PrototypeBean.class);
        PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);
        prototypeBean1.addCount();
        assertThat(prototypeBean1.getCount()).isEqualTo(1);
        PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);
        prototypeBean2.addCount();
        assertThat(prototypeBean2.getCount()).isEqualTo(1);
    }
    @Scope("prototype")
    static class PrototypeBean {
        private int count = 0;
        public void addCount() {
            count++;
        }
        public int getCount() {
            return count;
        }
        @PostConstruct
        public void init() {
            System.out.println("PrototypeBean.init " + this);
        }
        @PreDestroy
        public void destroy() {
            System.out.println("PrototypeBean.destroy");
        }
    }
}
```



---



## 싱글톤 빈에서 프로토타입 빈 사용

이번에는 `clientBean`이라는 싱글톤 빈이 의존관계 주입을 통해서 프로토타입 빈을 주입받아서 사용하는 예를 보자  



- **싱글톤에서 프로토타입 빈 사용1**

![](/assets/images/2022-03-23-prototypeScopeProblem/3.JPG)

- `clientBean`은 싱글톤, 보통 스프링 컨테이너 생성 시점에 함께 생성되고, 의존관계 주입도 발생

(1) `clientBean`은 의존관계 자동 주입을 사용. 주입 시점에 스프링 컨테이너에 프로토타입 빈을 요청

(2) 스프링 컨테이너는 프로토타입 빈을 생성해서 `clientBean`에 반환. 프로토타입 빈의 count 필드값은 0.

- 이제 `clientBean`은 프로토타입 빈을 내부 필드에 보관(정확히는 참조값을 보관)



- **싱글톤에서 프로토타입 빈 사용2**

![](/assets/images/2022-03-23-prototypeScopeProblem/4.JPG)

- 클라이언트 A는 `clientBean`을 스프링 컨테이너에 요청해서 받는다. 싱글톤이므로 항상 같은 `clientBean`이 반환

(3) 클라이언트 A는 `clientBean.logic()`을 호출

(4) `clientBean`은 prototypeBean의 `addCount()`를 호출해서 프로토타입 빈의 count를 증가(count값이 1이 된다)



- **싱글톤에서 프로토타입 빈 사용3**

![](/assets/images/2022-03-23-prototypeScopeProblem/5.JPG)

- 클라이언트 B는 `clientBean`을 스프링 컨테이너에 요청해서 받는다. 싱글톤이므로 항상 같은 `clientBean`이 반환
-  clientBean이 내부에 가지고 있는 프로토타입 빈은 이미 과거에 주입이 끝난 빈이다. **주입 시점에 스프링 컨테이너에 요청해서 프로토타입 빈이 새로 생성이 된 것**이지, **사용 할 때마다 새로 생성되는 것이 아니다!**

(3) 클라이언트 B

- **테스트 코드** 

```java
public class SingletonWithPrototypeTest1 {

    @Test
    void prototypeFind() {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);
        PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);
        prototypeBean1.addCount();
        assertThat(prototypeBean1.getCount()).isEqualTo(1);

        PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);
        prototypeBean2.addCount();
        assertThat(prototypeBean2.getCount()).isEqualTo(1);
    }

    @Test
    void singletonClientUsePrototype() {
        AnnotationConfigApplicationContext ac =
                new AnnotationConfigApplicationContext(ClientBean.class, PrototypeBean.class);

        ClientBean clientBean1 = ac.getBean(ClientBean.class);
        int count1 = clientBean1.logic();
        assertThat(count1).isEqualTo(1);

        ClientBean clientBean2 = ac.getBean(ClientBean.class);
        int count2 = clientBean2.logic();
        assertThat(count2).isEqualTo(1);
    }

    @Scope("singleton")
    static class ClientBean {

        @Autowired
        private Provider<PrototypeBean> prototypeBeanProvider;

        public int logic() {
            PrototypeBean prototypeBean = prototypeBeanProvider.get();
            prototypeBean.addCount();
            return prototypeBean.getCount();
        }
    }

    @Scope("prototype")
    static class PrototypeBean {
        private int count = 0;

        public void addCount() {
            count++;
        }

        public int getCount() {
            return count;
        }

        @PostConstruct
        public void init() {
            System.out.println("PrototypeBean.init " + this);
        }

        @PreDestroy
        public void destroy() {
            System.out.println("PrototypeBean.destroy");
        }
    }
}
```



- **정리**
  - 싱글톤 빈은 생성 시점에만 의존관계 주입을 받기 때문에, 프로토타입 빈이 새로 생성되기는 하지만, **싱글톤 빈과 함께 계속 유지되는 것이 문제!!**


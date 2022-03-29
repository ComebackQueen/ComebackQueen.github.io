---
layout: single
title:  "빈 스코프"
toc: true
toc_sticky: true
categories:
  - Spring_Basic 
---

#  프로토타입 스코프 - 싱글톤 빈과 함께 사용시 Provider로 문제 해결



## 스프링 컨테이너에 요청



싱글톤 빈과 프로토타입 빈을 함께 사용할 때, 어떻게 하면 사용할 때 마다 항상 새로운 프로토타입 빈을 생성할 수 있을까?

※ 가장 간단한 방법은 싱글톤 빈이 프로토타입을 사용할 때 마다 스프링 컨테이너에 새로 요청하는 것



```java
public class PrototypeProviderTest {
    @Test
    void providerTest() {
        AnnotationConfigApplicationContext ac = new
                AnnotationConfigApplicationContext(ClientBean.class, PrototypeBean.class);
        ClientBean clientBean1 = ac.getBean(ClientBean.class);
        int count1 = clientBean1.logic();
        assertThat(count1).isEqualTo(1);
        ClientBean clientBean2 = ac.getBean(ClientBean.class);
        int count2 = clientBean2.logic();
        assertThat(count2).isEqualTo(1);
    }
    static class ClientBean {
        @Autowired
        private ApplicationContext ac;
        public int logic() {
            PrototypeBean prototypeBean = ac.getBean(PrototypeBean.class);
            prototypeBean.addCount();
            int count = prototypeBean.getCount();
            return count;
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



**핵심 코드**

```java
@Autowired
    private ApplicationContext ac;
    public int logic() {
        PrototypeBean prototypeBean = ac.getBean(PrototypeBean.class);
        prototypeBean.addCount();
        int count = prototypeBean.getCount();
        return count;
    }
```

- 실행해보면 `ac.getBean()`을 통해서 항상 새로운 프로토타입 빈이 생성
- 직접 필요한 의존관계를 찾는 것을 Dependency Lookup(DL) 의존관계 조회라 한다
- 그런데 이렇게 스프링의 애플리케이션 컨텍스트 전체를 주입받게 되면, 스프링 컨테이너에 종속적인 코드가 되고, 단위 테스트도 어려워진다
- 지정한 프로토타입 빈을 컨테이너에서 대신 찾아주는 **DL** 정도의 기능만 제공하는 무언가가 있으면 된다



---



## ObjectFactory, ObjectProvider

지정한 빈을 컨테이너에서 대신 찾아주는 DL 서비스를 제공하는 것이 바로 `ObjectProvider`(과거에는 `ObjectFactory`가 있었으나 편의 기능을 추가한게 `ObjectProvider`)

```java
@Autowired
    private ObjectProvider<PrototypeBean> prototypeBeanProvider;
    public int logic() {
        PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
        prototypeBean.addCount();
        int count = prototypeBean.getCount();
        return count;
    }
```

- 실행해보면 `prototypeBeanProvider.getObject()`을 통해서 항상 새로운 프로토타입 빈이 생성
- `ObjectProvider`의 `getObject()`를 호출하면 내부에서는 스프링 컨테이너를 통해 해당 빈을 찾아서 반환(**DL**)
- `ObjectProvider`는 지금 딱 필요한 DL 정도의 기능만 제공



**특징**

- ObjectFactory : 기능 단순, 별도의 라이브러리 필요 X, 스프링에 의존
- ObjectProvider : ObjectFactory 상속, 옵션, 스트림 처리 등 편의 기능이 많고, 별도의 라이브러리 필요 X, 스프링에 의존

---



## JSR-330 Provider



마지막 방법은 `javax.inject.Provider`라는 JSR-330 자바 포준을 사용하는 방법



**javax.inject.Provider 참고용 코드**

```java
package javax.inject;
public interface Provider<T> {
    T get();
}
    //implementation 'javax.inject:javax.inject:1' gradle 추가 필수
    @Autowired
    private Provider<PrototypeBean> provider;
    public int logic() {
        PrototypeBean prototypeBean = provider.get();
        prototypeBean.addCount();
        int count = prototypeBean.getCount();
        return count;
    }
```

- 실행해보면 `provider.get()`을 통해서 항상 새로운 프로토타입 빈이 생성
- `provider`의 `get()`을 호출하면 내부에서는 스프링 컨테이너를 통해 해당 빈을 찾아 반환(DL)



**특징**

- `get()` 메서드 하나로 기능이 매우 단순
- 별도의 라이브러리가 필요
- 자바 표준이므로 스프링이 아닌 다른 컨테이너에서도 사용 O



**정리**

- 프로토 타입 빈을 언제 사용? 매번 사용할 때 마다 의존관계 주입이 완료된 새로운 객체가 필요하면 사용하면 됨(실무에서는 싱글톤 빈으로 대부분의 문제를 해결 가능 -> 프로토타입 빈을 직접적으로 사용하는 일은 드물다)
- `ObjectProvider`, `JSR330 Provider`등은 프로토타입 뿐만 아니라 DL이 필요한 경우는 언제든지 사용 O

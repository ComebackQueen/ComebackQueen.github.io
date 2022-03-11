---
layout: single
title:  "빈 생명주기 콜백"
toc: true
toc_sticky: true
categories:
  - Spring_Basic
---

#  빈 생명주기 콜백 시작



## 들어가기전...



- DB 커넥션 풀처럼 애플리케이션 시작 시점에 필요한 연결을 미리 해두고, 애플리케이션 종료 시점에 연결을 모두 종료하는 작업을 진행하려면, 객체의 초기화와 종료 작업이 필요함
- 스프링을 통해 이러한 초기화 작업과 종료 작업을 어떻게 진행하는지 예제로 확인해보자



---

## 예제 코드, 테스트 하위에 생성



```java
public class NetworkClient {

    private String url;

    public NetworkClient() {
        System.out.println("생성자 호출, url = " + url);
        connect();
        call("초기화 연결 메시지");
    }

    public void setUrl(String url) {
        this.url = url;
    }

    //서비스 시작시 호출
    public void connect() {
        System.out.println("connect : " + url);
    }

    public void call(String message) {
        System.out.println("call : " + url + " message = " + message);
    }

    //서비스 종료시 호출
    public void disconnect() {
        System.out.println("close : " + url);
    }
}
```



- **스프링 환경설정과 실행**

```java
public class BeanLifeCycleTest {

    @Test
    public void lifeCycleTest() {
        ConfigurableApplicationContext ac = new AnnotationConfigApplicationContext(LifeCycleConfig.class);
        NetworkClient client = ac.getBean(NetworkClient.class);
        ac.close();
    }

    @Configuration
    static class LifeCycleConfig {

        @Bean
        public NetworkClient networkClient() {
            NetworkClient networkClient = new NetworkClient();
            networkClient.setUrl("http://hello-spring.dev");
            return networkClient;
        }
    }
}
```



- 실행결과

```java
생성자 호출, url = null
connect: null
call: null message = 초기화 연결 메시지
```



스프링 빈은 간단하게 다음과 같은 라이프사이클을 가진다

- **객체 생성 -> 의존관계 주입**
- 개발자가 의존관계 주입이 모두 완료된 시점을 어떻게 알 수 있을까?
- **스프링은 의존관계 주입이 완료되면 스프링 빈에게 콜백 메서드를 통해서 초기화 시점을 알려주는 다양한 기능을 제공**
- **스프링은 스프링 컨테이너가 종료되기 직전에 소멸 콜백을 준다**



---

## 스프링 빈의 이벤트 라이프사이클

**스프링 컨테이너 생성 -> 스프링 빈 생성 -> 의존관계 주입 -> 초기화 콜백 -> 사용 -> 소멸전 콜백 -> 스프링 종료**



- **초기화 콜백** : 빈이 생성되고, 빈의 의존관계 주입이 완료된 후 호출
- **소멸전 콜백** : 빈이 소멸되기 직전에 호출



### **스프링은 크게 3가지 방법으로 빈 생명주기 콜백을 지원한다**

- 인터페이스(InitializingBean, DisposableBean)
- 설정 정보에 초기화 메서드, 종료 메서드 지정
- @PostConstruct, @PreDestroy 애노테이션 지원

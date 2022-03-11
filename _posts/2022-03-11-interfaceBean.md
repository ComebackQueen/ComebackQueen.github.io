---
layout: single
title:  "빈 생명주기 콜백"
toc: true
toc_sticky: true
categories:
  - Spring_Basic
---

#  인터페이스 InitializingBean, DisposableBean



## 인터페이스 InitializingBean, DisposableBean



```java
public class NetworkClient implements InitializingBean, DisposableBean {

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

    @Override
    public void afterPropertiesSet() throws Exception {
        connect();
        call("초기화 연결 메시지");
    }

    @Override
    public void destroy() throws Exception {
        disconnect();
    }
}   
```

- `InitializingBean`은 `afterPropertiesSet()` 메서드로 초기화를 지원한다
- `DisposableBean`은 `destroy()` 메서드로 소멸을 지원한다



**출력결과**

```java
생성자 호출, url = null
NetworkClient.afterPropertiesSet
connect: http://hello-spring.dev
call: http://hello-spring.dev message = 초기화 연결 메시지
13:24:49.043 [main] DEBUG 
org.springframework.context.annotation.AnnotationConfigApplicationContext - 
Closing NetworkClient.destroy
close + http://hello-spring.dev
```

- 초기화 메서드 주입 완료, 스프링 컨테이너의 종료가 호출되자 소멸 메서드가 호출



---

## 초기화, 소멸 인터페이스 단점



- 이 인터페이스는 스프링 전용 인터페이스(해당 코드가 스프링 전용 인터페이스에 의존)
- 초기화, 소멸 메서드의 이름을 변경 X
- 내가 코드를 고칠 수 없는 외부 라이브러이에 적용 X



※ 인터페이스를 사용하는 초기화, 종료 방법은 현재 거의 사용하지 않음...

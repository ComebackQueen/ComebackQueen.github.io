---
layout: single
title:  "싱글톤 컨테이너"
toc: true
toc_sticky: true
categories:
  - Spring_Basic
---

#  싱글톤 방식의 주의점



## 주의사항

- 싱글톤 방식은 여러 클라이언트가 하나의 같은 객체 인스턴스를 공유하기 때문에 싱글톤 객체는 **상태를 유지(stateful)하게 설계하면 안된다**
- **무상태(stateless)**로 설계해야 한다
  - 특정 클라이언트에 의존적인 필드 X
  - 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 X
  - 가급적 읽기만 가능
  - 필드 대신에 자바에서 공유되지 않는 **지역변수, 파라미터, ThreadLocal** 등을 사용

- 스프링 빈의 필드에 공유 값을 설정하면 정말 큰 장애가 발생할 수 있다!



**상태를 유지할 경우 발생하는 문제점 예시**

```java
public class StatefulService {

    private int price; // 상태를 유지하는 필드

    public void order(String name, int price) {
        System.out.println("name = " + name + " price = " + price);
        this.price = price; // 여기가 문제
    }

    public int getPrice() {
        return price;
    }
}
```



```java
class StatefulServiceTest {

    @Test
    void statefulServiceSingleton() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
        StatefulService statefulService1 = ac.getBean(StatefulService.class);
        StatefulService statefulService2 = ac.getBean(StatefulService.class);

        //ThreadA: A사용자 10000원 주문
       statefulService1.order("userA", 10000);
        //ThreadB: B사용자 20000원 주문
       statefulService1.order("userB", 20000);

        //ThreadA: 사용자A 주문 금액 조회
        int price = statefulService1.getPrice();
        //ThreadA : 사용자A는 10000원을 기대했지만, 기대와 다르게 20000원 출력
        System.out.println("price = " + price);

        Assertions.assertThat(statefulService1.getPrice()).isEqualTo(20000);
    }

    static class TestConfig {

        @Bean
        public StatefulService statefulService() {
            return new StatefulService();
        }
    }
}
```


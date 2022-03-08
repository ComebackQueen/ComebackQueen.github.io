---
layout: single
title:  "컴포넌트 스캔"
toc: true
toc_sticky: true
categories:
  - Spring_Basic
---

#  생성자 주입을 선택해라!



## 들어가기전...



### 최근에는 스프링을 포함한 DI 프레임워크 대부분이 생성자 주입을 권장한다!



---





**불변**

- 대부분의 의존관계는 **애플리케이션 종료 전까지 변하면 안됨**(불변)
- 수정자 주입을 사용하면, setXxx 메서드를 public으로 열어두어야함
- 생성자 주입은 객체를 생성할 때 **딱 1번만 호출됨**(불변하게 설계 가능!!)



**누락**

- 프레임워크 없이 순수한 자바 코드를 단위 테스트 하는 경우에 다음과 같이 수정자 의존관계인 경우

```java
public class OrderServiceImpl implements OrderService {

    private MemberRepository memberRepository;
 	private DiscountPolicy discountPolicy;
 
    @Autowired
 	public void setMemberRepository(MemberRepository memberRepository) {
 		this.memberRepository = memberRepository;
 	}
 
    @Autowired
 	public void setDiscountPolicy(DiscountPolicy discountPolicy) {
 		this.discountPolicy = discountPolicy;
 	}
 	//...
}
```

- **`@Autowired`**가 프레임워크 안에서 동작할 때는 의존관계가 없으면 오류가 발생하지만, 지금은 프레임워크 없이 순수한 자바 코드로만 단위 테스트를 수행하고 있다

  

**테스트 수행하면 실행은 된다**

```java
@Test
void createOrder() {
 	OrderServiceImpl orderService = new OrderServiceImpl();
 	orderService.createOrder(1L, "itemA", 10000);
}
```

- 결과는 NPE(널포)가 발생. memberRepository, discountPolicy 모두 의존관계 주입이 누락되었기 때문
- 생성자 주입을 사용하면 주입 데이터를 누락 했을때 **컴파일 오류**가 발생한다

---



## final 키워드



- 생성자 주입을 사용하면 필드에 `final` 키워드를 사용할 수 있다. 그래서 생성자에서 혹시라도 값이 설정되지 않는 오류를 컴파일 시점에 막아준다!
- **컴파일 오류는 세상에서 가장 빠르고, 좋은 오류이다!**



### **정리**

- 생성자 주입 방식을 선택하는 이유 : 순수한 자바 언어의 특징을 잘 살리는 방법(프레임워크에 의존 X)
- **기본** : 생성자 주입 사용, **필수 값이 아닌 경우** : 수정자 주입 방식을 옵션으로 부여(생성자 주입과 수정자 주입 동시 사용 가능)
- 항상 생성자 주입을 선택!! 옵션이 필요하면 수정자 주입을 선택(필드 주입은 사용하지 않는게 좋다) 

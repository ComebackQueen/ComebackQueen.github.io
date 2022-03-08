---
layout: single
title:  "의존관계 자동 주입"
toc: true
toc_sticky: true
categories:
  - Spring_Basic
---

#  다양한 의존관계 주입 방법



## 들어가기전...(의존관계 주입의 4가지 방법)

- 생성자 주입
- 수정자 주입(setter 주입)
- 필드 주입
- 일반 메서드 주입

---



## 생성자 주입



- 생성자를 통해서 의존 관계를 주입 받는 방법
- 특징
  - 생성자 호출시점에 딱 1번만 호출되는 것이 보장
  - **불변, 필수** 의존관계에 사용

```java
@Component
public class OrderServiceImpl implements OrderService {
 
    private final MemberRepository memberRepository;
 	private final DiscountPolicy discountPolicy;
 
    @Autowired
 	public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy 
discountPolicy) {
 		this.memberRepository = memberRepository;
 		this.discountPolicy = discountPolicy;
 	}
}
```

**⭐생성자가 딱 1개만 있으면 @Autowired를 생략해도 자동 주입 된다**



---



## 수정자 주입(setter 주입)



- setter라 불리는 필드의 값을 변경하는 수정자 메서드를 통해서 의존관계를 주입하는 방법
- 특징
  - **선택, 변경** 가능성이 있는 의존관계에 사용
  - 자바빈 프로퍼티 규약의 수정자 메서드 방식을 사용하는 방법

```java
@Component
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
    
}
```



---



## 필드 주입



- 이름 그대로 필드에 바로 주입하는 방법
- 특징
  - 코드가 간결하지만 외부에서 변경이 불가능해서 테스트 하기 힘들다는 치명적인 단점이 있음
  - DI 프레임워크가 없으면 아무것도 할 수 없다
  - 사용하지 말자!

```java
@Component
public class OrderServiceImpl implements OrderService {
 	
    @Autowired
 	private MemberRepository memberRepository;

    @Autowired
 	private DiscountPolicy discountPolicy;

}

```

---



## 일반 메서드 주입



- 일반 메서드를 통해서 주입 받을 수 있다
- 특징
  - 한번에 여러 필드를 주입 받을 수 있다
  - 일반적으로 잘 사용하지 않는다

```java
@Component
public class OrderServiceImpl implements OrderService {
 
    private MemberRepository memberRepository;
 	private DiscountPolicy discountPolicy;
 
    @Autowired
 	public void init(MemberRepository memberRepository, DiscountPolicy 
	discountPolicy) {
 		this.memberRepository = memberRepository;
 		this.discountPolicy = discountPolicy;
 	}
    
}

```


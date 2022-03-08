---
layout: single
title:  "컴포넌트 스캔"
toc: true
toc_sticky: true
categories:
  - Spring_Basic
---

#  조회 빈이 2개 이상 - 문제



## 조회 빈이 2개 이상 - 문제



### `@Autowired`는 타입(Type)으로 조회

```java
@Autowired
private DiscountPolicy discountPolicy
```

- 타입으로 조회하기 때문에, 마치 다음 코드와 유사하게 동작

  (`ac.getBean(DiscountPolicy.class)`)



`DiscountPolicy`의 하위 타입인 `FixDiscountPolicy`, `RateDiscountPolicy` 둘다 스프링 빈으로 선언

```java
@Component
public class FixDiscountPolicy implements DiscountPolicy {}

@Component
public class RateDiscountPolicy implements DiscountPolicy {}

//의존관계 자동 주입 실행
@Autowired
private DiscountPolicy discountPolicy

```

- `NoUniqueBeanDefinitionException` 오류가 발생
- 하나의 빈을 기대했는데 `fixDiscountPolicy` , `rateDiscountPolicy` 2개가 발견되었다고 알려준다
- 의존 관계 자동 주입에서 해결하는 여러 방법이 있다(다음 파트에~)


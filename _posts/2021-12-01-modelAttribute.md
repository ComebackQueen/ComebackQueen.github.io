---
layout: single
title:  "스프링 MVC - 구조 이해"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

#  HTTP 요청 파라미터 - @ModelAttribute



## ModelAttribute

- 요청 파라미터를 받아서 필요한 객체를 만들고 그 객체에 값을 넣어주어야 함.

```java
@RequestParam String username;
@RequestParam int age;

HelloData data = new HelloData();
data.setUsername(username);
data.setAge(age);
```

※ 스프링은 이 과정을 완전히 자동화해주는 @ModelAttribute 기능을 제공함



- **HelloData**

```java
@Data
public class HelloData {
    private String username;
    private int age;
}
```

- 롬복 @Data
  - @Getter, @Setter, @ToString, @EqualsAndHashCode, @RequiredArgsConstructor를 자동으로 적용

---



- @ModelAttribute 적용 - modelAttributeV1

```java
/**
 * @ModelAttribute 사용
 * 참고: model.addAttribute(helloData) 코드도 함께 자동 적용됨
 */
@ResponseBody
@RequestMapping("/model-attribute-v1")
public String modelAttributeV1(@ModelAttribute HelloData helloData) {
 log.info("username={}, age={}", helloData.getUsername(),
helloData.getAge());
 return "ok";
}
```

- 스프링MVC는 @ModelAttribute가 있으면 다음을 실행
  - HelloData 객체 생성
  - 요청 파라미터의 이름으로 HelloData 객체의 프로퍼티를 찾는다. 그리고 해당 프로퍼티의 setter를 호출해서 파라미터의 값을 입력(바인딩)
  - ex) 파라미터 이름이 username이면 setUsername() 메서드를 찾아서 호출하면서 값을 입력

---



@ModelAttribute 생략 - modelAttributeV2

```java
/**
 * @ModelAttribute 생략 가능
 * String, int 같은 단순 타입 = @RequestParam
 * argument resolver 로 지정해둔 타입 외 = @ModelAttribute
 */
@ResponseBody
@RequestMapping("/model-attribute-v2")
public String modelAttributeV2(HelloData helloData) {
 log.info("username={}, age={}", helloData.getUsername(),
helloData.getAge());
 return "ok";
}
```

※ @ModelAttribute는 생략할 수 있다

그런데 @RequestParam도 생략할 수 있으니 혼란이 발생할 수 있음



스프링은 해당 생략시 다음과 같은 규칙을 적용

- String, int, Integer 같은 단순 타입 = @RequestParam
- 나머지 = @ModelAtrribute(argument resolver로 지정해둔 타입 외)

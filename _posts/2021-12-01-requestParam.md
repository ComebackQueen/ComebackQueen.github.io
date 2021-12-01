---
layout: single
title:  "스프링 MVC - 구조 이해"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

#  HTTP 요청 파라미터 - @RequestParam



## requestParamV2,V3,V4



- **requestParamV2,V3,V4**

```java
	@ResponseBody
    @RequestMapping("/request-param-v2")
    public String requestParamV2(
            @RequestParam("username") String memberName,
            @RequestParam("age") int memberAge
    ) {
        log.info("username={}, age={}",memberName, memberAge);
        return "ok";
    }
	
	/**
 	* @RequestParam 사용
 	* HTTP 파라미터 이름이 변수 이름과 같으면 @RequestParam(name="xx") 생략 가능
 	*/
    @ResponseBody
    @RequestMapping("/request-param-v3")
    public String requestParamV3(
            @RequestParam String username,
            @RequestParam int age
    ) {
        log.info("username={}, age={}",username, age);
        return "ok";
    }
	
	/**
	* @RequestParam 사용
 	* String, int 등의 단순 타입이면 @RequestParam 도 생략 가능
 	*/
    @ResponseBody
    @RequestMapping("/request-param-v4")
    public String requestParamV4(String username, int age) {
        log.info("username={}, age={}",username, age);
        return "ok";
    }
```

- @RequestParam : 파라미터 이름으로 바인딩
- @ResponseBody : View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력

---

## 파라미터 필수 여부

- **requestParamRequired**

```java
/**
 * @RequestParam.required
 * /request-param -> username이 없으므로 예외
 *
 * 주의! - 파라미터 이름만 사용
 * /request-param?username= -> 빈문자로 통과
 *
 * 주의! - 기본형(primtive)에 null 입력
 * /request-param
 * int age -> null을 int에 입력하는 것은 불가능, 따라서 Integer 변경해야 함(또는 다음에 나오는
defaultValue 사용)
 */	

	@ResponseBody
    @RequestMapping("/request-param-required")
    public String requestParamRequired(
            @RequestParam(required = true) String username,
            @RequestParam(required = false) Integer age) {

        log.info("username={}, age={}",username, age);
        return "ok";
    }
```

- @RequestParam.required
  - 파라미터 필수 여부
  - 기본값이 파라미터 필수(true)이다.

---



## 기본 값 적용



- **requestParamDefault**

```java
/**
 * @RequestParam
 * - defaultValue 사용
 *
 * 참고: defaultValue는 빈 문자의 경우에도 적용
 * /request-param?username=
 */

	@ResponseBody
    @RequestMapping("/request-param-default")
    public String requestParamDefault(
            @RequestParam(required = true, defaultValue = "guest") String username,
            @RequestParam(required = false, defaultValue = "-1") int age) {

        log.info("username={}, age={}",username, age);
        return "ok";
    }
```

- 파라미터에 값이 없는 경우 defaultValue를 사용하면 기본 값을 적용할 수 있다

  (이미 기본 값이 있기 때문에 required는 의미가 없다)

---



## 파라미터를 Map으로 조회하기



- **requestParamMap**

```java
/**
 * @RequestParam Map, MultiValueMap
 * Map(key=value)
 * MultiValueMap(key=[value1, value2, ...] ex) (key=userIds, value=[id1, id2])
 */

	@ResponseBody
    @RequestMapping("/request-param-map")
    public String requestParamMap(
            @RequestParam Map<String, Object> paramMap) {

        log.info("username={}, age={}",paramMap.get("username"), paramMap.get("age"));
        return "ok";
    }
```


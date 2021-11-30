---
layout: single
title:  "스프링 MVC - 구조 이해"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

#  요청 매핑



## 매핑 정보



- **MappingController**

```java
@RestController
public class MappingController {

    private Logger log = LoggerFactory.getLogger(getClass());

    @RequestMapping("/hello-basic")
    public String helloBasic() {
        log.info("helloBasic");
        return "ok";
    }
}
```

- @RestController
  - @Controller는 반환 값이 String이면 뷰 이름으로 인식
  - @RestController는 반환 값으로 뷰를 찾는 것이 아니라, <mark style='background-color: #fff5b1'>HTTP 메시지 바디에 바로 입력</mark>
- @RequestMapping("/hello-basic")
  - /hello-basic URL 호출이 오면 이 메서드가 실행되도록 매핑



- HTTP 메서드 매핑

```java
/**
 * method 특정 HTTP 메서드 요청만 허용
 * GET, HEAD, POST, PUT, PATCH, DELETE
 */
@RequestMapping(value = "/mapping-get-v1", method = RequestMethod.GET)
public String mappingGetV1() {
 log.info("mappingGetV1");
 return "ok";
}
```

※ 만약 여기에 POST 요청을 하면 스프링 MVC는 HTTP 405 상태코드를 반환한다



- HTTP 메서드 매핑 축약

```java
/**
     * 편리한 축약 애노테이션 (코드보기)
     * @GetMapping
     * @PostMapping
     * @PutMapping
     * @DeleteMapping
     * @PatchMapping
     */
    @GetMapping(value = "/mapping-get-v2")
    public String mappingGetV2() {
        log.info("mapping-get-v2");
        return "ok";
    }
```

※ HTTP 메서드를 축약한 애노테이션을 사용하는 것이 더 직관적



- PathVariable(경로 변수) 사용

```java
/**
     * PathVariable 사용
     * 변수명이 같으면 생략 가능
     * @PathVariable("userId") String userId -> @PathVariable userId
     * /mapping/userA
     */
    @GetMapping("/mapping/{userId}")
    public String mappingPath(@PathVariable String userId) {
        log.info("mappingPath userId={}", userId);
        return "ok";
    }
```

- @RequestMapping은 URL 경로를 템플릿화 할 수 있는데, @PathVariable을 사용하면 매칭 되는 부분을 편리하게 조회할 수 있다



- PathVariable 사용 - 다중

```java
/**
 * PathVariable 사용 다중
 */
    @GetMapping("/mapping/users/{userId}/orders/{orderId}")
    public String mappingPath(@PathVariable String userId, @PathVariable Long
            orderId) {
        log.info("mappingPath userId={}, orderId={}", userId, orderId);
        return "ok";
    }
```



- 미디어 타입 조건 매핑 - HTTP 요청 Content-Type, consume

```java
/**
     * Content-Type 헤더 기반 추가 매핑 Media Type
     * consumes="application/json"
     * consumes="!application/json"
     * consumes="application/*"
     * consumes="*\/*"
     * MediaType.APPLICATION_JSON_VALUE
     */
    @PostMapping(value = "/mapping-consume", consumes = MediaType.APPLICATION_JSON_VALUE)
    public String mappingConsumes() {
        log.info("mappingConsumes");
        return "ok";
    }
```

※ HTTP 요청의 Content-Type 헤더를 기반으로 미디어 타입으로 매핑

만약 맞지 않으면 HTTP 415 상태코드(Unsupported Media Type)를 반환한다. 



- 미디어 타입 조건 매핑 - HTTP 요청 Accept, produce

```java
/**
     * Accept 헤더 기반 Media Type
     * produces = "text/html"
     * produces = "!text/html"
     * produces = "text/*"
     * produces = "*\/*"
     */
    @PostMapping(value = "/mapping-produce", produces = MediaType.TEXT_HTML_VALUE)
    public String mappingProduces() {
        log.info("mappingProduces");
        return "ok";
    }
```

※ HTTP 요청의 Accept 헤더를 기반으로 미디어 타입으로 매핑한다. 

만약 맞지 않으면 HTTP 406 상태코드(Not Acceptable)을 반환한다

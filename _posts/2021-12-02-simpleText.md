---
layout: single
title:  "스프링 MVC - 구조 이해"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

#  HTTP 요청 파라미터 - 단순 텍스트



## 들어가기 전

- **HTTP message body**에 데이터를 직접 담아서 요청
  - HTTP API에서 주로 사용, JSON, XML, TEXT
  - 데이터 형식은 주로 JSON 사용
  - POST, PUT, PATCH



※ 요청 파라미터와 다르게, HTTP 메시지 바디를 통해 데이터가 직접 넘어오는 경우는 @RequestParam, @ModelAttribute를 사용할 수 없다. (단, HTML Form 형식으로 전달되는 경우는 요청 파라미터로 인정)



- HTTP 메시지 바디의 데이터를 **InputStream**을 사용해서 직접 읽을 수 있다



- **RequestBodyStringController**

```java
@Slf4j
@Controller
public class RequestBodyStringController {

    @PostMapping("/request-body-string-v1")
    public void requestBodyString(HttpServletRequest request, HttpServletResponse response) throws IOException {
        ServletInputStream inputStream = request.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        log.info("messageBody={}", messageBody);

        response.getWriter().write("ok");
    }
}
```



---



- **Input, Output 스트림, Reader - requestBodyStringV2**

```java
/**
 * InputStream(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회
 * OutputStream(Writer): HTTP 응답 메시지의 바디에 직접 결과 출력
 */
@PostMapping("/request-body-string-v2")
    public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {

        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        log.info("messageBody={}", messageBody);

        responseWriter.write("ok");
    }
```

---



- **HttpEntity - requestBodyStringV3**

```java
/**
 * HttpEntity: HTTP header, body 정보를 편라하게 조회
 * - 메시지 바디 정보를 직접 조회(@RequestParam X, @ModelAttribute X)
 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
 *
 * 응답에서도 HttpEntity 사용 가능
 * - 메시지 바디 정보 직접 반환(view 조회X)
 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
 */	

	@PostMapping("/request-body-string-v3")
    public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) throws IOException {

        String messageBody = httpEntity.getBody();
        log.info("messageBody={}", messageBody);

        return new HttpEntity<>("ok");
    }
```

※ HttpEntity를 상속받은 다음 객체들도 같은 기능을 제공

- **RequestEntity**
  - HttpMethod, url 정보가 추가, 요청에서 사용
- **ResponseEntity**
  - HTTP 상태 코드 설정 가능, 응답에서 사용
  - return new ResponseEntity<String>("Hello World", responseHeaders, 
    HttpStatus.CREATED)

---



- **@RequestBody - requestBodyStringV4**

```java
/**
 * @RequestBody
 * - 메시지 바디 정보를 직접 조회(@RequestParam X, @ModelAttribute X)
 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
 *
 * @ResponseBody
 * - 메시지 바디 정보 직접 반환(view 조회X)
 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
 */
@ResponseBody
@PostMapping("/request-body-string-v4")
public String requestBodyStringV4(@RequestBody String messageBody) {
 log.info("messageBody={}", messageBody);
 return "ok";
}
```

- 파라미터에 값이 없는 경우 defaultValue를 사용하면 기본 값을 적용할 수 있다

  (이미 기본 값이 있기 때문에 required는 의미가 없다)



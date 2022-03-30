---
layout: single
title:  "빈 스코프"
toc: true
toc_sticky: true
categories:
  - Spring_Basic
---

#  request 스코프 예제 개발



## 들어가기 전...

- 동시에 여러 HTTP 요청이 오면 정확히 어떤 요청이 남긴 로그인지 구분하기 여려움 -> **request 스코프**

- 로그 ex)

  ```
  [d06b992f...] request scope bean create
  [d06b992f...][http://localhost:8080/log-demo] controller test
  [d06b992f...][http://localhost:8080/log-demo] service id = testId
  [d06b992f...] request scope bean close
  ```

- 기대하는 공통 포멧 : `[UUID][requestURL]{message}`



---



## request 스코프 예제 개발



### MyLogger

```java
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
// 이 빈은 HTTP 요청 당 하나씩 생성되고, HTTP 요청이 끝나는 시점에 소멸
public class MyLogger {

    private String uuid;
    private String requestURL;

    public void setRequestURL(String requestURL) {
        this.requestURL = requestURL;
    }

    public void log(String message) {
        System.out.println("[" + uuid + "]" + "[" + requestURL + "]" + message);
    }
	
    // 초기화 메서드를 사용하여 uuid를 생성해서 저장(이 빈은 HTTP 요청 당 하나씩 생성, uuid를 저장해두면 다른 	// HTTP 요청과 구분할 수 있음)
    @PostConstruct
    public void init() {
        uuid = UUID.randomUUID().toString();
        System.out.println("[" + uuid + "] request scope bean create: " + this);
    }
	
    // 이 빈이 소멸되는 시점에 @PreDestroy를 사용해서 종료 메시지를 남김
    @PreDestroy
    public void close() {
        System.out.println("[" + uuid + "] request scope bean close: " + this);
    }
}
```

- 로그를 출력하기 위한 `MyLogger` 클래스
- `requestURL`은 이 빈이 생성되는 시점에는 알 수 없으므로, 외부에서 setter로 입력 받음



### LogDemoController

```java
@Controller
@RequiredArgsConstructor
public class LogDemoController {

    private final LogDemoService logDemoService;
    private final MyLogger myLogger;

    @RequestMapping("log-demo")
    @ResponseBody
    public String logDemo(HttpServletRequest request) throws InterruptedException {
        String requestURL = request.getRequestURL().toString();
        myLogger.setRequestURL(requestURL);

        myLogger.log("controller test");
        logDemoService.logic("testID");
        return "OK";
    }

}
```

- 로거가 잘 작동하는지 확인하는 테스트용 컨트롤러
- HttpServletRequest를 통해서 요청 URL을 받음
- 이렇게 받은 requestURL 값을 myLogger에 저장
- 컨트롤러에서 controller test라는 로그를 남김



### LogDemoService 추가

```java
@Service
@RequiredArgsConstructor
public class LogDemoService {

    private final MyLogger myLogger;

    public void logic(String id) {
        myLogger.log("service id = " + id);
    }
}
```

- 비즈니스 로직이 있는 서비스 계층에서도 로그 출력
- request scope의 MyLogger 덕분에 이런 부분을 파라미터로 넘기지 않고, MyLogger의 멤버변수에 저장해서 코드와 계층을 깔끔하게 유지할 수 있음



- **실제 기대와 달리 실행 시점에 오류 발생**

```
Error creating bean with name 'myLogger': Scope 'request' is not active for the 
current thread; consider defining a scoped proxy for this bean if you intend to 
refer to it from a singleton;
```

- 이 빈은 실제 고객의 요청이 와야 생성 가능!!

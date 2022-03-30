---
layout: single
title:  "빈 스코프"
toc: true
toc_sticky: true
categories:
  - Spring_Basic
---

#  request 스코프 예제 개발

# 스코프와 Provider



## 스코프와 Provider

- 첫번째 해결방안은 앞서 배운 Provider를 사용하는 것

```java
@Controller
@RequiredArgsConstructor
public class LogDemoController {
    private final LogDemoService logDemoService;
    private final ObjectProvider<MyLogger> myLoggerProvider;
    @RequestMapping("log-demo")
    @ResponseBody
    public String logDemo(HttpServletRequest request) {
        String requestURL = request.getRequestURL().toString();
        MyLogger myLogger = myLoggerProvider.getObject();
        myLogger.setRequestURL(requestURL);
        myLogger.log("controller test");
        logDemoService.logic("testId");
        return "OK";
    }
}
```

```java
@Service
@RequiredArgsConstructor
public class LogDemoService {
    private final ObjectProvider<MyLogger> myLoggerProvider;
    public void logic(String id) {
        MyLogger myLogger = myLoggerProvider.getObject();
        myLogger.log("service id = " + id);
    }
}
```



- 로그

  ```
  [d06b992f...] request scope bean create
  [d06b992f...][http://localhost:8080/log-demo] controller test
  [d06b992f...][http://localhost:8080/log-demo] service id = testId
  [d06b992f...] request scope bean close
  ```

- `ObjectProvider` 덕분에 `ObjectProvider.getObject()`를 호출하는 시점까지 request scope **빈의 생성을 지연**할 수 있다

-  `ObjectProvider.getObject()`를 호출하는 시점에는 HTTP 요청이 진행중이므로 request scope 빈의 생성이 정상 처리됨

-  `ObjectProvider.getObject()`를 `LogDemoController`, `LogDemoService`에서 각각 한번씩 따로 호출해도 같은 HTTP 요청이면 같은 스프링 빈이 반환됨!

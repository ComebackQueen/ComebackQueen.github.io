---
layout: single
title:  "서블릿"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

# Http 요청 데이터(2)



## API 메시지 바디 - 단순 텍스트

- **HTTP message body**에 데이터를 직접 담아서 요청
  - HTTP API에서 주로 사용, JSON, XML, TEXT
  - 데이터 형식은 주로 JSON 사용
  - POST, PUT, PATCH

- RequestBodyStringServlet

```java
@WebServlet(name = "requestBodyStringServlet", urlPatterns = "/request-body-string")
public class RequestBodyStringServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        ServletInputStream inputStream = request.getInputStream(); // 데이터를 InputStream을 사용하여 직접 읽음
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        System.out.println("messageBody = " + messageBody);

        response.getWriter().write("ok");
    }
}
```

- 문자 전송
  - POST http://localhost:8080/request-body-string 
  - content-type: text/plain 
  - message body: hello 
  - 결과: messageBody = hello

---



## API 메시지 바디 - JSON

- JSON 형식 전송
  - POST http://localhost:8080/request-body-json 
  - content-type: **application/json** 
  - message body: {"username": "hello", "age": 20} 
  - 결과: messageBody = {"username": "hello", "age": 20}
- JSON 형식 파싱 추가

```java
/*
lombok이 제공하는 @Getter, @Setter 덕분에 getter/setter 메서드를 따로 생성하지 않아도 된다!
*/
@Getter @Setter
public class HelloData {

    private String username;
    private int age;

}
```

- RequestBodyJsonServlet

```java
@WebServlet(name = "requestBodyJsonServlet", urlPatterns = "/request-body-json")
public class RequestBodyJsonServlet extends HttpServlet {

    private ObjectMapper objectMapper = new ObjectMapper();
	// Jackson 라이브러리 -> JSON 결과 파싱
    
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        ServletInputStream inputStream = request.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        System.out.println("messageBody = " + messageBody);

        HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);

        System.out.println("helloData.username = " + helloData.getUsername());
        System.out.println("helloData.age = " + helloData.getAge());

        response.getWriter().write("ok");
    }
}
```



- 출력결과

```java
messageBody={"username": "hello", "age", 20}
data.username=hello
data.age=20
```

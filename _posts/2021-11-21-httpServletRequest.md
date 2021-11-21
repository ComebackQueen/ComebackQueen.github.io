---
layout: single
title:  "서블릿"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

# HttpServletResponse



## HttpServletResponse의 역할

- **HTTP 응답 메시지 생성**
  - HTTP 응답코드 지정
  - 헤더 생성
  - 바디 생성

- **편의 기능 제공**
  - Content-Type, 쿠키, Redirect


---



## HttpServletResponse - 기본 사용법



- ResponseHeaderServlet.java

```java
@WebServlet(name = "responseHeaderServlet", urlPatterns = "/response-header")
public class ResponseHeaderServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //[status-line]
        response.setStatus(HttpServletResponse.SC_OK);

        //[response-header]
        response.setHeader("Content-Type", "text/plain;charset=utf-8");
        response.setHeader("Cache-Control", "no-cache, no-store, must-revalidate");
        response.setHeader("Pragma", "no-cache");
        response.setHeader("my-header", "hello");

        //[Header 편의 메서드]
        //content(response);
        //cookie(response);
        redirect(response);

        PrintWriter writer = response.getWriter();
        writer.println("ok");
    }

    private void content(HttpServletResponse response) {
        //Content-Type: text/plain;charset=utf-8
        //Content-Length: 2
        //response.setHeader("Content-Type", "text/plain;charset=utf-8");
        response.setContentType("text/plain");
        response.setCharacterEncoding("utf-8");
        //response.setContentLength(2); //(생략시 자동 생성)
    }

    private void cookie(HttpServletResponse response) {
        //Set-Cookie: myCookie=good; Max-Age=600;
        //response.setHeader("Set-Cookie", "myCookie=good; Max-Age=600");
        Cookie cookie = new Cookie("myCookie", "good");
        cookie.setMaxAge(600); //600초
        response.addCookie(cookie);
    }

    private void redirect(HttpServletResponse response) throws IOException {
        //Status Code 302
        //Location: /basic/hello-form.html
        //response.setStatus(HttpServletResponse.SC_FOUND); //302
        //response.setHeader("Location", "/basic/hello-form.html");
        response.sendRedirect("/basic/hello-form.html");
    }

}
```

---



## HTTP 응답 데이터 - 단순 텍스트, HTML



- 단순 텍스트 응답 (writer.println("ok");)
- HTML 응답
- HTTP API - MessageBody JSON 응답



※ HTTP 응답으로 JSON을 반환할 때는 content-type을 <mark style='background-color: #fff5b1'>applicaion/json</mark>로 지정해야 한다.


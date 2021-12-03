---
layout: single
title:  "스프링 MVC - 구조 이해"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

#  HTTP 응답 - 정적 리소스, 뷰 템플릿



## 들어가기 전



### ※ 스프링에서 응답 데이터를 만드는 방법은 크게 3가지

- **정적 리소스**
  - ex) 웹 브라우저에 정적인 HTML, css, js를 제공할 때
- **뷰 템플릿 사용**
  - ex) 웹 브라우저에 동적인 HTML을 제공할 때
- **HTTP 메시지 사용**
  - HTTP API를 제공하는 경우에는 HTML이 아니라 데이터를 전달해야 하므로, HTTP 메시지 바디에 JSON 같은 형식으로 데이터를 실어 보낸다

---



## 정적 리소스

- 스프링 부트는 클래스패스의 다음 디렉토리에 있는 정적 리소스를 제공

  - /static, /public, /resources, /META-INF/resources

- **정적 리소스 경로**

  - src/main/resources/static

  - ex) 만약 (src/main/resources/static/basic/hello-form.html) 경로에 파일이 들어있으면 →

    웹 브라우저에서 (http://localhost:8080/basic/hello-form.html)와 같이 실행 된다

---



## 뷰 템플릿

 ⁕ 뷰 템플릿을 거쳐서 HTML이 생성되고, 뷰가 응답을 만들어서 전달

스프링 부트는 기본 뷰 템플릿 경로를 제공



- **뷰 템플릿 경로**

  - src/main/resources/templates

- **뷰 템플릿 생성**

  - src/main/resources/templates/response/hello.html

  ```html
  <!DOCTYPE html>
  <html xmlns:th="http://www.thymeleaf.org">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
  <p th:text="${data}">empty</p>
  </body>
  </html>
  ```

  

- **ResponseViewController - 뷰 템플릿을 호출하는 컨트롤러**

```java
@Controller
public class ResponseViewController {
	
    @RequestMapping("/response-view-v1")
    public ModelAndView responseViewV1() {
        ModelAndView mav = new ModelAndView("response/hello")
                .addObject("data", "hello!");

        return mav;
    }
	
    /**
	* String을 반환하는 경우 - View or HTTP 메시지
	* @ResponseBody가 없으면 response/hello로 뷰 리졸버가 실행되어서 뷰를 찾고, 렌더링
	* @ResponseBody가 있으면 뷰 리졸버를 실행X, HTTP 메시지 바디에 직접 response/hello라는 문자가 입력됨
	*/
    
    @RequestMapping("/response-view-v2")
    public String responseViewV2(Model model) {
        model.addAttribute("data", "hello!");

        return "response/hello";
    }
	
    /**
	* Void를 반환하는 경우
	* @Controller를 사용하고, HttpServletResponse , OutputStream(Writer) 같은 파라미터가 없으면
	* 요청 URL을 참고해서 논리 뷰 이름으로 사용(명시성이 떨어지고 딱 맞는 경우도 잘 없어서 권장X)
	*/
    
    @RequestMapping("/response/hello")
    public void responseViewV3(Model model) {
        model.addAttribute("data", "hello!");
    }
}
```


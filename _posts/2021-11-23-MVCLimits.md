---
layout: single
title:  "서블릿, JSP, MVC 패턴"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

# MVC 패턴 - 한계



## MVC 컨트롤러의 단점



- **포워드 중복**

  - View로 이동하는 코드가 항상 중복 호출되어야 한다. 해당 메서드 또한 항상 직접 호출해야 한다.

  ```java
  RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
  dispatcher.forward(request, response);
  ```



- **ViewPath에 중복**

```java
String viewPath = "/WEB-INF/views/new-form.jsp";
```

 prefix : /WEB-INF/views/, suffix : .jsp

⁕ 만약 jsp가 아닌 thymeleaf 같은 다른 뷰로 변경한다면 전체 코드를 다 변경해야 한다.



- **사용하지 않는 코드**
  - HttpServletRequest request, HttpServletResponse response
  - response는 현재 코드에서 사용되지 않고, HttpServletRequest/HttpServletResponse를 사용하는 코드는 테스트 케이스를 작성하기도 어려움



- **공통 처리가 어렵다**
  - 기능이 복잡해질수록 컨트롤러에서 공통으로 처리해야 하는 부분이 증가
  - 해당 메서드를 항상 호출해야 하고, 실수로 호출하지 않으면 문제가 발생(중복)



- **정리하면 공통 처리가 어렵다는 문제가 있다**
  - 이 문제를 해결하려면 컨트롤러 호출 전에 먼저 공통 기능을 처리해야 한다(수문장 역할)
  - <mark style='background-color: #fff5b1'>프론트 컨트롤러(Front Controller)</mark> 패턴을 도입 → 스프링 MVC의 핵심

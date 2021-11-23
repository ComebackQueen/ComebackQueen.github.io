---
layout: single
title:  "서블릿, JSP, MVC 패턴"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

# MVC 패턴 - 적용



- 서블릿 → 컨트롤러, JSP → 뷰
- Model은 HttpServletRequest 객체를 사용
- request.setAttribute(), request.getAttribute()를 사용하면 데이터를 보관하고 조회할 수 있음


---



## 회원 등록

- **회원 등록 폼 - 컨트롤러**

```java
@WebServlet(name = "mvcMemberFormServlet", urlPatterns = "/servlet-mvc/members/new-form")
public class MvcMemberFormServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String viewPath = "/WEB-INF/views/new-form.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```



- dispatcher.forward() : 다른 서블릿이나 JSP로 이동할 수 있는 기능. 서버 내부에서 다시 호출이 발생.
- **redirect vs forward**
  - 리다이렉트 : 클라이언트(웹 브라우저)가 인지할 수 있고, URL 경로도 실제로 변경됨
  - 포워드 : 서버 내부에서 일어나는 호출(클라이언트가 인지하지 못함)

---



- **회원 등록 폼 - 뷰**

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<form action="save" method="post">
    username: <input type="text" name="username" />
    age: <input type="text" name="age" />
    <button type="submit">전송</button>
</form>
</body>
</html>
```

※ form의 action -> 절대경로X, 상대경로O

상대경로를 사용하면 폼 전송시 현재 URL이 속한 계층 경로 + save가 호출

현재 계층 경로 : /servlet-mvc/members/

결과 : /servlet-mvc/members/save

---



## 회원 저장

- **회원 저장 - 컨트롤러**

```java
@WebServlet(name = "mvcMemberSaveServlet", urlPatterns = "/servlet-mvc/members/save")
public class MvcMemberSaveServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username,age);
        memberRepository.save(member);

        //Model에 데이터를 보관한다.
        request.setAttribute("member", member);

        String viewPath = "/WEB-INF/views/save-result.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

※ request가 제공하는 setAttribute()를 사용하면 request 객체에 데이터를 보관해서 뷰에 전달할 수 있다.

뷰는 <mark style='background-color: #fff5b1'>request.getAttribute()</mark>를 사용하여 데이터를 꺼낸다.

---



- **회원 저장 - 뷰**

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
성공
<ul>
    <li>id=${member.id}</li>
    <li>username=${member.username}</li>
    <li>age=${member.age}</li>
</ul>
<a href="/index.html">메인</a>
</body>
</html>
```

※ <%= request.getAttribute("member")%>로 모델에 저장한 member 객체를 꺼낼 수 있지만, 너무 복잡해진다.

→ 이를 위해, ${}문법을 제공 (request의 attribute에 담긴 데이터를 편리하게 조회 가능)

---



## 회원 목록 조회

- **회원 목록 조회 - 컨트롤러**

```java
@WebServlet(name = "mvcMemberListServlet", urlPatterns = "/servlet-mvc/members")
public class MvcMemberListServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        List<Member> members = memberRepository.findAll();

        request.setAttribute("members", members);
        /*
        request 객체를 사용하여 List<Member> members를 
        모델에 보관
        */

        String viewPath = "/WEB-INF/views/members.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);

    }
}
```

---

- **회원 목록 조회 - 뷰**

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<html>
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<a href="/index.html">메인</a>
<table>
    <thead>
    <th>id</th>
    <th>username</th>
    <th>age</th>
    </thead>
    <tbody>
    <c:forEach var="item" items="${members}">
        <tr>
            <td>${item.id}</td>
            <td>${item.username}</td>
            <td>${item.age}</td>
        </tr>
    </c:forEach>
    </tbody>
</table>
</body>
</html>
```

※  members를 taglib기능을 사용해서 반복하면서 출력

※  <c: forEach>를 사용하려면 다음과 같이 선언해야 한다.

<mark style='background-color: #fff5b1'><%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%></mark>

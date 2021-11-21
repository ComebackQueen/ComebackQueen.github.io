---
layout: single
title:  "서블릿, JSP, MVC 패턴"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

# JSP로 회원 관리 웹 애플리케이션 만들기



- **회원 등록 폼 JSP**
  
  ```html
  <%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <html>
  <head>
      <title>Title</title>
  </head>
  <body>
  <form action="/jsp/members/save.jsp" method="post">
      username: <input type="text" name="username" />
      age: <input type="text" name="age" />
      <button type="submit">전송</button>
  </form>
  </body>
  </html>
  ```
  
- <%@ page contentType="text/html;charset=UTF-8" language="java" %>

  - 첫 줄은 JSP문서라는 뜻


---



- **회원 저장 JSP**

```html
<%@ page import="hello.servlet.domain.member.Member" %>
<%@ page import="hello.servlet.domain.member.MemberRepository" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
    //request, response 사용 가능
    MemberRepository memberRepository = MemberRepository.getInstance();

    System.out.println("MemberSaveServlet.service");
    String username = request.getParameter("username");
    int age = Integer.parseInt(request.getParameter("age"));

    Member member = new Member(username,age);
    memberRepository.save(member);
%>
<html>
<head>
    <title>Title</title>
</head>
<body>
성공
<ul>
    <li>id=<%=member.getId()%></li>
    <li>username=<%=member.getUsername()%></li>
    <li>age=<%=member.getAge()%></li>
</ul>
<a href="/index.html">메인</a>
</body>
</html>
```

**JSP는 자바 코드를 그대로 다 사용할 수 있다**

- <%@ page import="hello.servlet.domain.member.Member" %>
  - 자바의 import 문과 같음
- <% ~~ %>
  - 자바 코드를 입력
- <%= ~~ %>
  - 자바 코드를 출력

---



- **회원 목록 JSP**

```html
<%@ page import="hello.servlet.domain.member.MemberRepository" %>
<%@ page import="hello.servlet.domain.member.Member" %>
<%@ page import="java.util.List" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
    MemberRepository memberRepository = MemberRepository.getInstance();
    List<Member> members = memberRepository.findAll();
%>

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
    <%
        for (Member member : members) {
            out.write(" <tr>");
            out.write(" <td>" + member.getId() + "</td>");
            out.write(" <td>" + member.getUsername() + "</td>");
            out.write(" <td>" + member.getAge() + "</td>");
            out.write(" </tr>");
        }
    %>
    </tbody>
</table>
</body>
</html>
```

---



## 서블릿과 JSP의 한계

- 서블릿의 한계 : 뷰화면을 위한 HTML을 만드는 작업이 지저분하고 복잡
- JSP의 한계 : JSP가 너무 많은 역할을 함(JAVA 코드, 데이터 조회 등 다양한 코드 노출) -> 유지보수 복잡



## MVC 패턴의 등장

- 비즈니스 로직 -> 서블릿, 화면(view) 그리기 -> JSP

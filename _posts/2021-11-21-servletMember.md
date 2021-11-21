---
layout: single
title:  "서블릿, JSP, MVC 패턴"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

# 서블릿으로 회원 관리 웹 애플리케이션 만들기



- **MemberFormServlet - 회원 등록 폼**
  
  ```java
  @WebServlet(name = "memberFormServlet", urlPatterns = "/servlet/members/new-form")
  public class MemberFormServlet extends HttpServlet {
  
      private MemberRepository memberRepository = MemberRepository.getInstance();
  
      @Override
      protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
          response.setContentType("text/html");
          response.setCharacterEncoding("utf-8");
  
          PrintWriter w = response.getWriter();
          w.write("<!DOCTYPE html>\n" +
                  "<html>\n" +
                  "<head>\n" +
                  " <meta charset=\"UTF-8\">\n" +
                  " <title>Title</title>\n" +
                  "</head>\n" +
                  "<body>\n" +
                  "<form action=\"/servlet/members/save\" method=\"post\">\n" +
                  " username: <input type=\"text\" name=\"username\" />\n" +
                  " age: <input type=\"text\" name=\"age\" />\n" +
                  " <button type=\"submit\">전송</button>\n" +
                  "</form>\n" +
                  "</body>\n" +
                  "</html>\n");
      }
  }
  ```

- 단순하게 회원 정보를 입력할 수 있는 HTML Form을 만들어서 응답

---



- **MemberSaveServlet - 회원 저장**

```java
@WebServlet(name = "memberSaveServlet", urlPatterns = "/servlet/members/save")
public class MemberSaveServlet extends HttpServlet {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("MemberSaveServlet.service");
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username,age);
        memberRepository.save(member);

        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");
        PrintWriter w = response.getWriter();
        w.write("<html>\n" +
                "<head>\n" +
                " <meta charset=\"UTF-8\">\n" +
                "</head>\n" +
                "<body>\n" +
                "성공\n" +
                "<ul>\n" +
                " <li>id="+member.getId()+"</li>\n" +
                " <li>username="+member.getUsername()+"</li>\n" +
                " <li>age="+member.getAge()+"</li>\n" +
                "</ul>\n" +
                "<a href=\"/index.html\">메인</a>\n" +
                "</body>\n" +
                "</html>");
    }
}
```

**MemberSaveServlet은 다음 순서로 동작**

1. 파라미터를 조회해서 Member 객체 생성
2. Member 객체를 MemberRepository를 통해서 저장
3. Member 객체를 사용해서 결과 화면용 HTML을 동적으로 만들어서 응답

---



- **MemberListServlet - 회원 목록**

```java
@WebServlet(name = "memberListServlet", urlPatterns = "/servlet/members")
public class MemberListServlet extends HttpServlet {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        List<Member> members = memberRepository.findAll();

        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");

        PrintWriter w = response.getWriter();
        w.write("<html>");
        w.write("<head>");
        w.write(" <meta charset=\"UTF-8\">");
        w.write(" <title>Title</title>");
        w.write("</head>");
        w.write("<body>");
        w.write("<a href=\"/index.html\">메인</a>");
        w.write("<table>");
        w.write(" <thead>");
        w.write(" <th>id</th>");
        w.write(" <th>username</th>");
        w.write(" <th>age</th>");
        w.write(" </thead>");
        w.write(" <tbody>");
        for (Member member : members) {
            w.write(" <tr>");
            w.write(" <td>" + member.getId() + "</td>");
            w.write(" <td>" + member.getUsername() + "</td>");
            w.write(" <td>" + member.getAge() + "</td>");
            w.write(" </tr>");
        }
        w.write(" </tbody>");
        w.write("</table>");
        w.write("</body>");
        w.write("</html>");
    }
}
```

**MemberListServlet은 다음 순서로 동작**

1. memberRepository.findAll()을 통해 모든 회원을 조회
2. 회원 목록 HTML을 for 루프를 통해서 회원 수 만큼 동적으로 생성 및 응답

---

## 템플릿 엔진으로

- 서블릿으로 동적인 HTML을 만드는 것은 매우 비효율적임
- HTML 문서에 동적으로 변경해야 하는 부분만 자바 코드를 넣는 것이 더 편리(템플릿 엔진)
- 템플릿 엔진 : JSP, Thymeleaf, Freemarker, Velocity

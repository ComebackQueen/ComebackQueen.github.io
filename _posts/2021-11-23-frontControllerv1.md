---
layout: single
title:  "MVC 프레임워크 만들기"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

#  프론트 컨트롤러 도입 - v1



## v1 구조

![](/assets/images/2021-11-23-frontControllerv1/1.JPG)



- **ControllerV1**

```java
public interface ControllerV1 {

    void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;

}
```

⁕ 서블릿과 비슷한 모양의 컨트롤러 인터페이스 도입



- **MemberFormControllerV1 - 회원 등록 컨트롤러**

  ```java
  public class MemberFormControllerV1 implements ControllerV1 {
  
      @Override
      public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
          String viewPath = "/WEB-INF/views/new-form.jsp";
          RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
          dispatcher.forward(request, response);
      }
  }
  ```

- **MemberSaveControllerV1 - 회원 저장 컨트롤러**

```java
public class MemberSaveControllerV1 implements ControllerV1 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
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

- **MemberListControllerV1 - 회원 목록 컨트롤러**

```java
public class MemberListControllerV1 implements ControllerV1 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        List<Member> members = memberRepository.findAll();

        request.setAttribute("members", members);

        String viewPath = "/WEB-INF/views/members.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

※ 내부 로직은 기존 서블릿과 거의 같음



- **FrontControllerServletV1 - 프론트 컨트롤러**

```java
@WebServlet(name = "frontControllerServletV1", urlPatterns = "/front-controller/v1/*")
public class FrontControllerServletV1 extends HttpServlet {
    private Map<String, ControllerV1> controllerMap = new HashMap<>();
    public FrontControllerServletV1() {
        controllerMap.put("/front-controller/v1/members/new-form", new
                MemberFormControllerV1());
        controllerMap.put("/front-controller/v1/members/save", new
                MemberSaveControllerV1());
        controllerMap.put("/front-controller/v1/members", new
                MemberListControllerV1());
    }
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse
            response)
            throws ServletException, IOException {
        System.out.println("FrontControllerServletV1.service");
        String requestURI = request.getRequestURI();
        ControllerV1 controller = controllerMap.get(requestURI);
        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }
        controller.process(request, response);
    }
}
```

---



## 프론트 컨트롤러 분석



- **urlPatterns**
  - urlPatterns = "/front-controller/v1/*" : /front-controller/v1 를 포함한 하위 모든 요청은 이 서블릿에서 받아들인다. 
  - ex) /front-controller/v1 , /front-controller/v1/a , /front-controller/v1/a/b
- **ControllerMap**
  - key : 매핑 URL
  - value : 호출될 컨트롤러
- **service()**
  - 먼저 requestURI를 조회해서 실제 호출할 컨트롤러를 controllerMap에서 찾는다
  - 만약 없다면 404 상태 코드 반환
  - 컨트롤러 찾고 controller.process(request, response);을 호출해서 해당 컨트롤러를 실행

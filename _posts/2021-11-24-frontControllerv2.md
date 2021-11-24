---
layout: single
title:  "MVC 프레임워크 만들기"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

#  View 분리 - v2

⁕ v1에서 모든 컨트롤러에서 뷰로 이동하는 부분에 중복이 있고, 깔끔하지 않다

```java
String viewPath = "/WEB-INF/views/new-form.jsp";
RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
dispatcher.forward(request, response);
```



## v2 구조

![](/assets/images/2021-11-23-frontControllerv2/1.JPG)



- **MyView**

```java
public class MyView {
    private String viewPath;

    public MyView(String viewPath) {
        this.viewPath = viewPath;
    }

    public void render(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException{
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }

    public void render(Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        modelToRequestAttribute(model, request);
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }

    private void modelToRequestAttribute(Map<String, Object> model, HttpServletRequest request) {
        model.forEach((key, value) -> request.setAttribute(key, value)); // 들어오는 키,벨류값을 루프를 돌려서 받음
    }
}
```

⁕ 컨트롤러가 뷰를 반환하는 특징



- **ControllerV2**

  ```java
  public interface ControllerV2 {
      MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;
  }
  ```
  
  
  
- **MemberFormControllerV2 - 회원 등록 폼**

```java
public class MemberFormControllerV2 implements ControllerV2 {

    @Override
    public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        return new MyView("/WEB-INF/views/new-form.jsp");
    }
}
```

 ※ 각 컨트롤러는 복잡한 dispatcher.forward()를 직접 생성해서 호출하지 않아도 됨. -> MyView 객체를 생성하고 거기에 뷰 이름만 넣고 반환하면 됨.



- **MemberSaveControllerV2 - 회원 저장**

```java
public class MemberSaveControllerV2 implements ControllerV2 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username,age);
        memberRepository.save(member);

        //Model에 데이터를 보관한다.
        request.setAttribute("member", member);

        return new MyView("/WEB-INF/views/save-result.jsp");
    }
}
```



- **MemberListControllerV2 - 회원 목록**

```java
public class MemberListControllerV2 implements ControllerV2 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        List<Member> members = memberRepository.findAll();

        request.setAttribute("members", members);

        return new MyView("/WEB-INF/views/members.jsp");
    }
}
```



- **프론트 컨트롤러 V2**

```java
@WebServlet(name = "frontControllerServletV2", urlPatterns = "/front-controller/v2/*")
public class FrontControllerServletV2 extends HttpServlet {
    private Map<String, ControllerV2> controllerMap = new HashMap<>();
    public FrontControllerServletV2() {
        controllerMap.put("/front-controller/v2/members/new-form", new
                MemberFormControllerV2());
        controllerMap.put("/front-controller/v2/members/save", new
                MemberSaveControllerV2());
        controllerMap.put("/front-controller/v2/members", new
                MemberListControllerV2());
    }
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse
            response)
            throws ServletException, IOException {

        String requestURI = request.getRequestURI();
        ControllerV2 controller = controllerMap.get(requestURI);
        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        MyView view = controller.process(request, response);
        view.render(request, response);
    }
}
```

 ※ ControllerV2의 반환 타입이 **MyView**이므로 프론트 컨트롤러는 컨트롤러의 호츌 결과로 **MyView**를 반환받는다.

※ view.render()를 호출하면 forward 로직을 수행해서 JSP가 실행됨.



## 정리



```java
public void render(HttpServletRequest request, HttpServletResponse response)
throws ServletException, IOException {
 RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
 dispatcher.forward(request, response);
}
```

- 프론트 컨트롤러의 도입으로 **MyView** 객체의 **render()**를 호출하는 부분을 모두 일관되게 처리할 수 있다. 각각의 컨트롤러는 **MyView** 객체를 생성만 해서 반환하면 된다.

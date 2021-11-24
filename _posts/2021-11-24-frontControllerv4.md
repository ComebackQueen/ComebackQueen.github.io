---
layout: single
title:  "MVC 프레임워크 만들기"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

#  단순하고 실용적인 컨트롤러 - v4



## 들어가기 전 - 이전 버전의 문제점



- ModelView 객체를 생성하고 반환해야 하는 번거로움
- 개발자 또한 편리하게 사용할 수 있어야 함

---



## V4 구조

![](/assets/images/2021-11-24-frontControllerv4/1.JPG)

 ※ 기본적인 구조는 V3와 같다. 대신 컨트롤러가 ModelView를 반환하지 않고, 

​      ViewName만 반환.



- **ControllerV4**

```java
public interface ControllerV4 {
    /**
     *
     * @param paramMap
     * @param model
     * @return viewName
     */
    String process(Map<String, String> paramMap, Map<String, Object> model);
}
```

- 이번 버전은 인터페이스에서 ModelView가 없다




- **MemberFormControllerV4**

  ```java
  package hello.servlet.web.frontcontroller.v4.Controller;
  
  import hello.servlet.web.frontcontroller.v4.ControllerV4;
  
  import java.util.Map;
  
  public class MemberFormControllerV4 implements ControllerV4 {
      @Override
      public String process(Map<String, String> paramMap, Map<String, Object> model) {
          return "new-form"; // 뷰의 논리 이름만 반환하면 된다
      }
  }
  
  ```
  



- **MemberSaveControllerV4**

```java
public class MemberSaveControllerV4 implements ControllerV4 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public String process(Map<String, String> paramMap, Map<String, Object> model) {
        String username = paramMap.get("username");
        int age = Integer.parseInt(paramMap.get("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        model.put("member", member);
        return "save-result";
    }
}
```

 ※ model.put("member", member);

​	모델이 파라미터로 전달되기 때문에, 모델을 직접 생성하지 않아도 된다.



- **MemberListControllerV4**

```java
public class MemberListControllerV4 implements ControllerV4 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public String process(Map<String, String> paramMap, Map<String, Object> model) {
        List<Member> members = memberRepository.findAll();

        model.put("members", members);
        return "members";
    }
}
```



- **FrontControllerServletV4**

```java
@WebServlet(name = "frontControllerServletV4", urlPatterns = "/front-controller/v4/*")
public class FrontControllerServletV4 extends HttpServlet {
    private Map<String, ControllerV4> controllerMap = new HashMap<>();
    public FrontControllerServletV4() {
        controllerMap.put("/front-controller/v4/members/new-form", new
                MemberFormControllerV4());
        controllerMap.put("/front-controller/v4/members/save", new
                MemberSaveControllerV4());
        controllerMap.put("/front-controller/v4/members", new
                MemberListControllerV4());
    }
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse
            response)
            throws ServletException, IOException {

        String requestURI = request.getRequestURI();
        ControllerV4 controller = controllerMap.get(requestURI);
        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        //paramMap

        Map<String, String> paramMap = createParamMap(request);
        Map<String, Object> model = new HashMap<>(); // 추가
        String viewName = controller.process(paramMap, model);

        MyView view = viewResolver(viewName);

        view.render(model,request, response);
    }

    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }
}
```



- **뷰의 논리 이름을 직접 반환**

```java
String viewName = controller.process(paramMap, model);
MyView view = viewResolver(viewName);
```

 컨트롤러가 직접 뷰의 논리 이름을 반환하므로 이 값을 사용해서 실제 물리 뷰를 찾을 수 있다.



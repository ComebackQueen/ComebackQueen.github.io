---
layout: single
title:  "MVC 프레임워크 만들기"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

#  Model 추가 - v3



## 들어가기 전



- **서블릿 종속성 제거**
  - 컨트롤러 입장에서 HttpServletRequest, HttpServletResponse의 필요성?
  - 별도의 Model 객체를 만들어서 반환! → <mark style='background-color: #fff5b1'>구현 코드 단순, 테스트 코드 작성이 쉬움</mark>
- **뷰 이름 중복 제거**
  - 컨트롤러가 뷰의 논리 이름을 반환하고, 실제 물리 위치의 이름은 프론트 컨트롤러에서 처리하도록 단순화

​			⁕ /WEB-INF/views/new-form.jsp → **new-form**

​			⁕ /WEB-INF/views/save-result.jsp → **save-result**

​			⁕ /WEB-INF/views/members.jsp →  **members**

---



## V3 구조

![](/assets/images/2021-11-24-frontControllerv3/1.JPG)



- **ModelView**

```java
public class ModelView {
    private String viewName;
    private Map<String, Object> model = new HashMap<>();

    public ModelView(String viewName) {
        this.viewName = viewName;
    }

    public String getViewName() {
        return viewName;
    }

    public void setViewName(String viewName) {
        this.viewName = viewName;
    }

    public Map<String, Object> getModel() {
        return model;
    }

    public void setModel(Map<String, Object> model) {
        this.model = model;
    }
}
```

- 서블릿의 종속성 제거를 위해 Model을 직접 만들고, 추가로 View 이름까지 전달하는 객체.




- **ControllerV3**

  ```java
  public interface ControllerV3 {
      ModelView process(Map<String, String> paramMap);
  }
  
  ```
  

 ⁕ 이 컨트롤러는 서블릿 기술을 전혀 사용 X(구현 단순, 테스트 하기 쉬움)



- **MemberFormControllerV3 - 회원 등록 폼**

```java
public class MemberFormControllerV3 implements ControllerV3 {
    @Override
    public ModelView process(Map<String, String> paramMap) {
        return new ModelView("new-form");
        // ModelView를 생성할 때 new-form이라는 view의 논리적인 이름을 지정
    }
}
```





- **MemberSaveControllerV3 - 회원 저장**

```java
public class MemberSaveControllerV3 implements ControllerV3 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public ModelView process(Map<String, String> paramMap) {
        String username = paramMap.get("username");
        int age = Integer.parseInt(paramMap.get("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        ModelView mv = new ModelView("save-result");
        mv.getModel().put("member", member);
        return mv;
    }
}
```

- paramMap.get("username");
  - 파라미터 정보는 map에 담겨있다. map에서 필요한 요청 파라미터를 조회하면 된다
- mv.getModel().put("member", member);
  - 모델은 단순한 map이므로 모델에 뷰에서 필요한 member 객체를 담고 반환한다.



- **MemberListControllerV3 - 회원 목록**

```java
public class MemberListControllerV3 implements ControllerV3 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public ModelView process(Map<String, String> paramMap) {
        List<Member> members = memberRepository.findAll();
        ModelView mv = new ModelView("members");
        mv.getModel().put("members", members);
        return mv;
    }
}
```



- **FrontControllerServletV3**

```java
@WebServlet(name = "frontControllerServletV3", urlPatterns = "/front-controller/v3/*")
public class FrontControllerServletV3 extends HttpServlet {
    private Map<String, ControllerV3> controllerMap = new HashMap<>();
    public FrontControllerServletV3() {
        controllerMap.put("/front-controller/v3/members/new-form", new
                MemberFormControllerV3());
        controllerMap.put("/front-controller/v3/members/save", new
                MemberSaveControllerV3());
        controllerMap.put("/front-controller/v3/members", new
                MemberListControllerV3());
    }
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse
            response)
            throws ServletException, IOException {

        String requestURI = request.getRequestURI();
        ControllerV3 controller = controllerMap.get(requestURI);
        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        //paramMap

        Map<String, String> paramMap = createParamMap(request);
        ModelView mv = controller.process(paramMap);

        String viewName = mv.getViewName();

        MyView view = viewResolver(viewName);

        view.render(mv.getModel(),request, response);
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

-  createParamMap()
  - HttpServletRequest에서 파라미터 정보를 꺼내서 Map으로 반환
  - 해당 Map(paramMap)을 컨트롤러에 전달하면서 호출

---



## 뷰 리졸버



**MyView view = viewResolver(viewName)**

 ⁕ 컨트롤러가 반환한 논리 뷰 이름을 실제 물리 뷰 경로로 변경. 그리고 실제 물리 경로가 있는 MyView 객체를 반환.

- 논리 뷰 이름 : members
- 물리 뷰 경로 : /WEB-INF/views/members.jsp



**view.render(mv.getModel(), request, response)**

- 뷰 객체를 통해서 HTML 화면을 렌더링
- 뷰 객체의 render()는 모델 정보도 함께 받는다
- JSP는 request.getAttribute() 로 데이터를 조회하기 때문에, 모델의 데이터를 꺼내서 request.setAttribute() 로 담아둔다.
- JSP로 포워드 해서 JSP를 렌더링 한다.

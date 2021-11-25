---
layout: single
title:  "MVC 프레임워크 만들기"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

#  유연한 컨트롤러 - v5(1)



## 들어가기 전



- 어떤 개발자는 ControllerV3 방식으로 개발하고 싶고, 어떤 개발자는 ControllerV4 방식으로 개발하고 싶다면???

- **어댑터 패턴**
  - v3가 110v이고, v4가 220v 전기 콘센트와 같이 **어탭터**를 활용.

---



## V5 구조

![](/assets/images/2021-11-25-frontControllerv5_1/1.JPG)

- **핸들러 어댑터** : 중간에 어댑터 역할을 하는 어댑터(다양한 종류의 컨트롤러 호출 가능)
- **핸들러** : 컨트롤러의 이름을 더 넓은 범위인 핸들러로 변경



- **MyHandlerAdapter**

```java
public interface MyHandlerAdapter {

    boolean supports(Object handler);

    ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException;
}

```

- boolean supports(Object handler);
  - handler는 컨트롤러
  - 어댑터가 해당 컨트롤러를 처리할 수 있는지 판단하는 메서드

- ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException;
  - 어댑터는 실제 컨트롤러를 호출하고, 그 결과로 ModelView를 반환





- **ControllerV3HandlerAdapter**

  ```java
  public class ControllerV3HandlerAdapter implements MyHandlerAdapter {
      @Override
      public boolean supports(Object handler) {
          return (handler instanceof ControllerV3);
          // ControllerV3를 처리할 수 있는 어댑터를 뜻함
      }
  
      @Override
      public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
          ControllerV3 controller = (ControllerV3) handler;
  
          Map<String, String> paramMap = createParamMap(request);
          ModelView mv = controller.process(paramMap);
  
          return mv;
      }
  
      private Map<String, String> createParamMap(HttpServletRequest request) {
          Map<String, String> paramMap = new HashMap<>();
          request.getParameterNames().asIterator()
                  .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
          return paramMap;
      }
  }
  ```
  



- **FrontControllerServletV5**

```java
@WebServlet(name = "frontControllerServletV5", urlPatterns = "/front-controller/v5/*")
public class FrontControllerServletV5 extends HttpServlet {

    private final Map<String, Object> handlerMappingMap = new HashMap<>();
    private final List<MyHandlerAdapter> handlerAdapters = new ArrayList<>();

    public FrontControllerServletV5() {
        initHandlerMappingMap(); // 핸들러 매핑 초기화(등록)
        initHandlerAdapters(); // 어댑터 초기화(등록)
    }

    private void initHandlerMappingMap() {
        handlerMappingMap.put("/front-controller/v5/v3/members/new-form", new
                MemberFormControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members/save", new
                MemberSaveControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members", new
                MemberListControllerV3());


        handlerMappingMap.put("/front-controller/v5/v4/members/new-form", new
                MemberFormControllerV4());
        handlerMappingMap.put("/front-controller/v5/v4/members/save", new
                MemberSaveControllerV4());
        handlerMappingMap.put("/front-controller/v5/v4/members", new
                MemberListControllerV4());
    }

    private void initHandlerAdapters() {
        handlerAdapters.add(new ControllerV3HandlerAdapter());
        handlerAdapters.add(new ControllerV4HandlerAdapter());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        Object handler = getHandler(request);

        if (handler == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        MyHandlerAdapter adapter = getHandlerAdapter(handler);
        //paramMap

        ModelView mv = adapter.handle(request, response, handler);

        String viewName = mv.getViewName();
        MyView view = viewResolver(viewName);

        view.render(mv.getModel(),request, response);
    }

    private Object getHandler(HttpServletRequest request) {
        String requestURI = request.getRequestURI();
        return handlerMappingMap.get(requestURI);
    }

    private MyHandlerAdapter getHandlerAdapter(Object handler) {
        for (MyHandlerAdapter adapter : handlerAdapters) {
            if(adapter.supports(handler)){
                return adapter;
            }
        }
        throw new IllegalArgumentException("handler adapter를 찾을 수 없습니다. handler=" + handler);
    }

    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }
}
```

 ※ 컨트롤러 → 핸들러

​	컨트롤러 뿐만 아니라 어댑터가 지원하기만 하면, 어떤 것이라도 URL에 매핑해서 사용할 수 있다.



- 매핑정보

  **private final Map handlerMappingMap = new HashMap<>();**

  - 매핑 정보의 값이 ControllerV3, V4 같은 인터페이스에서 아무 값이나 받을 수 있는 Object로 변경

- 핸들러 매핑

```java
private Object getHandler(HttpServletRequest request) {
        String requestURI = request.getRequestURI();
        return handlerMappingMap.get(requestURI);
    }
```

 ⁕ 핸들러 매핑 정보인 handlerMappingMap에서 URL에 매핑된 핸들러(컨트롤러) 객체를 찾아서 반환



- 핸들러를 처리할 수 있는 어댑터 조회

  ```
  for (MyHandlerAdapter adapter : handlerAdapters) {
              if(adapter.supports(handler)){
                  return adapter;
              }
          }
  ```

  - handler를 처리할 수 있는 어댑터를 adapter.supports(handler)를 통해서 찾는다
  - handler가 ControllerV3 인터페이스를 구현했다면, ControllerV3HandlerAdapter 객체가 반환

- 어댑터 호출

ModelView mv = adapter.handle(request, response, handler);

​	※ 어댑터의 handle(request, response, handler) 메서드를 통해 실제 어댑터 호출된다.


---
layout: single
title:  "스프링 MVC - 구조 이해"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

#  스프링 MVC 전체 구조



## 직접 만든 MVC 프레임워크 VS 스프링 MVC



- **직접 만든 MVC 프레임워크 구조**

![](/assets/images/2021-11-25-springMVC/1.JPG)

- **SpringMVC구조**

![](/assets/images/2021-11-25-springMVC/2.JPG)



- 직접 만든 프레임워크 → 스프링 MVC 비교
  - FrontController → DispatcherServlet
  - handlerMappingMap → HandlerMapping
  - MyHandlerAdapter → HandlerAdapter
  - ModelView → ModelAndView
  - viewResolver → ViewResolver
  - MyView → View

---



## DispatcherServlet 구조 살펴보기

org.springframework.web.servlet.DispatcherServlet

**디스패처 서블릿이 바로 스프링 MVC의 핵심!(프론트 컨트롤러)**



- **DispacherServlet 서블릿 등록**

  - DispacherServlet도 부모 클래스에서 HttpServlet을 상속 받아서 사용, 서블릿으로 동작

  - 스프링 부트는 DispacherServlet을 서블릿으로 자동 등록, 모든 경로

    (urlPatterns="/")에 대해서 매핑한다



- **요청 흐름**
  - 서블릿이 호출되면 HttpServlet이 제공하는 <mark style='background-color: #fff5b1'>service()</mark>가 호출
  - 스프링 MVC는 DispacherServlet의 부모인 FrameworkServlet에서 service()를 오버라이드 해두었다.
  - FrameworkServlet.service()를 시작으로 여러 메서드가 호출되면서 DispacherServlet.doDispatch()가 호출된다.



- **DispacherServlet.doDispatch()**

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse 
response) throws Exception {
HttpServletRequest processedRequest = request;
HandlerExecutionChain mappedHandler = null;
ModelAndView mv = null;
    
// 1. 핸들러 조회
mappedHandler = getHandler(processedRequest);
if (mappedHandler == null) {
noHandlerFound(processedRequest, response);
return;
}
    
// 2. 핸들러 어댑터 조회 - 핸들러를 처리할 수 있는 어댑터
HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
    
// 3. 핸들러 어댑터 실행 -> 4. 핸들러 어댑터를 통해 핸들러 실행 -> 5. ModelAndView 반환
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
    
processDispatchResult(processedRequest, response, mappedHandler, mv,
dispatchException);
}

private void processDispatchResult(HttpServletRequest request,
HttpServletResponse response, HandlerExecutionChain mappedHandler, ModelAndView 
mv, Exception exception) throws Exception {
    
// 뷰 렌더링 호출
render(mv, request, response);
    
}
protected void render(ModelAndView mv, HttpServletRequest request,
HttpServletResponse response) throws Exception {
    
View view;
String viewName = mv.getViewName();
    
// 6. 뷰 리졸버를 통해서 뷰 찾기, 7. View 반환
view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
    
// 8. 뷰 렌더링
view.render(mv.getModelInternal(), request, response);
}

```



- **SpringMVC 구조**

![](/assets/images/2021-11-25-springMVC/2.JPG)



- **동작 순서**
  1. **핸들러 조회** : 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러) 조회
  2. **핸들러 어댑터 조회**
  3. **핸들러 어댑터 실행**
  4. **핸들러 실행**
  5. **ModelAndView 반환** : 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환
  6. **ViewResolver 호출**
  7. **View 반환** : 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를 반환한다.
  8. **뷰 렌더링**

---



## 인터페이스 살펴보기

- 스프링 MVC의 큰 강점은 DispacherServlet 코드의 변경 없이, 원하는 기능을 변경하거나 확장할 수 있음



#### **주요 인터페이스 목록**

- 핸들러 매핑
- 핸들러 어댑터
- 뷰 리졸버
- 뷰

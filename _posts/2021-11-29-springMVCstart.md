---
layout: single
title:  "스프링 MVC - 구조 이해"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

#  스프링 MVC - 시작하기



## @RequestMapping



- **@RequestMapping**
  - RequestMappingHandlerMapping
  - RequestMappingHandlerAdapter




 ⁕ 가장 우선순위가 높은 핸들러 매핑과 핸들러 어댑터 → RequestMappingHandlerMapping, RequestMappingHandlerAdapter

 ⁕ 현재 스프링에서 주로 사용하는 애노테이션 기반의 핸들러 매핑과 어댑터



- **SpringMemberFormControllerV1 - 회원 등록 폼**

```java
@Controller
public class SpringMemberFormControllerV1 {

    @RequestMapping("springmvc/v1/members/new-form")
    public ModelAndView process(){
        return new ModelAndView("new-form");
    }
}
```

- **@Controller**
  - 스프링이 자동으로 스프링 빈으로 등록함(내부에 @Component 애노테이션이 있어서 컴포넌트 스캔 대상이 됨)
  - 스프링 MVC에서 애노테이션 기반 컨트롤러로 인식
- **@RequestMapping**
  - 요청 정보를 매핑
  - 해당 URL이 호출되면 이 메서드가 호출
  - 애노테이션 기반, 메서드의 이름은 임의로 지으면 됨
- **ModelAndView**
  - 모델과 뷰 정보를 담아서 반환하면 된다



- **SpringMemberSaveControllerV1 - 회원 저장**

```java
@Controller
public class SpringMemberSaveControllerV1 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/springmvc/v1/members/save")
    public ModelAndView process(HttpServletRequest request, HttpServletResponse response) {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        ModelAndView mv = new ModelAndView("save-result");
        mv.addObject("member", member);
        return mv;
    }
}
```

- **mv.addObject("member", member)**
  - ModelAndView를 통해 Model 데이터를 추가할 때는 addObject()를 사용하면 된다. 이 데이터는 이후 <mark style='background-color: #fff5b1'>뷰를 렌더링</mark> 할 때 사용된다.



- **SpringMemberListControllerV1 - 회원 목록**

```java
@Controller
public class SpringMemberListControllerV1 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/springmvc/v1/members")
    public ModelAndView process() {

        List<Member> members = memberRepository.findAll();

        ModelAndView mv = new ModelAndView("members");
        mv.addObject("members", members);
        return mv;
    }
}
```


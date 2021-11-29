---
layout: single
title:  "스프링 MVC - 구조 이해"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

#  스프링 MVC - 컨트롤러 통합



## 통합



- **@RequestMapping**을 잘 보면 클래스 단위가 아니라 메서드 단위에 적용됨 →

  컨트롤러 클래스를 유연하게 하나로 통합할 수 있음



- **SpringMemberControllerV2**

```java
@Controller
@RequestMapping("/springmvc/v2/members")
public class SpringMemberControllerV2 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/new-form")
    public ModelAndView newForm(){
        return new ModelAndView("new-form");
    }

    @RequestMapping("/save")
    public ModelAndView save(HttpServletRequest request, HttpServletResponse response) {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        ModelAndView mv = new ModelAndView("save-result");
        mv.addObject("member", member);
        return mv;
    }

    @RequestMapping
    public ModelAndView members() {

        List<Member> members = memberRepository.findAll();

        ModelAndView mv = new ModelAndView("members");
        mv.addObject("members", members);
        return mv;
    }
}
```

- **조합**
  - 컨트롤러 클래스를 통합하는 것을 넘어서 조합도 가능

```
@Controller
@RequestMapping("/springmvc/v2/members")
public class SpringMemberControllerV2 {}

/*
클래스 레벨 @RequestMapping("/springmvc/v2/members")

	메서드 레벨 @RequestMapping("/new-form") 
	-> /springmvc/v2/members/new-form
	메서드 레벨 @RequestMapping("/save")
	-> /springmvc/v2/members/save
	메서드 레벨 @RequestMapping 
	-> /springmvc/v2/members
*/
```


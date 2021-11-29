---
layout: single
title:  "스프링 MVC - 구조 이해"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

#  스프링 MVC - 실용적인 방식



## 실무에서 주로 쓰는 방식



- **SpringMemberControllerV3**

```java
public class SpringMemberControllerV3 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @GetMapping("/new-form")
    public String newForm(){
        return "new-form";
    }

    @PostMapping("/save")
    public String save(
            @RequestParam("username") String username,
            @RequestParam("age") int age,
            Model model) {

        Member member = new Member(username, age);
        memberRepository.save(member);

        model.addAttribute("member", member);
        return "save-result";
    }

    @GetMapping
    public String members(Model model) {
        List<Member> members = memberRepository.findAll();

        model.addAttribute("members", members);
        return "members";
    }

}
```

- **Model 파라미터**
  - save(), member()를 보면 Model를 파라미터로 받고 있음(스프링 MVC의 편의 기능)
- **ViewName 직접 반환**
  - 뷰의 논리 이름을 반환할 수 있다

- **@RequestParam 사용**
  - @RequestParam("username") == request.getParameter("username")

- **@RequestMapping → @GetMapping, @PostMapping**
  - @RequestMapping은 URL만 매칭하는 것이 아니라, HTTP Method도 함께 구분 가능
  - ex) @RequestMapping(value = "/new-form", method = RequestMethod.GET)
  - 위의 부분을 @GetMapping, @PostMapping으로 더 편리하게 사용할 수 있다
  - Get, Post, Put, Delete, Patch 모두 애노테이션이 준비되어 있다





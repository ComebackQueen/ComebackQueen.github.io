---
layout: single
title:  "스프링 MVC - 구조 이해"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

#  HTTP 메시지 컨버터



## 들어가기 전



- **HTTP 메시지 컨버터 위치**

![](/assets/images/2021-12-13-messageConverter/1.JPG)

-  **요청**
  - @RequestBody를 처리하는 ArgumentResolver가 있고, HttpEntity를 처리하는 ArgumentResolver가 있다
  - ArgumentResolver들이 HTTP 메시지 컨버터를 사용해서 필요한 객체를 생성
- **응답**
  - @ResponseBody와 HttpEntity를 처리하는 ReturnValueHandler가 있다
  - HTTP 메시지 컨버터를 호출해서 응답 결과 생성

---



## 확장

- 스프링은 필요하면 언제든지 기능을 확장할 수 있다
  - HandlerMethodArgumentResolver
  - HandlerMethodReturnValueHandler
  - HttpMessageConverter



※ 실제로 기능 확장할 일이 많지는 않음
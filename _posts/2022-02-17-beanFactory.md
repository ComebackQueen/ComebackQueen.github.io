---
layout: single
title:  "스프링 컨테이너와 스프링 빈"
toc: true
toc_sticky: true
categories:
  - Spring_Basic
---

#  BeanFactory와 ApplicationContext



## 들어가기 전  



![](/assets/images/2022-02-17-beanFactory/1.JPG)



---



## BeanFactory vs ApplicationContext



### **BeanFactory **  

- 스프링 컨테이너의 최상위 인터페이스
- 스프링 빈을 관리, 조회하는 역할
- `getBean()`을 제공 



### **ApplicationContext**

- BeanFactory 기능을 모두 상속받아서 제공
- BeanFactory와의 차이는? 부가기능 제공



**ApplicationContext가 제공하는 부가기능**

![](/assets/images/2022-02-17-beanFactory/2.JPG)

- 메시지소스를 활용한 **국제화** 기능
  - ex) 한국에서 들어오면 한국어로, 영어권에서 들어오면 영어로 출력
- **환경변수**
  - 로컬, 개발, 운영등을 구분해서 처리
- **애플리케이션 이벤트**
  - 이벤트를 발행하고 구독하는 모델을 편리하게 지원
- **편리한 리소스 조회**
  - 파일, 클래스패스, 외부 등에서 리소스를 편리하게 조회  



※ BeanFactory를 직접 사용할 일은 거의 없음. 부가기능이 포함된 **ApplicationContext를 사용!**





**4. 스프링 빈 의존관계 설정 - 완료**

![](/assets/images/2022-02-16-springContainer/4.JPG)

- 스프링 컨테이너는 설정 정보를 참고해서 의존관계 주입(DI)

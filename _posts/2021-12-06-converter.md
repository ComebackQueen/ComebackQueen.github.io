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



※ HTTP API로 JSON 데이터를 HTTP 메시지 바디에서 직접 읽거나 쓰는 경우 HTTP 메시지 컨버터를 사용하면 편리



- **스프링 MVC는 다음의 경우에 HTTP 메시지 컨버터를 적용**
  - HTTP 요청 : @RequestBody, HttpEntity(RequestEntity)
  - HTTP 응답 : @ResponseBody, HttpEntity(REsponseEntity)



---



## HTTP 메시지 컨버터 인터페이스

- **org.springframework.http.converter.HttpMessageConverter**

  ```java
  package org.springframework.http.converter;
  
  public interface HttpMessageConverter<T> {
  	boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);
  	boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);
      
  	List<MediaType> getSupportedMediaTypes();
      
  	T read(Class<? extends T> clazz, HttpInputMessage inputMessage)
  		throws IOException, HttpMessageNotReadableException;
  	void write(T t, @Nullable MediaType contentType, HttpOutputMessage 
  outputMessage)
  		throws IOException, HttpMessageNotWritableException;
  }
  ```
  
  - HTTP 메시지 컨버터는 HTTP 요청, HTTP 응답 둘 다 사용
  
    - canRead(), canWrite() : 메시지 컨버터가 해당 클래스, 미디어타입을 지원하는지 체크
  - read(), write() : 메시지 컨버터를 통해서 메시지를 읽고 쓰는 기능

---



## 스프링 부트 기본 메시지 컨버터

(일부 생략)

```java
0 = ByteArrayHttpMessageConverter
1 = StringHttpMessageConverter 
2 = MappingJackson2HttpMessageConverter
```

 ※ 스프링 부트는 다양한 메시지 컨버터를 제공, 대상 클래스 타입과 미디어 타입 둘을 체크해서 사용여부 결정

만약 만족하지 않으면 다음 메시지 컨버터로 우선순위가 넘어간다

- **ByteArrayHttpMessageConverter** : byte[] 데이터를 처리한다.
  - 클래스 타입: byte[] , 미디어타입: */* 
  - 요청 예) @RequestBody byte[] data
  - 응답 예) @ResponseBody return byte[] 쓰기 미디어타입 application/octet-stream
- **StringHttpMessageConverter** : String 문자로 데이터를 처리한다.
  - 클래스 타입: String , 미디어타입: */* 
  - 요청 예) @RequestBody String data
  - 응답 예) @ResponseBody return "ok" 쓰기 미디어타입 text/plain
- **MappingJackson2HttpMessageConverter** : application/json
  - 클래스 타입: 객체 또는 HashMap , 미디어타입 application/json 관련
  - 요청 예) @RequestBody HelloData data
  - 응답 예) @ResponseBody return helloData 쓰기 미디어타입 application/json 관련

---



- **HTTP 요청 데이터 읽기**
  - HTTP 요청, 컨트롤러에서 @RequestBody, HttpEntity 파라미터 사용
  - 메시지 컨버터가 메시지를 읽을 수 있는지 확인하기 위해 **canRead()**를 호출
    - 대상 클래스 타입을 지원하는가?
      - ex) @RequestBody의 대상 클래스 (byte[], String, HelloData)
    - HTTP 요청의 Content-Type 미디어 타입을 지원하는가?
      - ex) text/plain, application/json, */*
  - canRead() 조건을 만족하면 read()를 호출해서 객체 생성, 반환



- **HTTP 응답 데이터 생성**
  - 컨트롤러에서 @ResponseBody, HttpEntity로 값이 반환
  - 메시지 컨버터가 메시지를 읽을 수 있는지 확인하기 위해 **canWrite()**를 호출
    - 대상 클래스 타입을 지원하는가?
      - ex) return의 대상 클래스 (byte[], String, HelloData)
    - HTTP 요청의 Accept 미디어 타입을 지원하는가?(@RequestMapping의 produces)
      - ex) text/plain, application/json, */*
  - canWrite() 조건을 만족하면 write()를 호출해서 HTTP 응답 메시지 바디에 데이터를 생성

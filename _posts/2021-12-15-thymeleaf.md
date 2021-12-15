---
layout: single
title:  "스프링 MVC - 웹 페이지 만들기"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

#  상품 목록 - 타임리프



## 들어가기 전

- **BasicItemController**

```java
@Controller
@RequestMapping("/basic/items")
@RequiredArgsConstructor
public class BasicItemController {

    private final ItemRepository itemRepository;

    @GetMapping
    public String items(Model model) {
        List<Item> items = itemRepository.findAll();
        model.addAttribute("items", items);
        return "basic/items";
    }
    
    /**
     * 테스트용 데이터 추가
     */
    @PostConstruct // 해당 빈의 의존관계가 모두 주입되고 나면 초기화 용도로 호출
    public void init() {
        itemRepository.save(new Item("itemA", 10000, 10));
        itemRepository.save(new Item("itemB", 20000, 20));
    }
}
```

※ 컨트롤러 로직은 itemRepository에서 모든 상품을 조회한 다음에 모델에 담는다. 그리고 뷰 템플릿을 호출.



- ```@RequiredArgsConstructor```

  - ```final```이 붙은 멤버변수만 사용해서 생성자를 자동으로 만들어준다

  ```java
  public BasicItemController(ItemRepository itemRepository) {
   this.itemRepository = itemRepository;
  }
  // 생성자가 딱 1개만 있으면 스프링이 해당 생성자에 @Autowired로 의존관계를 주입해준다
  ```

  - **final 키워드를 빼면 안됨!** → ItemRepository 의존관계 주입이 안된다

  

---



## 타임리프 간단히 알아보기

**타임리프 사용 선언**

```<html xmlns:th="http://www.thymeleaf.org">```



**속성 변경 - th:href**

- ```href="value1"```을 ```th:href="value2"```의 값으로 변경
- 타임리프 뷰 템플릿을 거치게 되면 원래 값을 ```th:xxx```값으로 변경(만약 값이 없다면 새로 생성)
- HTML을 그대로 볼 때는 href 속성이 사용되고, 뷰 템플릿을 거치면 ```th:href```의 값이 href로 동적으로 대체



**타임리프 핵심**

- ```th:xxx```가 붙은 부분은 서버사이드에서 렌더링 되고, 기존 것을 대체. ```th:xxx```이 없으면 기존의 html ```xxx```속성이 그대로 사용된다
- HTML을 파일로 직접 열었을 때, ```th:xxx```가 있어도 웹 브라우저는 ```th:```속성을 알지 못하므로 무시한다
- HTML을 파일 보기를 유지하면서 템플릿 기능도 할 수 있다



**URL 링크 표현식 - @{...}**

```th:href="@{/css/bootstrap.min.css}"```

- ```@{...}``` : 타임리프는 URL 링크를 사용하는 경우 ```@{...}```를 사용, 이를 URL 링크 표현식이라 한다
- URL 링크 표현식을 사용하면 서블릿 컨텍스트를 자동으로 포함



**상품 등록 폼으로 이동**

**속성 변경 - th:onClick**

- `onclick="location.href='addForm.html'"`

- `th:onclick="|location.href='@{/basic/items/add}'|"`

  ※여기는 리터럴 대체 문법이 사용됨



**리터럴 대체 - |...|**

`|...|` 

- 타임리프에서 문자와 표현식 등은 분리되어 있기 때문에 더해서 사용해야 한다
  - `<span th:text="'Welcome to our application, ' + ${user.name} + '!'">`
- 다음과 같이 리터럴 대체 문법을 사용하면, 더하기 없이 편리하게 사용 가능
  - `<span th:text="|Welcome to our application, ${user.name}!|">`



**반복 출력 - th:each**

- `<tr th:each="item : ${items}">`
- 반복은 `th:each`를 사용.
- 컬렉션의 수 만큼 `<tr>..</tr>`이 하위 태그를 포함해서 생성



**변수 표현식 - ${...}**

- `<td th:text="${item.price}">10000</td>`
- 모델에 포함된 값이나, 타임리프 변수로 선언한 값을 조회가능
- 프로퍼티 접근법을 사용(`item.getPrice()`)



**내용 변경 - th:text**

- `<td th:text="${item.price}">10000</td>`
- 내용의 값을 `th:text`의 값으로 변경
- 여기서 10000을 `${item.price}`의 값으로 변경



**URL 링크 표현식2 - @{...}**

- `th:href="@{/basic/items/{itemId}(itemId=${item.id})}"`
- URL 링크 표현식을 사용하면 경로를 템플릿처럼 편리하게 사용할 수 있다
- 경로 변수(`itemId`) 뿐만 아니라 쿼리 파라미터도 생성한다
- ex)`th:href="@{/basic/items/{itemId}(itemId=${item.id}, query='test')}"`
  - 생성 링크 : `http://localhost:8080/basic/items/1?query=test`



**URL 링크 간단히**

- `th:href="@{|/basic/items/${item.id}|}"`
- 리터럴 대체 문법을 활용해서 간단히 사용할 수 있다
---
layout: single
title:  "스프링 MVC - 웹 페이지 만들기"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

#  RedirectAttributes



## 들어가기 전

⁕ 상품 저장 후 상품 상세 화면으로 리다이렉트 O

하지만 고객 입장에서 저장이 잘 된 것인지 아닌지 확인 불가

이를 위해 "저장되었습니다"라는 메시지가 뜨게 해결해야함



---



## RedirectAttributes

- **BasicItemController에 추가**

```java
/*
	리다이렉트 시 status=true를 추가 -> 뷰 템플릿에서 이 값이 있으면 저장되었습니다가 출력
*/
@PostMapping("/add")
public String addItemV6(Item item, RedirectAttributes redirectAttributes){
        Item saveItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", saveItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/basic/items/{itemId}";
    }
```



- **RedirectAttributes**
  - `RedirectAttributes`를 사용하면 URL 인코딩, `pathVariable`, 쿼리 파라미터까지 처리



- `redirect:/basic/items/{itemId}`
  - pathVariable 바인딩: `{itemId}`
  - 나머지는 쿼리 파라미터로 처리 : `?status=true`

---



## 뷰 템플릿 메시지 추가



- **`resources/templates/basic/item.html`**

```html
<div class="container">
    
 <div class="py-5 text-center">
 <h2>상품 상세</h2>
 </div>
    
 <!-- 추가 -->
 <h2 th:if="${param.status}" th:text="'저장 완료!'"></h2>
```

- `th:if` : 해당 조건이 참이면 실행
- `${param.status}` : 타임리프에서 쿼리 파라미터를 편리하게 조회하는 기능

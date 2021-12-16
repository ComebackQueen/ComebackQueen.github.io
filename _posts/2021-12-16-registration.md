---
layout: single
title:  "스프링 MVC - 웹 페이지 만들기"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

#  상품 상세 / 상품 등록 폼



## 상품 상세

- **BasicItemController에 추가**

```java
@GetMapping("/{itemId}")
public String item(@PathVariable long itemId, Model model) {
    Item item = itemRepository.findById(itemId);
    model.addAttribute("item", item);
    return "basic/item";
}
```

※ `PathVariable`로 넘어온 상품ID로 상품을 조회, 모델에 담아둔다. 그리고 뷰 템플릿을 호출

- **상품 상세 뷰**

`/resources/templates/basic/item.html`

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
    <link th:href="@{/css/bootstrap.min.css}"
            href="../css/bootstrap.min.css" rel="stylesheet">
    <style>
        .container {
            max-width: 560px;
        }
    </style>
</head>
<body>
<div class="container">
    <div class="py-5 text-center">
        <h2>상품 상세</h2>
    </div>
    <!-- 추가 -->
    <h2 th:if="${param.status}" th:text="'저장 완료'"></h2>

    <div>
        <label for="itemId">상품 ID</label>
        <input type="text" id="itemId" name="itemId" class="form-control"
               value="1" th:value="${item.id}" readonly>
    </div>
    <div>
        <label for="itemName">상품명</label>
        <input type="text" id="itemName" name="itemName" class="form-control"
               value="상품A" th:value="${item.itemName}" readonly>
    </div>
    <div>
        <label for="price">가격</label>
        <input type="text" id="price" name="price" class="form-control"
               value="10000" th:value="${item.price}" readonly>
    </div>
    <div>
        <label for="quantity">수량</label>
        <input type="text" id="quantity" name="quantity" class="form-control"
               value="10" th:value="${item.quantity}" readonly>
    </div>
    <hr class="my-4">
    <div class="row">
        <div class="col">
            <button class="w-100 btn btn-primary btn-lg"
                    onclick="location.href='editForm.html'"
                    th:onclick="|location.href='@{/basic/items/{itemId}/edit(itemId=${item.id})}'|"
                    type="button">상품 수정</button>
        </div>
        <div class="col">
            <button class="w-100 btn btn-secondary btn-lg"
                    onclick="location.href='items.html'"
                    th:onclick="|location.href='@{/basic/items}'|"
                    type="button">목록으로</button>
        </div>
    </div>
</div> <!-- /container -->
</body>
</html>
```

- **속성 변경 - th:value**

  `th:value="${item.id}"`

  - 모델에 있는 item 정보를 획득하고 프로퍼티 접근법으로 출력(item.getId())
  - `value`속성을 `th:value` 속성으로 변경

- **상품수정 링크**

  - `th:onclick="|location.href='@{/basic/items/{itemId}/edit(itemId=${item.id})}'|"`

- **목록으로 링크**

  - `th:onclick="|location.href='@{/basic/items}'|"`



---



## 상품 등록 폼

**BasicItemController에 추가**

```java
@GetMapping("/add")
    public String addForm(){
        return "basic/addForm";
    }
```



**상품 등록 폼 뷰**

`/resources/templates/basic/addForm.html`

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
    <link th:href="@{/css/bootstrap.min.css}"
            href="../css/bootstrap.min.css" rel="stylesheet">
    <style>
        .container {
            max-width: 560px;
        }
    </style>
</head>
<body>
<div class="container">
    <div class="py-5 text-center">
        <h2>상품 등록 폼</h2>
    </div>
    <h4 class="mb-3">상품 입력</h4>
    <form action="item.html" th:action method="post">
        <div>
            <label for="itemName">상품명</label>
            <input type="text" id="itemName" name="itemName" class="form-control" placeholder="이름을 입력하세요">
        </div>
        <div>
            <label for="price">가격</label>
            <input type="text" id="price" name="price" class="form-control"
                   placeholder="가격을 입력하세요">
        </div>
        <div>
            <label for="quantity">수량</label>
            <input type="text" id="quantity" name="quantity" class="form-control" placeholder="수량을 입력하세요">
        </div>
        <hr class="my-4">
        <div class="row">
            <div class="col">
                <button class="w-100 btn btn-primary btn-lg" type="submit">상품
                    등록</button>
            </div>
            <div class="col">
                <button class="w-100 btn btn-secondary btn-lg"
                        onclick="location.href='items.html'"
                        th:onclick="|location.href='@{/basic/items}'|"
                        type="button">취소</button>
            </div>
        </div>
    </form>
</div> <!-- /container -->
</body>
</html>

```

- **속성 변경 - th-action**
  - HTML form에서 action에 값이 없으면 현재 URL에 데이터를 전송
  - 상품 등록 폼의 URL과 실제 상품 등록을 처리하는 URL을 똑같이 맞추고 HTTP 메서드로 두 기능을 구분
    - 상품 등록 폼 : GET`/basic/items/add`
    - 상품 등록 처리 : POST `/basic/items/add`



- **취소**
  - 취소시 상품 목록으로 이동
  - `th:onclick="|location.href='@{/basic/items}'|"`
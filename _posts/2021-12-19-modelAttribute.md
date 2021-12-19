---
layout: single
title:  "스프링 MVC - 웹 페이지 만들기"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

#  상품 등록 처리 - @ModelAttribute



## POST - HTML Form

- `content-type: application/x-www-form-urlencoded`
- 메시지 바디에 쿼리 파라미터 형식으로 전달 → `itemName=itemA&price=10000&quantity=10`
- ex) 회원 가입, 상품 주문, HTML Form 사용



---

## 상품 등록 처리 - @RequestParam



- **addItemV1 - BasicItemController에 추가**

```java
    @PostMapping("/add")
    public String addItemV1(@RequestParam String itemName,
                       @RequestParam int price,
                       @RequestParam Integer quantity,
                       Model model){

        Item item = new Item();
        item.setItemName(itemName);
        item.setPrice(price);
        item.setQuantity(quantity);

        itemRepository.save(item);

        model.addAttribute("item", item);

        return "basic/item";
    }
```

- `@Requestparam String itemName` : itemName 요청 파라미터 데이터를 해당 변수에 받음
- `Item`객체를 생성하고 `itemRepository`를 통해서 저장
- 저장된 `item`을 모델에 담아서 뷰에 전달

---

## 상품 등록 처리 - @ModelAttribute



- **addItemV2 - 상품 등록 처리 - ModelAttribute**

```java
    @PostMapping("/add")
    public String addItemV2(@ModelAttribute("item") Item item, Model model){

        itemRepository.save(item);
//        model.addAttribute("item", item); //자동 추가, 생략 가능

        return "basic/item";
    }
```

- **@ModelAttribute - 요청 파라미터 처리**
  - `@ModelAttribute`는 `Item`객체를 생성하고, 요청 파라미터의 값을 프로퍼티 접근법(setXxx)으로 입력



- **@ModelAttribute - Model 추가**
  - `@ModelAttribute`의 중요한 기능? 모델에 `@ModelAttribute`로 지정한 객체를 자동으로 넣어준다

※ `@ModelAttribute`의 이름을 다르게 지정한다면?

`@ModelAttribute("hello") Item item` → 이름을 `hello`로 지정

model.addAttribute("hello", item) → 모델에 `hello`이름으로 저장



- **addItemV3 - 상품 등록 처리 - ModelAttribute 이름 생략**

```java
/**
 * @ModelAttribute name 생략 가능
 * model.addAttribute(item); 자동 추가, 생략 가능
 * 생략시 model에 저장되는 name은 클래스명 첫글자만 소문자로 등록 Item -> item
 */  
@PostMapping("/add")
public String addItemV3(@ModelAttribute Item item, Model model){

        itemRepository.save(item);
//        model.addAttribute("item", item); //자동 추가, 생략 가능

        return "basic/item";
    }
```



- **addItemV4 - 상품 등록 처리 - ModelAttribute 전체 생략**

```java
/**
 * @ModelAttribute 자체 생략 가능
 * model.addAttribute(item) 자동 추가
 */
@PostMapping("/add")
public String addItemV4(Item item){

        itemRepository.save(item);
//        model.addAttribute("item", item); //자동 추가, 생략 가능

        return "basic/item";
    }
```


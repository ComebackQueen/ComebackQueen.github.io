---
layout: single
title:  "스프링 MVC - 웹 페이지 만들기"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

#  상품 도메인 개발



## Item - 상품 객체

```java
@Data
public class Item {

    private Long id;
    private String itemName;
    private Integer price;
    private Integer quantity;

    public Item() {
    }

    public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}
```

- @Data
  - @Getter @Setter @RequiredArgsConstructor @ToString 등 모두 포함 시킨 어노테이션
  - 위험 (@RequiredArgsConstructor @EqualsAndHash 를 조합한 어노테이션이기 때문에 각 어노테이션의 주의사항이 포함됨) → @Getter @Setter를 사용하는게 낫다

---



## ItemRepository - 상품 저장소

```java
@Repository
public class ItemRepository {

    private static final Map<Long, Item> store = new HashMap<>(); //static
    private static long sequence = 0L; //static

    public Item save(Item item) {
        item.setId(++sequence);
        store.put(item.getId(), item);
        return item;
    }

    public Item findById(Long id) {
        return store.get(id);
    }

    public List<Item> findAll() {
        return new ArrayList<>(store.values());
    }

    public void update(Long itemId, Item updateParam) {
        Item findItem = findById(itemId);
        findItem.setItemName(updateParam.getItemName());
        findItem.setPrice(updateParam.getPrice());
        findItem.setQuantity(updateParam.getQuantity());
    }

    public void clearStore() {
        store.clear();
    }
}
```

- @Repository
  - DAO를 인식시켜주기 위한 설정(※DAO란? DB를 사용해 데이터를 조회하거나 조작하는 기능을 전담하도록 만든 오브젝트)